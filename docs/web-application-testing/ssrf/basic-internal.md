# SSRF - Basic Internal Service Access

**Title:** SSRF to Access Internal Admin Panel

**Severity:** High

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N

**Vulnerability Type:** Server-Side Request Forgery

**Target:** URL parameter used to fetch remote content

## Impact

Access to internal resources and services not intended to be exposed.

## Description

The application fetches content from user-supplied URLs without validation, allowing requests to internal IPs and services.

## Steps to Reproduce

Change protocols (`file://`, `gopher://`) and observe behavior.

Example:
```
http://127.0.0.1:8080/admin
file:///etc/passwd
```

## Recommendation

Whitelist allowed destinations and schemes. Disable file:// and other dangerous protocols.

## References

- OWASP SSRF
