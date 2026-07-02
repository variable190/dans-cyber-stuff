# Business Logic Vulnerabilities - Price Manipulation

**Title:** Price and Quantity Manipulation in E-commerce

**Severity:** High

**Impact:** High

**Exploitability:** High

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:N

**Vulnerability Type:** Business Logic

**Target:** Checkout or cart parameters

## Impact

Purchase items at reduced or negative prices.

## Description

Client-side values for price/quantity not validated server-side.

## Steps to Reproduce

Intercept cart update, change price or quantity parameters to negative or zero.

## Recommendation

Calculate and validate all prices server-side. Never trust client-submitted totals.

## References

PortSwigger Business Logic.
