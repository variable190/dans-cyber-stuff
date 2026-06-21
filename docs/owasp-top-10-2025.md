# OWASP Top 10 2025

## A01:2025 - Broken Access Control
- Test for IDOR by modifying user IDs, object references in requests
- Check horizontal/vertical privilege escalation (e.g., admin functions as low-priv user)
- Bypass role checks via parameter tampering or forced browsing
- Test direct object references without proper authorisation
- Verify if SSRF or path traversal allows unauthorised access

## A02:2025 - Security Misconfiguration
- Scan for default credentials on admin panels, databases, or services
- Check for exposed error messages, stack traces, or debug modes
- Test cloud configs (S3 buckets, IAM roles, CORS policies)
- Look for unnecessary features enabled (directory listing, HTTP methods)
- Verify security headers missing (CSP, HSTS, X-Frame-Options)

## A03:2025 - Software Supply Chain Failures
- Identify dependencies via npm, Maven, or package.json files
- Check for vulnerable/outdated libraries using Retire.js or OWASP Dependency-Check
- Test for compromised build pipelines or CI/CD exposure
- Look for untrusted third-party components or malicious updates
- Verify code signing and integrity checks in deployment

## A04:2025 - Cryptographic Failures
- Test for weak TLS/SSL configs (expired certs, weak ciphers) with testssl.sh
- Check for sensitive data in transit (HTTP instead of HTTPS)
- Look for weak hashing (MD5, SHA1) or unsalted passwords
- Test client-side crypto (e.g., JWT none algorithm, weak keys)
- Verify proper key management and random number generation

## A05:2025 - Injection
- Test SQLi in login forms, search fields, and parameters (union, blind)
- Check for command injection in file uploads or system calls
- Test NoSQL injection in MongoDB/JSON endpoints
- Look for LDAP, XPath, or code injection vectors
- Use payloads from SQLMap or Burp Intruder

## A06:2025 - Insecure Design
- Analyse business logic for flaws (e.g., missing rate limits on sensitive actions)
- Test for race conditions or improper workflow enforcement
- Check if design allows insecure defaults or bypassable flows
- Verify lack of threat modelling in features like password reset
- Look for insufficient input validation in multi-step processes

## A07:2025 - Authentication Failures
- Test for weak password policies, brute-force, and credential stuffing
- Check session management (fixation, hijacking, improper logout)
- Verify MFA bypass or weak token implementation
- Look for insecure password recovery (predictable tokens)
- Test for broken authentication in APIs or OAuth flows

## A08:2025 - Software or Data Integrity Failures
- Test for insecure deserialization (Java, PHP, .NET gadgets)
- Check unsigned or tampered software updates/auto-updates
- Look for CI/CD pipeline tampering or artifact poisoning
- Verify lack of integrity checks on uploads or data in transit
- Test for XML External Entity (XXE) or unsafe file handling

## A09:2025 - Security Logging and Alerting Failures
- Check if actions (logins, failures, changes) are logged properly
- Test for missing or insufficient logging of security events
- Verify rate-limiting or alerts on suspicious activity
- Look for log injection or tampering possibilities
- Check if logs expose sensitive data

## A10:2025 - Mishandling of Exceptional Conditions
- Test error handling for information leaks (stack traces, paths)
- Check for improper exception handling leading to DoS or bypasses
- Verify "fail open" scenarios in auth or access controls
- Look for unhandled edge cases in business logic
- Test for resource exhaustion via error loops or infinite exceptions