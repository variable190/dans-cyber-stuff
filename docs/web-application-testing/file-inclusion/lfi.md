# File Inclusion - Local File Inclusion

**Title:** LFI to Read Files

**Severity:** High

**Impact:** High

**Exploitability:** High

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N

**Vulnerability Type:** File Inclusion

**Target:** Include parameter

## Impact

Read sensitive files.

## Description

User input used in include without validation.

## Steps to Reproduce

?page=../../../etc/passwd

## Recommendation

Whitelists, no user input in includes.

## References

PayloadsAllTheThings.
