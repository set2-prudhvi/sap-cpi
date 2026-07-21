# Day 5-6 — Mapping & Message Transformation (XML to JSON)

## What it does
An integration flow that receives XML customer data over HTTPS, converts 
it to JSON using a Message Transformer, and forwards it to a public echo 
endpoint to confirm the conversion happened correctly.

**Flow:** HTTPS Sender → XML to JSON Converter → HTTP Receiver → postman-echo.com

## Adapters & steps used
- **Sender:** HTTPS, address `/Customerdata`, Authorization: User Role 
  (`ESBMessaging.send`), CSRF Protection disabled for testing.
- **Message Transformer:** XML to JSON Converter, default configuration 
  (no namespace mapping, no root suppression).
- **Receiver:** HTTP, address `https://postman-echo.com/post`, 
  Authentication: None.

## Test payload (XML sent)
```xml
<Customer>
  <Id>1001</Id>
  <Name>Prudhvi</Name>
  <Number>9876543210</Number>
</Customer>
```

## Result
`200 OK`. postman-echo's response confirmed the payload arrived as JSON:
```json
"data": {
  "Customer": {
    "Id": "1001",
    "Name": "Prudhvi",
    "Number": "9876543210"
  }
}
```

## Theory covered
- **Converter vs Message Mapping** — a Converter (what this iFlow uses) 
  is a pure structural translator: it reformats XML tags into JSON keys 
  with no business logic and no schema requirement. Message Mapping is a 
  different tool entirely — a graphical canvas where source and target 
  fields are connected via schemas (XSD/JSON Schema), with functions 
  (concatenate, uppercase, lookups, conditions) applied along the way. 
  Use a Converter when structure already matches; use Message Mapping 
  when fields, names, or structure genuinely differ (e.g. merging 
  `FirstName` + `LastName` into `FullName`).
- **JSON to XML (reverse direction)** — same converter family, opposite 
  direction. Common use case: a modern system sends JSON but needs to 
  reach an older system (e.g. SAP ECC via IDoc) that only understands 
  XML. Since JSON has no native "root element" concept, the converter 
  has to invent one if the source JSON doesn't clearly have a single 
  top-level wrapper.
- **XML has no data types** — everything inside an XML tag is text. The 
  XML to JSON Converter carries this forward by default, meaning numeric 
  looking fields land in JSON as strings, not numbers.

## Known limitation / finding
The XML to JSON Converter's Processing tab (Namespace Mapping, JSON 
Prefix Separator, JSON Output Encoding, Suppress JSON Root Element, 
Streaming) has **no per-field data type control**. Numeric fields like 
`Id` and `Number` are output as JSON strings (`"1001"`) instead of 
numbers (`1001`). Some strict receiving APIs would reject this as a 
type mismatch.

**Fix planned for Day 7:** a Groovy Script step after the converter to 
explicitly parse the JSON body and cast fields like `Id` and `Number` 
to integers before sending downstream.

## File ##
<img width="1366" height="768" alt="Screenshot (124)" src="https://github.com/user-attachments/assets/cb8f54cb-6b6e-4d5b-97a2-9656c7a8ad68" />
<img width="1366" height="768" alt="Screenshot (126)" src="https://github.com/user-attachments/assets/7e5d4b31-59b4-4656-add3-ac6c7beb4213" />
<img width="1366" height="768" alt="Screenshot (127)" src="https://github.com/user-attachments/assets/83dc1bf9-5619-4c91-9658-25f9bb210513" />
[XML_To_JSON_CustomerData.zip](https://github.com/user-attachments/files/30211943/XML_To_JSON_CustomerData.zip)
