# Cross Site Request Forgery (CSRF)

## What is CSRF?

Where the attacker causes the victim to perform unintended actions.
This is a method of circumventing the same orign policy.

## Impact

Further compromise; change an email or password.
Potential financial loss; buy items or services with the victims saved cards or account balance.
Impersonation; publish messages or comments that look like they came from the victim.
Admin takeover; take any administrative actions available to the victim.

## How does CSRF work?

Prerequisits:
- The attacker requires knowledge of what actions the victim can perform.
- The inteded action relies solely on session cookies for authorisation.
- The attacker needs to know or beable to guess the request parameters required to take the action.

Steps:
- The attacker constructs and serves a web page that contains the malicious request.
- When the victim visits the malicious page a HTTP request is triggered to take the desired action on behalf of the victim.
- If the user is logged in to vulnerable service their browser will automatically include their session cookie in the request.
- The vulnerable website processes the request as if it was made by the victim.

Can be done with a simple get request.



## Prevention and Remediation

CSRF tokens
SameSite cookies
Referer based validation
integrate session and csrf token cookies

## Demo

### Bypasses 

Change request type
Remove csrf token parameter
Reuse own csrf token

