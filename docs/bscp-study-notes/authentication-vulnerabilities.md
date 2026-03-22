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

