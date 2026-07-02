# Broken Access Control

## What is Broken Access Control?

Broken Access Control occurs when users can access resources or perform actions they are not authorised for. This includes Insecure Direct Object References (IDOR), missing function level access control, and broken object/property level authorisation in APIs.

## Exploits

- [HTTP Verb Tampering](broken-access-control/verb-tampering.md)
- [Insecure Direct Object References (IDOR)](broken-access-control/idor.md)
- [API-Specific Broken Access Control (BOLA / BFLA)](broken-access-control/api-bola.md)

## Attack Surface

- Any endpoint or parameter that references user-owned objects (user IDs, file IDs, order numbers, etc.)
- Admin vs user functionality
- APIs and AJAX calls that may bypass UI controls
- Multi-tenant applications
- File download/view features
- Update or delete operations that rely on client-supplied identifiers

## Identification

- Map the application while authenticated as different roles/users.
- Look for IDs, UUIDs, filenames, or other references in URLs, forms, AJAX, cookies, or JWT payloads.
- Compare functionality available to different user roles.
- Test changing identifiers (numeric, hashed, encoded) in requests.
- Look for hidden parameters in requests that control authorisation (uid, role, is_admin, etc.).
- Check for predictable patterns in object references (sequential IDs, simple hashes of filenames).

## Impact

- Unauthorised access to other users' data (PII, financial records, messages)
- Privilege escalation (horizontal and vertical)
- Data modification or deletion belonging to other accounts
- Complete account takeover or administrative control in severe cases
- Compliance violations (GDPR, etc.)

## Prevention

- Implement access control checks on every request, on the server side.
- Use indirect object references or GUIDs that cannot be guessed or enumerated.
- Enforce authorisation at the object and function level, not just the UI.
- For APIs: validate that the authenticated user owns or is permitted to access the requested object for every operation.
- Avoid exposing internal IDs when possible; use ACLs or capability-based tokens.
- Apply the principle of least privilege.
- Regularly test authorisation logic with different user roles.

## Tools & Payloads

- Burp Suite Repeater and Intruder for testing ID changes and verb tampering
- curl for quick manual tests and scripting mass access
- jq for parsing JSON API responses
- Hashing/encoding tools: `base64`, `md5sum`, CyberChef for recreating client-side transformations
- Scripts for enumeration (see examples above)

See also the dedicated [Authentication](broken-authentication.md) page for related session and auth bypass techniques that can compound access control issues.
