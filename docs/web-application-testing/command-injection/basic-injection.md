# Command Injection - Basic Command Injection

**Title:** OS Command Injection via User Input

**Severity:** Critical

**Impact:** High

**Exploitability:** High

**CVSS Base Score:** 9.8

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H

**Vulnerability Type:** Command Injection

**Target:** Input fields that trigger system commands (e.g. ping tool)

## Impact

Full remote code execution on the server with the privileges of the web application process. This can lead to complete server compromise.

## Description

The application passes unsanitized user input directly to operating system commands using functions like system(), exec(), or shell_exec().

## Steps to Reproduce

Test with command separators:

- `& echo aiwefwlguh &`
- `; id`
- `|| whoami ||`

Useful commands table from testing:

| Purpose              | Linux          | Windows             |
|----------------------|----------------|---------------------|
| Current user         | whoami         | whoami              |
| OS version           | uname -a       | ver                 |
| Network config       | ifconfig       | ipconfig /all       |
| Network connections  | netstat -an    | netstat -an         |
| Running processes    | ps -ef         | tasklist            |

## Recommendation

Avoid calling OS commands from application code. Use safe APIs instead. If unavoidable, use strict whitelisting and proper escaping.

## References

- PayloadsAllTheThings Command Injection
- OWASP Command Injection
