# Broken Authentication - Password Reset Attacks

**Title:** Vulnerable Password Reset and Token Manipulation

**Severity:** High

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N

**Vulnerability Type:** Broken Authentication

**Target:** Password reset flow (security question, token, email)

## Impact

Attackers can reset passwords for other accounts, leading to complete account takeover without knowing the original credentials.

## Description

The password reset functionality is vulnerable to logic flaws, weak tokens, guessable security questions, host header poisoning, and error message differences that can be abused to brute force or bypass the reset process.

## Steps to Reproduce

### Guessable Security Questions

```bash
cat world-cities.csv | cut -d ',' -f1 > city_wordlist.txt
cat world-cities.csv | grep Germany | cut -d ',' -f1 > german_cities.txt
ffuf -w ./city_wordlist.txt -u http://pwreset.htb/security_question.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=39b54j201u3rhu4tab1pvdb4pv" -d "security_response=FUZZ" -fr "Incorrect response."
```

### Check for predictable or decryptable reset tokens

- Test if tokens in reset URLs are base64, hex, or easily reversible.
- Check if the token from the initial request is actually required on the final password submission step.

### Host Header Poisoning

Set X-Forward-Host header to attacker server when submitting reset request. The reset link may be sent to the attacker.

### Error Message Abuse

During final reset step, submit mismatched passwords and observe different errors. These can be used to brute force the new password value.

## Recommendation

Use high-entropy, single-use, time-limited tokens delivered out-of-band. Always validate the token on every step of the reset flow. Return generic error messages. Do not use user-controlled headers for delivery of reset links.

## References

- OWASP Authentication Cheat Sheet
- PortSwigger: Authentication vulnerabilities
