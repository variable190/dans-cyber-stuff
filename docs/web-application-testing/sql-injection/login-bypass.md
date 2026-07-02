# SQL Injection - Login Bypass

**Title:** SQL Injection Login Bypass on Authentication Endpoint

**Severity:** High

**Impact:** High

**Exploitability:** High

**CVSS Base Score:** 8.1

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:N

**Vulnerability Type:** SQL Injection

**Target:** Login form (username and password parameters)

## Impact

Complete authentication bypass allowing attackers to access any user account, including administrator accounts, without knowing the password. This can lead to full application compromise, data theft, and unauthorized actions.

## Description

The login functionality is vulnerable to SQL injection due to unsanitized user input being directly concatenated into SQL queries. An attacker can use a single quote and SQL comment to bypass the password check entirely.

## Steps to Reproduce

1. Navigate to the login page.
2. In the username field, enter: `administrator'--`
3. In the password field, enter any value (e.g. `password`).
4. Submit the form.

The application will authenticate as the administrator user because the `--` comments out the password check in the query.

Example payload:
```
administrator'--
```

## Recommendation

- Use parameterized queries (prepared statements) instead of string concatenation for all database queries.
- Implement proper input validation and escaping.
- Use an ORM or query builder that handles parameterization automatically.
- Enforce strong password policies and account lockouts.
- Consider multi-factor authentication.

## References

- PortSwigger SQL Injection Cheat Sheet
- OWASP SQL Injection Prevention Cheat Sheet
