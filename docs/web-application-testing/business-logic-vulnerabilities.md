# Business Logic Vulnerabilities

## Overview

Business logic vulnerabilities arise when an application fails to properly enforce its intended business rules, workflows, or constraints. Unlike technical vulnerabilities (e.g., injection flaws), these often stem from flawed assumptions in how the application should behave under different conditions. Attackers exploit them by manipulating inputs, skipping steps, or abusing edge cases to achieve unintended outcomes such as unauthorized discounts, privilege changes, or data manipulation.

These issues can exist even in applications with strong authentication and input validation because the logic itself is the problem.

## Attack Surface

Business logic flaws commonly appear in:

- Multi-step processes (e.g., shopping carts, checkout flows, account registration, password resets)
- Financial or transactional operations (pricing, transfers, quantity limits)
- Role-based or state-based workflows
- Email or notification handling
- API endpoints that assume certain sequences or validations occur on the client or previous steps

## Identification

- Analyze the application's intended workflow and look for places where assumptions are made about user behavior or data.
- Intercept requests and experiment with unconventional values (negative numbers, extremely large values, unexpected data types).
- Remove or reorder parameters in requests to see if backend logic still accepts the action.
- Test forced browsing: navigate directly to later steps in a process without completing prerequisites.
- Look for locations where user input is encoded/decoded — this can sometimes leak or be abused for other purposes like tokens.
- Monitor for discrepancies between client-side limits and server-side enforcement.

## Exploitation

### Manipulating Data and Limits

- Change product IDs, prices, quantities, or other values in intercepted requests.
- Submit unconventional values:
  - Negative values (e.g., negative transfer amounts)
  - Exceptionally high values
  - Check what happens when limits are reached or exceeded.
- Repeatedly add the maximum allowed quantity using tools like Burp Intruder or Repeater to exceed intended stock or limits.
- Create accounts or records with extremely long values to test for truncation or buffer issues.
- Test whether changing an email address (or other attribute) to one associated with higher privileges grants access without proper verification.

### Workflow and Sequence Bypass

- Remove one parameter at a time from requests and observe if the application still processes the action.
- Navigate the site in unexpected ways (forced browsing). Example: directly access "order confirmed" after adding to cart without going through payment.
- Drop or skip intermediate requests (e.g., skip role selector steps).
- Test second-order effects: data stored in one place (e.g., username set to a path traversal string) may be processed unsafely elsewhere.

### Email Parser and Encoding Discrepancies

Certain email parsers can be abused for spoofing or bypasses. Test various encodings:

```html
=?iso-8859-1?q?=61=62=63?=foo@ginandjuice.shop
=?utf-8?q?=61=62=63?=foo@ginandjuice.shop
=?utf-7?q?&AGEAYgBj-?=foo@ginandjuice.shop
```

See the [PortSwigger research on email atom splitting](https://portswigger.net/research/splitting-the-email-atom) for more details and examples.

## Impact

Successful exploitation can lead to:

- Unauthorized financial transactions or discounts
- Privilege escalation or account takeover via workflow abuse
- Data leakage or corruption
- Circumvention of business rules (e.g., exceeding purchase limits, bypassing approvals)
- Reputational and monetary damage to the organization

## Prevention

- Implement all critical business rules and validations on the server side — never trust the client.
- Enforce strict state management and sequence checks for multi-step processes.
- Use server-side authorization checks at every sensitive action, not just at entry points.
- Validate and sanitize all inputs against business constraints (ranges, formats, relationships between fields).
- Perform thorough threat modeling during design to identify assumptions that could be abused.
- Consider using established frameworks that provide built-in protections for common workflows.
- Log and monitor for anomalous sequences or out-of-order actions.

## Tools & Payloads

- Burp Suite (Repeater for manual testing, Intruder for automation of repeated actions or parameter tampering)
- Browser developer tools for observing client-side assumptions
- Custom scripts for mass enumeration or forced browsing
- Proxy tools to drop or reorder requests

**Tip:** When testing, always document the "happy path" first, then systematically deviate from it.
