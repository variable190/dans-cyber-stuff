# Broken Access Control - API-Specific Broken Access Control (BOLA / BFLA)

**Title:** API BOLA / BFLA - Broken Object and Function Level Authorisation

**Severity:** High

**Impact:** High

**Exploitability:** High

**CVSS Base Score:** 8.1

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:N

**Vulnerability Type:** Broken Access Control

**Target:** API endpoints (object IDs, admin functions)

## Impact

Unauthorized access to other users' objects or execution of privileged functions, leading to data exposure and privilege escalation.

## Description

APIs often lack proper object-level or function-level authorisation checks, allowing low-priv users to access or modify other users' data or call admin-only endpoints.

## Steps to Reproduce

**Broken Object Level Authorisation (BOLA / IDOR in APIs):** Authenticate as low-priv user, then iterate object IDs in API calls.

**Broken Function Level Authorisation (BFLA):** Discover admin-only endpoints and call them as a regular user.

**Broken Object Property Level Authorisation:** Modify sensitive properties (e.g., `isAdmin`, `role`) that should not be controllable by the user.

Example automation for BOLA:
```bash
for ((i=1; i<=20; i++)); do
  curl -s ... /api/v1/suppliers/quarterly-reports/$i ...
done
```

## Recommendation

Enforce authorisation at the object and function level on the server for every API operation. Validate that the authenticated user owns or is permitted to access the requested object.

## References

- OWASP Broken Access Control
- PortSwigger Access control vulnerabilities
