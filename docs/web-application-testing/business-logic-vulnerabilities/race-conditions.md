# Business Logic Vulnerabilities - Race Conditions

**Title:** Race Condition in Limited Resource Actions

**Severity:** High

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:N

**Vulnerability Type:** Business Logic

**Target:** Actions with limited resources (e.g. coupon use, balance transfer)

## Impact

Exploit limited actions multiple times or in inconsistent state.

## Description

Lack of proper locking or sequencing allows concurrent requests to succeed when they should not.

## Steps to Reproduce

Use multiple concurrent requests (e.g. with Burp or scripts) to the same limited action.

## Recommendation

Use database transactions, locks, or idempotency keys for critical actions.

## References

PortSwigger Race Conditions.
