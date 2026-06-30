# Broken Access Control

## Overview

Broken Access Control occurs when users can access resources or perform actions they are not authorized for. This includes Insecure Direct Object References (IDOR), missing function level access control, and broken object/property level authorization in APIs.

It is distinct from authentication issues: the user may be authenticated, but the application fails to properly enforce what that user is allowed to do.

Common manifestations:
- IDOR: accessing other users' data by changing identifiers
- Verb tampering and HTTP method abuse
- Horizontal and vertical privilege escalation via authorization flaws
- API-specific issues (BOLA, BFLA)

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
- Look for hidden parameters in requests that control authorization (uid, role, is_admin, etc.).
- Check for predictable patterns in object references (sequential IDs, simple hashes of filenames).

## Exploitation

### HTTP Verb Tampering

Applications may only apply security controls to certain HTTP methods (commonly GET).

Example vulnerable code (only validates on GET):
```php
$pattern = "/^[A-Za-z\s]+$/";
if(preg_match($pattern, $_GET["code"])) {
    $query = "Select * from ports where port_code like '%" . $_REQUEST["code"] . "%'";
    ...
}
```

Exploitation:
- Use Burp to change the verb on intercepted requests (try POST, PUT, DELETE, PATCH, OPTIONS, HEAD).
- Discover allowed methods: `curl -i -X OPTIONS http://target/`

### Insecure Direct Object Referencing (IDOR)

**Finding references:**
- URL parameters and APIs: `?uid=1`, `?filename=file_1.pdf`
- AJAX calls for privileged functions.
- Hashed or encoded references (e.g., MD5 of filename or base64 + hash).
- Predictable naming: `/documents/Invoice_1_09_2021.pdf`

**Mass enumeration example** (adapt to target):
```bash
#!/bin/bash
url="$1"

for i in {1..10}; do
    for link in $(curl -s "$url/documents.php?uid=$i" | grep -oP "\/documents.*?.pdf"); do
        wget -q $url/$link
    done
done
```

**Hashed IDOR example:**
```bash
echo -n 1 | base64 -w 0 | md5sum
# Then use the hash in requests
```

**Insecure APIs example:**
APIs often expose more data or actions. Look for hidden fields like uid, uuid, role in JSON or form data. Changing them (even via GET instead of PUT) can allow access to other users' data or admin functions.

### API-Specific Broken Access Control

- **Broken Object Level Authorization (BOLA / IDOR in APIs)**: Authenticate as low-priv user, then iterate object IDs in API calls.
- **Broken Function Level Authorization (BFLA)**: Discover admin-only endpoints and call them as a regular user.
- **Broken Object Property Level Authorization**: Modify sensitive properties (e.g., `isAdmin`, `role`) that should not be controllable by the user.

Example automation for BOLA:
```bash
for ((i=1; i<=20; i++)); do
  curl -s ... /api/v1/suppliers/quarterly-reports/$i ...
done
```

## Impact

- Unauthorized access to other users' data (PII, financial records, messages)
- Privilege escalation (horizontal and vertical)
- Data modification or deletion belonging to other accounts
- Complete account takeover or administrative control in severe cases
- Compliance violations (GDPR, etc.)

## Prevention

- Implement access control checks on every request, on the server side.
- Use indirect object references or GUIDs that cannot be guessed or enumerated.
- Enforce authorization at the object and function level, not just the UI.
- For APIs: validate that the authenticated user owns or is permitted to access the requested object for every operation.
- Avoid exposing internal IDs when possible; use ACLs or capability-based tokens.
- Apply the principle of least privilege.
- Regularly test authorization logic with different user roles.

## Tools & Payloads

- Burp Suite Repeater and Intruder for testing ID changes and verb tampering
- curl for quick manual tests and scripting mass access
- jq for parsing JSON API responses
- Hashing/encoding tools: `base64`, `md5sum`, CyberChef for recreating client-side transformations
- Scripts for enumeration (see examples above)

**Example hashed IDOR recreation:**
```bash
echo -n 1 | base64 -w 0 | md5sum | tr -d ' -'
```

See also the dedicated [Authentication](broken-authentication.md) page for related session and auth bypass techniques that can compound access control issues.
