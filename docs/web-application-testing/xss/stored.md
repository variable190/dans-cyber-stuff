# XSS - Stored Cross-Site Scripting

**Title:** Stored XSS in User-Generated Content

**Severity:** High

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 6.1

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N

**Vulnerability Type:** Cross-Site Scripting

**Target:** Comments, profiles, posts or any stored user input displayed to others

## Impact

Persistent execution of malicious scripts for all users viewing the content, leading to mass session hijacking or phishing.

## Description

Malicious input is saved on the server and served without proper encoding to other users.

## Steps to Reproduce

Submit payload in a stored field:

<script>alert(document.cookie)</script>

Or more advanced for data theft.

View as another user or admin.

## Recommendation

Sanitize all stored user input with libraries like DOMPurify. Encode output. Use CSP.

## References

PayloadsAllTheThings XSS section and OWASP XSS Prevention.