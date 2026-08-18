# Attacking Web Applications with Ffuf

## Flags

| Flag    | Use                            | Example                     |
|---------|--------------------------------|-----------------------------|
| -u      | Specify target URL             | -u http://example.com/FUZZ  |
| -w      | Set wordlist file              | -w wordlist.txt             |
| -ic     | Ignore wordlist comments       | -ic                         |
| -H      | Add custom HTTP headers        | -H "Authorization: Bearer token" |
| -X      | Set HTTP method                | -X POST                     |
| -e      | Extend wordlist with extensions | -e .php,.html              |
| -s      | Enable silent mode             | -s                          |
| -v      | Increase verbosity             | -v                          |
| -t      | Set number of threads          | -t 50                       |
| -k      | Ignore SSL/TLS errors          | -k                          |
| -o      | Output results to file         | -o results.txt              |
| -timeout | Set request timeout in seconds | -timeout 30                |
| -recursion | Enable recursive directory scanning | -recursion          |
| -recursion-depth | Set maximum recursion depth   | -recursion-depth 2  |
| -s      | Filter by status codes         | -s 200,404                  |
| -mc     | Match by status codes          | -mc 200                     |
| -ml     | Match by line count            | -ml 50                      |
| -mw     | Match by word count            | -mw 100                     |
| -ms     | Match by size in bytes         | -ms 1024                    |
| -fc     | Filter by status codes         | -fc 404                     |
| -fl     | Filter by line count           | -fl 0                       |
| -fw     | Filter by word count           | -fw 0                       |
| -fs     | Filter by size in bytes        | -fs 512                     |
| -ac     | Automatically calibrate filtering | -ac                      |

## Commands

| Command | Description |
|---------|-------------|
| `ffuf -h` | Display ffuf help menu |
| `ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ` | Directory Fuzzing to discover hidden directories |
| `ffuf -w /usr/share/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://SERVER_IP:PORT/indexFUZZ` | Extension Fuzzing to identify file types |
| `ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/blog/FUZZ.php` | Page Fuzzing to find dynamic pages |
| `ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u https://FUZZ.hackthebox.eu/` | Sub-domain Fuzzing to enumerate subdomains |
| `ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb' -fs xxx` | VHost Fuzzing to detect virtual hosts, filtering by size |
| `ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs xxx` | Parameter Fuzzing - GET to find injectable parameters, filter by size |
| `ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx` | Parameter Fuzzing - POST to test POST-based parameters |
| `ffuf -w ids.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx` | Value Fuzzing to test parameter value vulnerabilities |
| `ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ -recursion -recursion-depth 1 -e .php -v` | Recursive Fuzzing to explore subdirectories with verbose output |
| `ffuf -w /usr/share/SecLists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://faculty.academy.htb:STMPO/FUZZ -recursion -recursion-depth 1 -e .php,.phps,.php7 -fs 287 -mr "You don't have access!" -t 100` | Recursive Fuzzing to explore subdirectories, check for variation of the php extensions, match a specific response, 100 threads |
| `ffuf -w /usr/share/SecLists/Usernames/Names/names.txt:FUZZ -u http://faculty.academy.htb:STMPO/courses/linux-security.php7 -X POST -d 'username=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs 781` | Fuzzing the username post parameter and filter response size |

## Wordlists

## Common Wordlists

| Wordlist                                      | Description            |
|-----------------------------------------------|------------------------|
| `/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt` | Directory/Page Wordlist |
| `/usr/share/seclists/Discovery/Web-Content/web-extensions.txt` | Extensions Wordlist    |
| `/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt` | Domain Wordlist         |
| `/usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt` | Parameters Wordlist     |

[Seclists](https://github.com/danielmiessler/SecLists)

Create a sequence wordlist for value fuzzing:
```bash
for i in $(seq 1 1000); do echo $i >> ids.txt; done
seq -w 0 9999 > tokens.txt # -w pads numbers with prepending zeros
```

## Misc

| Command | Description |
|---------|-------------|
| `sudo sh -c 'echo "SERVER_IP academy.htb" >> /etc/hosts'` | Add a DNS entry to resolve custom domains |
| `curl http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=key' -H 'Content-Type: application/x-www-form-urlencoded'` | Example curl command with POST request |