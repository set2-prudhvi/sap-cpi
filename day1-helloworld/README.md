# Day 1 — Hello World iFlow (HTTP Echo)

## What it does
A simple integration flow that receives an HTTP POST request, injects a 
custom message using a Content Modifier, and forwards it to a public 
echo endpoint to confirm the message content and headers.

**Flow:** HTTPS Sender → Content Modifier → HTTP Receiver → postman-echo.com

## Adapters used
- **Sender:** HTTPS, address `/helloworld`, Authorization: User Role 
  (`ESBMessaging.send`), CSRF Protection disabled for testing.
- **Content Modifier:** Sets message body to a static string, adds a 
  dynamic Timestamp header using CPI expression syntax.
- **Receiver:** HTTP, address `https://postman-echo.com/post`, 
  Authentication: None.

## Debugging log (real errors encountered)
- **401 Unauthorized** — sender had no auth credentials in the test request.
- **403 Forbidden** — assigned role name didn't match the adapter's 
  expected technical role (`ESBMessaging.send`), and CSRF protection 
  was blocking direct POST calls.
- **500 Internal Server Error** — receiver adapter still had leftover 
  "Client Certificate" authentication selected with no certificate configured.
- **503 Service Unavailable** — the original test receiver (httpbin.org) 
  was temporarily down; switched to postman-echo.com, which resolved it.
- **200 OK** — full flow confirmed working end to end.

## Screenshots
<img width="1366" height="768" alt="Screenshot (33)" src="https://github.com/user-attachments/assets/f24ab52b-4e07-44c6-9930-886c897aa277" />

<img width="1366" height="768" alt="Screenshot (48)" src="https://github.com/user-attachments/assets/b585eca7-5f6a-4dc1-ad54-410c103853c5" />


## Files
- `HelloWorld_HTTP_Echo.zip` — exported iFlow package
