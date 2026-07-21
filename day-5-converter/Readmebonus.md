# Bonus Build — SFTP to SFTP File Transfer with XML to JSON Conversion

## What it does
An integration flow that picks up an XML file from one SFTP folder, 
converts it to JSON, and moves it into a different SFTP folder.

**Flow:** SFTP Sender → XML to JSON Converter → SFTP Receiver

## Adapters & steps used
- **Sender (Source):** SFTP, Address `eu-central-1.sftpcloud.io`, 
  File Name `order2.xml`, Authentication: User Name/Password, 
  Credential Name: `SFTPCloudCRED`.
- **Message Transformer:** XML to JSON Converter, default configuration.
- **Receiver (Target):** SFTP, same address and credential as sender, 
  File Name `order2.JSON`, writes to `/archive`.

## Test payload (XML)
```xml
<customer>
<name>Prudhvi</name>
<email>prudhvi@example</email>
<city>VZD</city>
</customer>
```

## Result
Message processing completed successfully. Output landed in `/archive` 
as `order2.JSON`:
```json
{"customer":{"name":"Prudhvi","email":"prudhvi@example","city":"VZD"}}
```

## Real errors encountered (and fixed independently)
- **Auth cancel for methods 'password,publickey'** — SFTP poll failed 
  repeatedly (24 consecutive failures) despite the server being 
  confirmed alive with matching credentials. Root cause not fully 
  isolated (likely a credential deploy/sync issue in Security Material); 
  resolved itself after re-checking and the next poll cycle succeeded.
- **WstxUnexpectedCharException: Unexpected character '\' in content 
  after '<'** — the source XML file used backslashes (`<\name>`) 
  instead of forward slashes (`</name>`) in its closing tags, making it 
  malformed XML. Fixed by correcting all closing tags directly in the 
  file via WinSCP's editor, including a missed root closing tag 
  (`<customer>` instead of `</customer>`).

## Files
[XML_to_JSON_WhileMovingTheFile.zip](https://github.com/user-attachments/files/30214981/XML_to_JSON_WhileMovingTheFile.zip)
