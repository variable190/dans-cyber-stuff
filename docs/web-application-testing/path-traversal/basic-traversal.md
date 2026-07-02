# Path Traversal - Basic Directory Traversal

**Title:** Path Traversal to Read Sensitive Files

**Severity:** High

**Impact:** High

**Exploitability:** High

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N

**Vulnerability Type:** Path Traversal

**Target:** File path parameters

## Impact

Read arbitrary files including source, configs, keys.

## Description

Unsanitized path allows traversal outside webroot.

## Steps to Reproduce

?file=../../../etc/passwd

Test encodings and null bytes.

## Recommendation

Canonicalize paths, use whitelists, safe APIs.

## References

OWASP Path Traversal.
