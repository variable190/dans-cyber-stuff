# Insecure Deserialization

## Overview

Insecure deserialization vulnerabilities arise when an application deserializes untrusted data without sufficient validation. Attackers can craft malicious serialized objects that, when deserialized, execute arbitrary code, modify application logic, or lead to denial of service.

This is a language-specific issue (common in Java, PHP, Python, .NET, Node.js) and is often referred to as "object injection."

## Attack Surface

- Features that accept serialized data (cookies, API parameters, file uploads, session data, cached objects).
- Use of functions like `unserialize()` (PHP), `readObject()` (Java), `pickle` (Python), etc. on user-controlled input.

## Identification

- Look for parameters, cookies, or files containing base64 or binary data that looks like serialized objects.
- Test by modifying serialized data and observing application behavior or errors.
- Use known gadget chains for the target language/framework.
- Monitor for exceptions or unusual processing during deserialization.

## Exploitation

### General Approach

1. Identify where serialized data is accepted and deserialized.
2. Obtain or generate a valid serialized object for the target (often by serializing a legitimate object first).
3. Modify or replace the object with a malicious payload (gadget chain) that executes code on deserialization.
4. Deliver the payload.

### PHP Example (Phar Deserialization via Upload + LFI)

From file upload techniques:
```bash
php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
```
Then trigger via LFI:
`/index.php?language=phar://./uploads/shell.jpg/shell.txt&cmd=id`

### Other Common Vectors

- Cookie or session manipulation with serialized user objects.
- API endpoints that accept serialized data for caching or state.
- File processing that internally deserializes content.

**Note:** Detailed gadget chains and language-specific payloads are extensive. See resources such as PayloadsAllTheThings (Insecure Deserialization section) and ysoserial for Java.

## Impact

- Remote code execution
- Authentication bypass or privilege escalation
- Data tampering
- Denial of service (via resource exhaustion during deserialization)

## Prevention

- Avoid deserializing untrusted data whenever possible.
- Use safe, language-native data formats like JSON with strict schemas instead of native serialization.
- Implement integrity checks (signing/hashing) on serialized data.
- Use whitelisting of allowed classes during deserialization (where supported).
- Run applications with least privilege.
- Keep all libraries and runtimes updated (many deserialization gadgets rely on vulnerable dependencies).

## Tools & Payloads

- ysoserial (Java)
- Custom scripts to generate/modify serialized objects
- Burp extensions for deserialization testing
- Phar creation tools for PHP scenarios (combined with LFI/File Upload)

See the [File Upload Attacks](file-upload-attacks.md) and [File Inclusion](file-inclusion.md) pages for practical PHP phar and wrapper chaining examples. OWASP Top 10 also highlights this as a recurring risk area.
