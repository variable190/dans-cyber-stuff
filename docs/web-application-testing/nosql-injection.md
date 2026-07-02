# NoSQL Injection

## What is NoSQL Injection?

NoSQL databases (such as MongoDB) use query languages and data models that differ from traditional SQL. Injection vulnerabilities arise when user input is used unsafely in database queries, often through operators that can alter the intended logic of the query (e.g., `$ne`, `$gt`, `$regex`).

## Exploits

- [Operator-based Authentication Bypass](nosql-injection/operator-bypass.md)
- [In-Band Data Extraction](nosql-injection/inband-extraction.md)
- [Blind Data Extraction](nosql-injection/blind-extraction.md)

## Attack Surface

- Login and authentication endpoints that use NoSQL backends
- Search, filter, and API parameters passed directly to database queries
- Applications using MongoDB, CouchDB, or similar with insufficient input handling

## Identification

- Test for operator injection by supplying JSON-like structures or URL-encoded equivalents.
- Look for differences in responses when using comparison or logical operators.
- Test authentication forms with operators that should return unexpected results (e.g., matching any document).
- Use regex or other evaluation operators to perform blind or in-band data extraction.

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
