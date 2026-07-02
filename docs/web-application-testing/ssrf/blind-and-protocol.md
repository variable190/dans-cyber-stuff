# SSRF - Blind SSRF and Protocol Abuse

**Title:** Blind SSRF with File and Gopher Protocols

**Severity:** High

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N

**Vulnerability Type:** Server-Side Request Forgery

**Target:** URL fetch parameters

## Impact

Access to local files, internal services via different protocols.

## Description

Support for file:// , gopher:// etc allows reading local files or complex interactions.

## Steps to Reproduce

file:///etc/passwd

gopher:// for advanced.

Change protocols and observe behavior or callbacks.

## Recommendation

Disable unnecessary schemes in HTTP clients. Validate URLs strictly.

## References

PortSwigger SSRF and protocol abuse examples.