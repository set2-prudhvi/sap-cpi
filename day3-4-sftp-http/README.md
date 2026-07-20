# Day 3-4 — SFTP to HTTP File Transfer

## What it does
An integration flow that polls an SFTP server on a schedule, picks up a
file dropped in an inbound folder, forwards it to an HTTP endpoint, and
moves the processed file to an archive folder — a classic file-based
integration pattern (vendor drops a file, CPI picks it up and pushes it
downstream).

**Flow:** SFTP Sender (poll) → HTTP Receiver → postman-echo.com

## Concepts covered
- **Polling vs Push** — CPI checks the SFTP server on a schedule
  (polling), instead of waiting for the source system to trigger it
  (push). SFTP is inherently a pull-based pattern.
- **SFTP adapter behavior** — file-based, moves files to an archive
  folder after processing; does not edit file content in transit.
- **Adapter landscape** — HTTP/SOAP/OData (push, real-time) vs SFTP
  (pull, scheduled) vs IDoc (SAP-proprietary, requires Cloud Connector
  to bridge on-prem ECC to cloud CPI).
- **SSH host key trust** — why an unknown SFTP server gets rejected by
  default, and how to explicitly trust it via a Known Hosts entry
  rather than bypassing the check.

## Adapters used
- **Sender:** SFTP
  - Source Directory: `/inbound`
  - File Name: `order1.csv`
  - Address: `eu-central-1.sftpcloud.io:22` (no `sftp://` prefix — CPI
    adds it automatically; including it causes a double-prefix bug)
  - Authentication: User Name/Password via Security Material
    credential (`SFTPCloudCRED`)
  - Post-Processing: Move File → Archive Directory `/archive`
  - Scheduler: Recurring, every 30 sec, daily 00:00–24:00 GMT
- **Receiver:** HTTP, address `https://postman-echo.com/post`,
  Authentication: None

## Debugging log (real errors encountered)
- **JSchUnknownHostKeyException: reject HostKey** — CPI refused to
  connect because it didn't recognize the SFTP server's SSH host key.
  This isn't a config mistake — it's SSH refusing to trust an unknown
  server's identity. Fixed by:
  1. Running `ssh-keyscan -p 22 <host>` locally to retrieve the
     server's public host key(s).
  2. Redirecting the output to a file (`> hostkey.txt`) rather than
     trying to copy text out of Command Prompt, which doesn't support
     normal clipboard behavior by default.
  3. Uploading that file directly into **Manage Security Material →
     Create → Known Hosts (SSH)** in CPI (the dialog accepts a file
     upload, not pasted text) and deploying it.
  4. Redeploying the iFlow so it picked up the newly trusted host key.
- **Expired test credentials mid-troubleshooting** — the free SFTP
  test server (sftpcloud.io) only lives ~1 hour. It expired partway
  through the first debugging pass, killing the credentials being
  tested. Resolved by spinning up a fresh server, updating the
  Security Material credential, re-uploading the test file, and
  redoing the host key registration against the new host.
- **False alarm — file appeared "stuck" in `/inbound`** — Monitor
  showed the message as `Completed`, but WinSCP still showed the file
  in `/inbound` with nothing in `/archive`. This was a stale WinSCP
  view, not a real failure — reconnecting/refreshing the session
  confirmed the file had, in fact, moved to `/archive` at the same
  timestamp as the completed message.
- **200 / Completed** — full flow confirmed working end to end: file
  picked up from `/inbound`, delivered to the HTTP receiver, and moved
  to `/archive`.

## Key takeaway
The actual integration logic (SFTP sender → HTTP receiver) was simple
to configure. The real learning was in the security layer around it —
understanding *why* an unknown host gets rejected, and fixing it
properly instead of disabling the check, plus recognizing a stale
monitoring view instead of assuming the flow was broken.

## Files
<img width="1366" height="768" alt="Screenshot (119)" src="https://github.com/user-attachments/assets/97a06216-6d93-42de-b4bc-c28e340e8517" />
[SFTP_HTTP_FileTransfer.zip](https://github.com/user-attachments/files/30176586/SFTP_HTTP_FileTransfer.zip)
<img width="1366" height="768" alt="Screenshot (110)" src="https://github.com/user-attachments/assets/7daf6213-4c07-40c1-87f8-1882808768e3" />
<img width="1366" height="768" alt="Screenshot (111)" src="https://github.com/user-attachments/assets/7677a0e7-5c48-4f79-9e64-e665a2fedb2f" />
<img width="1366" height="768" alt="Screenshot (116)" src="https://github.com/user-attachments/assets/1744d5f2-5e27-4ed9-8951-2338ec7ce172" />
<img width="1366" height="768" alt="Screenshot (118)" src="https://github.com/user-attachments/assets/01610606-ad60-4b3a-aa3e-e18bc6b2a625" />
<img width="1366" height="768" alt="Screenshot (117)" src="https://github.com/user-attachments/assets/4875a9b2-82d1-42d7-a0c5-0f56b226ddd4" />

