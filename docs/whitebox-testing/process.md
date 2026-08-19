# Whitebox Pentesting Process Summary

1. Code review
2. Local testing
3. PoC
4. Patching and remediation

## Code Review

1. **Planning and gathering**
    - Gain an understanding of the app design/functionality from the devs/stakeholders
    - Gather required/available assets/access
        - Source Code
        - Backend server
        - DB
        - Server logs
        - User roles/privileges
        - Documentation
        - Test server
2. **Scope selection**
    - Select functions
        - By app design
        - Using search
        - By using the app
3. **Reverse engineering**
    - Trace variables and functions
    - Comment what each line/function/parameter does
4. **Prioritisation**
    - Prioritise functions by sensitivity
    - Priotitise what you're most skilled at
    - Use the impact x probability risk matrix

## Local testing

1. **Backend replication**
    - If provided use a test server
    - Consult documentation if available
    - Use a dependcies file
    - Examine codebase to identify dependencies
2. **Testing**
    - Trigger the target function
    - Trace the input
    - Control how the input reaches/affects the target function
    - Record if the target is vulnerable or not
3. **Exploitation**
    - Confirm basic exploitation of the target function
    - No need to overcome security filters at this stage

## Proof of Concept

1. **Full chain exploitation**
    - Document step-by-step process
    - Bypass any restrictions
    - Record
        - Initial target location
        - Client-side payload
        - Any other payloads for chained vulnerabilites
        - Bypasses in payloads or web requests
2. **Automate exploit where possible**
    - Python:
        - Network/web application
        - Binary
    - JavaScript:
        - Client-side
    - Python & Javascript
        - Chained web and client-side
    - Bash/PowerShell/CMD
        - Operating system
3. **Test on real target**
    - Ensure all actions can be reverted
        - Delete any created accounts
        - Reset any modified data (passwords etc)
        - Clean any traces of backend/DB interaction
        - Handle errors cleanly, reverting any changes up to the break point
    - DO NOT MODIFY CRITICAL DATA
    - DO NOT CAUSE DOWNTIME, DATA LOSS OR PERMANENT MODIFICATION

## Patching and remediation

1. **Patching**
    - Patch all found vulnerabilities in the test environment where possible
    - Retest the PoC exploit confirming vulnerabilities patched at all stages
    - Ensure patches do not break original functionality
2. **Reporting**
    - Use standard template
    - Highlight vulnerable functions
    - Provide the patches, documenting what was modified
    - Confirm whether patches were fully tested or not
    - Provide secure coding tips