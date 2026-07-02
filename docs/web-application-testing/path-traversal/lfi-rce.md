# Path Traversal - LFI to RCE

**Title:** LFI Chained to Remote Code Execution

**Severity:** Critical

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 9.8

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H

**Vulnerability Type:** Path Traversal

**Target:** LFI + upload or writable files

## Impact

Code execution.

## Description

Include uploaded or poisoned file.

## Steps to Reproduce

Upload shell, LFI it.

Or poison logs with PHP code then include log.

## Recommendation

As above.

## References

HackTricks LFI RCE.
