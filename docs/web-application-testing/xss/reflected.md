# XSS - Reflected Cross-Site Scripting

**Title:** Reflected XSS in Search Parameter

**Severity:** High

**Impact:** High

**Exploitability:** High

**CVSS Base Score:** 6.1

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:L/I:L/A:N

**Vulnerability Type:** Cross-Site Scripting

**Target:** Search or reflected parameter

## Impact

Execution of attacker-controlled JavaScript in the victim's browser in the context of the vulnerable site. Can lead to session hijacking, phishing, or defacement.

## Description

User input is reflected directly in the page without proper output encoding.

## Steps to Reproduce

Submit basic payload:
```js
<script>alert(window.origin)</script>
```

If blocked, try:
```js
<img src="" onerror=alert(window.origin)>
<script>document.body.style.background = "#141d2b"</script>
```

**Context matters:** Adapt payloads to the injection point (HTML, attribute, JS string).

## Recommendation

Encode all output according to context. Use auto-escaping frameworks. Implement strong CSP.

## References

- PayloadsAllTheThings XSS section
- PortSwigger XSS labs
