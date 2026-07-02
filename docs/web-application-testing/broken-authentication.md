# Broken Authentication

## What is Broken Authentication?

Broken authentication refers to flaws in the implementation of authentication mechanisms that allow attackers to bypass or defeat authentication controls, leading to unauthorized access or account compromise.

## Exploits

- [User Enumeration and Brute Force Attacks](broken-authentication/user-enumeration-brute-force.md)
- [Password Reset Attacks](broken-authentication/password-reset-attacks.md)
- [MFA Attacks](broken-authentication/mfa-attacks.md)
- [Authentication Bypasses](broken-authentication/authentication-bypasses.md)
- [Session Attacks](broken-authentication/session-attacks.md)

## Attack Surface

Authentication mechanisms and related functionality present multiple areas that can be targeted:

- Login and authentication endpoints
- Password reset and account recovery flows
- Multi-factor / 2FA verification steps
- Session management and token handling (cookies, tokens)
- Protected pages, resources, and admin interfaces
- API authentication endpoints
- "Remember me" or persistent login features

Weaknesses often arise from insufficient server-side validation, predictable or low-entropy tokens, lack of rate limiting, overly verbose error messages, or missing access checks that rely only on client-side redirects.

## Identification

- Analyze login, reset, or MFA responses for differences that indicate valid vs. invalid usernames/accounts (varying error messages, HTTP status codes, response delays/timing, or account lockout behavior).
  - Status code changes between valid and invalid usernames.
  - Different (sometimes subtle) error messages.
  - Delayed responses for certain inputs.
  - Correct usernames may receive a lockout message after a certain number of failed login attempts (while invalid usernames do not).
- Probe for absence of rate limiting or weak protections on authentication actions such as login attempts, password resets, and 2FA code submission.
  - Correct/successful login details may receive a different rate limit response (or none) compared to failed attempts.
- Examine session cookies or other tokens for patterns: insufficient entropy, sequential/incrementing values, encoding of user data (e.g. base64), or large fixed portions with only small varying components.
  - Check "stay logged in" or persistent cookies to see if they are easily decrypted or contain predictable data.
- Attempt direct navigation to protected URLs and observe whether authentication/authorization is properly enforced server-side.
- Review password reset and recovery flows for guessable tokens, logic issues (e.g. token not required after initial step), or information leakage.
  - Check if a reset URL token is easily decrypted or predictable.
  - Verify whether the reset token is actually required to complete the password reset after the initial link is followed.
  - Test for different error messages when new password fields do not match (these differences can sometimes be abused to brute-force values for a victim's account during reset).
- Test whether client-side controls, parameter values, or response status codes can be manipulated to bypass authentication.

## Impact

Successful exploitation of broken authentication can result in:

- Complete compromise of user accounts
- Unauthorized access to sensitive data and application functionality
- Privilege escalation (e.g. from regular user to administrator)
- Session hijacking and full user impersonation
- Circumvention of other security controls

## Prevention

- Enforce strong password policies (length, complexity) and support the use of password managers.
- Implement rate limiting and account lockouts for authentication attempts, password resets, and MFA, while ensuring responses do not leak whether an account is valid.
- Require and correctly implement multi-factor authentication (MFA/2FA) for sensitive accounts or actions.
- Use cryptographically secure, high-entropy session identifiers. Regenerate the session ID after successful login and on privilege elevation. Apply `HttpOnly`, `Secure`, and `SameSite` cookie attributes. Enforce reasonable idle and absolute timeouts.
- Protect password reset and recovery flows with high-entropy, single-use, time-limited tokens delivered out-of-band. Avoid (or strongly secure) security questions.
- Do not differentiate error messages or behavior between valid and invalid usernames or credentials.
- Perform all authentication and authorization decisions server-side. Never trust client-side logic, hidden fields, or simple redirect responses (e.g. 302) for protection.
- Ensure logout fully invalidates sessions server-side.
- Prefer well-tested authentication libraries/frameworks over custom implementations.
- Consider additional controls such as device fingerprinting, IP reputation, or anomaly detection for high-value applications.

## Tools & Payloads

### Enumeration and Brute Forcing with ffuf

User enumeration, password brute-forcing, reset token brute-forcing, and 2FA brute-forcing examples are shown in the Exploitation section.

### Generating Sequential Test Values

```bash
seq -w 0 9999 > tokens.txt # -w pads numbers with prepending zeros (useful for 4-digit codes)
seq 1 1000 > user_ids.txt
```

### Filtering Wordlists to Match Password Policies

```bash
grep '[[:upper:]]' /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt | grep '[[:lower:]]' | grep '[[:digit:]]' | grep -E '.{10}' > custom_wordlist.txt
```

### Analyzing / Decoding Session Tokens

```bash
echo -n dXNlcj1odGItc3RkbnQ7cm9sZT11c2Vy | base64 -d
echo -n 'user=htb-stdnt;role=admin' | base64 # base64 encode
echo -n 'user=htb-stdnt;role=admin' | xxd -p # hex encode
```

### Password Reset / Security Question Wordlist Example

```bash
cat world-cities.csv | cut -d ',' -f1 > city_wordlist.txt # all world cities
cat world-cities.csv | grep Germany | cut -d ',' -f1 > german_cities.txt # just german cities
```

### Useful Resources for Default Credentials

- [CIRT.net](https://cirt.net/passwords/)
- SecLists Default Credentials (`SecLists/Passwords/Default-Credentials/`)
- [SCADAPASS](https://github.com/scadastrangelove/SCADAPASS/tree/master)

### Common Testing Tools

- `ffuf` (fuzzing/brute force)
- `seq` (generate number sequences)
- `grep` (wordlist filtering)
- Burp Suite / browser dev tools (intercept and modify responses, analyze cookies, tamper with MFA/verify cookies, test host headers)
- Base64 / hex / other decoding tools (for token analysis, including stay-logged-in and reset tokens)
- Manual header manipulation (X-Forwarded-For, X-Forward-Host) for rate limit evasion and password reset poisoning

### Types of Brute Forcing (Authentication Context)

| Attack Type          | Description                                                                 | Best Used When |
|----------------------|-----------------------------------------------------------------------------|----------------|
| Simple Brute Force   | Tries every possible character combination                                 | No prior password information |
| Dictionary Attack    | Uses pre-compiled lists of common passwords                                 | Weak or patterned passwords likely |
| Hybrid Attack        | Dictionary + mutations (numbers/symbols)                                    | Slightly modified common passwords |
| Credential Stuffing  | Uses leaked credentials from other breaches                                 | Password reuse suspected |
| Password Spraying    | Common passwords across many accounts                                       | Lockout policies in place |
| Reverse Brute Force  | Known password against many usernames                                       | Password reuse across accounts |

**Default Credentials Resources**:
- Common device defaults (Linksys/D-Link/Netgear admin/admin or admin/password, etc.)
- SecLists `Passwords/Default-Credentials/`
- SCADAPASS and similar lists for specialized systems

Fold these into rate limit and password reset testing where relevant.
