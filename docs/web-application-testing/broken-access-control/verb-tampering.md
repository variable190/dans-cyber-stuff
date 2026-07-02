# Broken Access Control - HTTP Verb Tampering

**Title:** HTTP Verb Tampering to Bypass Access Controls

**Severity:** Medium

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 6.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:N

**Vulnerability Type:** Broken Access Control

**Target:** Endpoints with method-specific access controls

## Impact

Bypass of access controls allowing unauthorized modification or access to data.

## Description

The application applies security controls only to certain HTTP methods (commonly GET).

## Steps to Reproduce

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

## Recommendation

Apply authorization checks regardless of HTTP method. Use a centralised authorisation layer.

## References

- OWASP Broken Access Control
