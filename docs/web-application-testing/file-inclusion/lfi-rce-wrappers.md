# File Inclusion - LFI to RCE via Wrappers

**Title:** PHP Wrappers for LFI to RCE

**Severity:** Critical

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 9.8

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H

**Vulnerability Type:** File Inclusion

**Target:** LFI with wrappers enabled

## Impact

RCE.

## Description

Use php:// or phar wrappers.

## Steps to Reproduce

Combine with upload phar or filter.

## Recommendation

Disable wrappers.

## References

HackTricks.
