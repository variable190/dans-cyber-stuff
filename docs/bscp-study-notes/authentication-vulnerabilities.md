# Authentication vulnerabilities

## Username Enumeration

- Status code changes
- Different error messages (can be subtle)
- Delayed responses
- Correct usernames may recieve a lockout message after a certain number of failed login attempts

## Brute force evasion

- Randomise X-Forward-For header IP if rate limited for one IP
- Iterate between successful login and bruteforce attempt
- Correct login details may recieve a different or no login rate limit message

## Bypass MFA

- Adjust verify cookie to target to send code to them, then bruteforce code

## Cookies

- Check if stay logged in cookie is easily decrypted
- XSS to gain the users session cookie

## Password reset

- Check if reset URL token is easily decrypted
- Check if reset URL token is required to reset password, after following URL
- Set X-Forward-Host header to your own vulnerable server and send password reset request to target
- Test if different error messages are sent when new password fields do not match, could be used to brute force a victim