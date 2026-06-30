# XXE

## Overview

XML External Entity (XXE) Injection is a vulnerability that occurs when an application processes XML input containing a user-controlled Document Type Definition (DTD). Attackers can define external entities that cause the XML parser to access local or remote resources, leading to file disclosure, SSRF, or in some cases remote code execution.

## Attack Surface

- Any feature that accepts or processes XML data (file uploads, SOAP services, REST APIs with XML, document imports, configuration uploads).
- Applications using vulnerable XML parsers (many default parsers in Java, PHP, .NET, Python are vulnerable unless hardened).

## Identification

- Look for endpoints that accept XML in requests or file uploads.
- Test by injecting a simple external entity that triggers an out-of-band request or error.
- May need to change requests from GET to POST to include XML bodies.
- Monitor for differences in responses or out-of-band interactions.

## Exploitation

### Basic XML Structure

```xml
<?xml version="1.0" encoding="UTF-8"?>
<email>
  <date>01-01-2022</date>
  <sender>john@example.com</sender>
  ...
</email>
```

### DTD for External Entities

```xml
<!DOCTYPE email [
  <!ELEMENT email (date, time, sender, recipients, body)>
  ...
  <!ENTITY xxe SYSTEM "file:///etc/passwd">
]>
```

Use the entity in the document:
```xml
<sender>&xxe;</sender>
```

**Tip:** Convert GET requests to POST when sending XML payloads.

### Common Payload Goals

- Read local files (`file:///etc/passwd`)
- SSRF to internal services
- Billion Laughs / entity expansion for DoS (less common in modern parsers)
- Out-of-band exfiltration

## Impact

- Disclosure of local files and sensitive data (source code, configuration, credentials)
- Server-side request forgery (SSRF)
- Potential remote code execution in certain parser/language combinations
- Denial of service

## Prevention

- Disable external entity resolution in the XML parser (preferred).
- Use less complex data formats such as JSON when possible.
- Apply input validation and whitelisting on XML data.
- Use hardened XML parsers and keep libraries updated.
- Run the application with minimal privileges so that even successful XXE yields limited access.

## Tools & Payloads

- Manual crafting in Burp Repeater
- PayloadsAllTheThings XXE section for advanced variants
- Out-of-band tools (Burp Collaborator, Interactsh) for blind detection

**Basic test payload skeleton:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<root><data>&xxe;</data></root>
```

See the [SSRF](ssrf.md) page for related techniques.
