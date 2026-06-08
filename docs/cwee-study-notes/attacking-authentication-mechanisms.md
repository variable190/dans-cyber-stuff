# Attacking Authentication Mechanisms

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

