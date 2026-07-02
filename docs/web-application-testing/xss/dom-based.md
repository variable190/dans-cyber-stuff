# XSS - DOM-based Cross-Site Scripting

**Title:** DOM-based XSS in Client-Side JavaScript

**Severity:** High

**Impact:** High

**Exploitability:** High

**CVSS Base Score:** 6.1

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N

**Vulnerability Type:** Cross-Site Scripting

**Target:** Client-side code that unsafely handles URL or input data

## Impact

Execution in the DOM without server reflection, hard to detect server-side.

## Description

Vulnerability in client-side JS processing input unsafely (e.g. location.hash, document.write).

## Steps to Reproduce

Inject via URL fragment or parameter that JS reads:

#<script>alert(1)</script>

Or other DOM sinks.

## Recommendation

Avoid unsafe DOM APIs with user data. Use textContent, sanitize inputs.

## References

OWASP DOM XSS and PortSwigger DOM-based XSS.