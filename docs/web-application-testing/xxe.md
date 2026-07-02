# XXE

## What is XXE?

XML External Entity (XXE) Injection allows an attacker to interfere with an application's processing of XML data, often leading to disclosure of internal files or server-side request forgery.

## Exploits

- [Basic XXE File Disclosure](xxe/basic-xxe.md)
- [XXE for SSRF](xxe/xxe-ssrf.md)

## Impact

File disclosure, SSRF, DoS.

## Prevention

Disable external entities in XML parsers. Prefer JSON.

## Tools & Payloads

Burp Repeater, custom XML payloads with the basic entity example from content.
