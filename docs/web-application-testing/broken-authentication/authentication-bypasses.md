# Broken Authentication - Authentication Bypasses

**Title:** Direct Access and Parameter Manipulation to Bypass Authentication

**Severity:** High

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:N

**Vulnerability Type:** Broken Authentication

**Target:** Protected pages and admin functionality

## Impact

Bypass of authentication controls allowing unauthorized access to protected resources and administrative functions.

## Description

Authentication can be bypassed by directly accessing protected URLs, manipulating status codes, or guessing/brute forcing object identifiers in parameters.

## Steps to Reproduce

### Direct Access

- Navigate directly to hidden admin page (e.g. /admin.php)
- Intercept the 302 redirect in Burp
- Change status code to 200 to access the page

### Parameter Manipulation

Log in as low-priv user and note parameter like ?user_id=183, then guess or brute force other IDs:

```bash
seq 1 1000 > user_ids.txt
ffuf -w user_ids.txt -u http://STMIP:STMPO/admin.php?user_id=FUZZ -fr "Could not load admin data."
```

## Recommendation

Perform all authorization checks server-side on every request. Do not rely on client-side redirects or status codes. Use proper access control for all sensitive functionality.

## References

- OWASP Broken Access Control
- PortSwigger: Authentication vulnerabilities
