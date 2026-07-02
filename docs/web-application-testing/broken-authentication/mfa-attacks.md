# Broken Authentication - MFA Attacks

**Title:** Multi-Factor Authentication Bypass via Cookie Manipulation and Brute Force

**Severity:** High

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N

**Vulnerability Type:** Broken Authentication

**Target:** MFA verification step

## Impact

Bypass of multi-factor protection allowing full account access even when strong passwords are in use.

## Description

MFA flows can be bypassed by manipulating cookies to redirect verification codes or by brute forcing low-entropy codes when rate limiting is weak or bypassable.

## Steps to Reproduce

### Bypass MFA via cookie manipulation

1. Initiate login for target account.
2. Intercept and modify the "verify" or MFA state cookie to point the code to attacker-controlled destination.
3. Brute force the code (see 2FA brute force in user enumeration report).

### Brute Forcing 2FA

```bash
seq -w 0 9999 > tokens.txt
ffuf -w ./tokens.txt -u http://bf_2fa.htb/2fa.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=fpfcm5b8dh1ibfa7idg0he7l93" -d "otp=FUZZ" -fr "Invalid 2FA Code"
```

## Recommendation

Properly validate MFA state server-side on every step. Use high-entropy, time-limited codes with strong rate limiting. Never trust client-side cookies or state for completing MFA.

## References

- OWASP Authentication Cheat Sheet
- PortSwigger: Authentication vulnerabilities
