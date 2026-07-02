# CSRF

## What is CSRF?

Cross-Site Request Forgery (CSRF) is an attack that forces an authenticated user to execute unwanted actions on a web application in which they are currently authenticated.

## Exploits

- [Basic CSRF Form Submission](csrf/basic-csrf.md)
- [CSRF with Weak Token Bypass](csrf/weak-token.md)
- [Host Header Poisoning in Password Reset](csrf/host-header.md)

## Impact

Unauthorized changes to user accounts or settings, financial transactions, privilege escalation, data deletion.

## Prevention

Use synchronizer tokens, SameSite cookies, custom request headers for AJAX.

## Tools & Payloads

- Burp Suite to capture and analyze requests/tokens
- Simple HTML + JavaScript proof-of-concept pages
- MD5 or other hash libraries when testing weak token generation

See the XSS page for powerful chaining opportunities and the Broken Authentication page for related session issues.
