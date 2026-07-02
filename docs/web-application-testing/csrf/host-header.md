# CSRF - Host Header Poisoning in Password Reset

**Title:** CSRF via Host Header Poisoning on Password Reset

**Severity:** High

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:N/I:H/A:N

**Vulnerability Type:** Cross-Site Request Forgery

**Target:** Password reset functionality

## Impact

Unauthorized password reset for victims, leading to account takeover.

## Description

The reset link/token delivery can be poisoned using the Host or X-Forward-Host header, causing the reset to be sent to an attacker-controlled server. Combined with other bypasses like missing token validation on final step.

## Steps to Reproduce

### Host header poisoning on reset

Set the `X-Forward-Host` header (pointing to an attacker-controlled server) when submitting a password reset request. This can cause the reset link/token to be sent to the attacker's server instead of (or in addition to) the legitimate user.

### Additional related bypasses

- Submit empty or missing token values.
- Use a token from a different account if validation is insufficient.
- Change request method (POST → GET).
- Remove the token parameter entirely.
- For double-submit cookie patterns: set a known cookie value and matching parameter.
- Bypass Referer checks by manipulating the header or using meta tags: `<meta name="referrer" content="no-referrer">`
- Open redirect chaining to control where the victim is sent.

## Recommendation

Do not rely on user-controlled headers (Host, X-Forward-Host) for generating reset links. Always validate tokens server-side on every step. Use out-of-band delivery for sensitive tokens.

## References

- OWASP CSRF Prevention Cheat Sheet
- PortSwigger Authentication vulnerabilities
