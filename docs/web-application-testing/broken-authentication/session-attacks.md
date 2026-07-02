# Broken Authentication - Session Attacks

**Title:** Session Hijacking, Fixation and Cookie Manipulation

**Severity:** High

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N

**Vulnerability Type:** Broken Authentication

**Target:** Session cookies and token handling

## Impact

Session hijacking, fixation, or impersonation leading to full account takeover without needing credentials.

## Description

Session tokens have insufficient entropy, are predictable, encoded, or lack proper security attributes. This allows decoding, brute forcing, fixation, or hijacking of user sessions.

## Steps to Reproduce

### Cookie Analysis and Brute Force

- Capture multiple session tokens.
- Compare for patterns (fixed parts, incremental values).
- Decode:
```bash
echo -n dXNlcj1odGItc3RkbnQ7cm9sZT11c2Vy | base64 -d
echo -n 'user=htb-stdnt;role=admin' | base64
echo -n 'user=htb-stdnt;role=admin' | xxd -p
```

- Brute force low-entropy cookies.

### Session Fixation

1. Attacker obtains valid session ID.
2. Forces victim to use it (via link or XSS).
3. Victim logs in.
4. Attacker uses the same session ID to access the account.

### Improper Session Timeout

Sessions remain valid longer than necessary after logout or idle periods.

## Recommendation

Use cryptographically secure high-entropy session IDs. Regenerate session ID on login and privilege change. Set HttpOnly, Secure, SameSite attributes. Implement proper timeouts and server-side invalidation on logout.

## References

- OWASP Session Management Cheat Sheet
- PortSwigger: Authentication vulnerabilities
- OWASP Session Fixation
