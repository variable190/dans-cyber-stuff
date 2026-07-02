# Business Logic Vulnerabilities - Workflow Bypass

**Title:** Bypassing Multi-Step Business Workflow

**Severity:** High

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 7.5

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:L/UI:N/S:U/C:H/I:H/A:N

**Vulnerability Type:** Business Logic

**Target:** Multi-step processes like checkout or approval flows

## Impact

Perform actions out of order or skip steps, leading to fraud or unauthorized access.

## Description

Application does not properly enforce sequence or state in business processes.

## Steps to Reproduce

Manipulate parameters or skip steps in workflow (e.g. add to cart then directly checkout with manipulated price).

Use Burp to replay or alter requests in sequence.

## Recommendation

Enforce all business rules server-side at each step. Use server-controlled state.

## References

PortSwigger Business Logic vulnerabilities.
