# OWASP Top 10 2025

## A01:2025 - Broken Access Control
Applications fail to properly restrict what authenticated users are allowed to do or access.

**Typical Exploits/Issues**

- IDOR / BOLA / BFLA
- Path traversal / LFI
- HTTP verb tampering / method bypass
- SSRF (when used for unauthorized access)
- Privilege escalation via flawed authorization

**Relevant study notes:**

- [Path Traversal](bscp-study-notes/path-traversal.md)
- [Web Attacks](cwes-study-notes/web-attacks.md)
- [API Attacks](cwes-study-notes/api-attacks.md)
- [Server-Side Attacks](cwes-study-notes/server-side-attacks.md)

## A02:2025 - Security Misconfiguration
Security settings are incorrectly configured or left at insecure defaults.

**Typical Exploits/Issues**

- Default credentials
- Verbose error messages exposing sensitive info
- Insecure HTTP headers or missing security headers
- Unnecessary services or features enabled
- Directory listing enabled

## A03:2025 - Software Supply Chain Failures
Vulnerabilities are introduced through third-party components or compromised build and deployment processes.

**Typical Exploits/Issues**

- Vulnerable third-party libraries / dependencies
- Malicious packages in public repositories
- Compromised CI/CD pipelines
- Unsigned or unverified updates

## A04:2025 - Cryptographic Failures
Sensitive data is not adequately protected due to weak encryption or key management.

**Typical Exploits/Issues**

- Weak encryption algorithms or hashing
- Hardcoded keys or secrets
- Improper certificate validation (e.g. SSL pinning bypass)
- Sensitive data transmitted without encryption
- Poor key management

## A05:2025 - Injection
User input is not properly validated or sanitised before being processed by an interpreter.

**Typical Exploits/Issues**

- SQL Injection
- Command / OS Command Injection
- NoSQL Injection
- XXE (XML External Entity Injection)
- File Inclusion (LFI / RFI)
- Cross-Site Scripting (XSS)

**Relevant study notes:**

- [SQL Injection](bscp-study-notes/sql-injection.md)
- [SQL Injection Fundamentals](cwes-study-notes/sql-injection-fundamentals.md)
- [SQLMap Essentials](cwes-study-notes/sqlmap-essentials.md)
- [Command Injection](bscp-study-notes/command-injection.md)
- [Command Injections](cwes-study-notes/command-injections.md)
- [Injection Attacks](cwee-study-notes/injection-attacks.md)
- [Introduction to NoSQL Injection](cwee-study-notes/introduction-to-nosql-injection.md)
- [File Inclusion](cwes-study-notes/file-inclusion.md)
- [Cross-Site Scripting (XSS)](cwes-study-notes/cross-site-scripting-(xss).md)
- [Cross-Site Scripting](bscp-study-notes/cross-site-scripting.md)

## A06:2025 - Insecure Design
Fundamental flaws in the application's design and business logic cannot be fixed by secure implementation alone.

**Typical Exploits/Issues**

- Flawed business logic and workflow bypasses
- Insecure direct object references by design
- Missing security controls at the design/architecture level
- Overly trusting client-side data or inputs in core processes

**Relevant study notes:**

- [Business Logic Vulnerabilities](bscp-study-notes/business-logic-vulnerabilities.md)

## A07:2025 - Authentication Failures
Weak authentication and session management allow user accounts to be compromised.

**Typical Exploits/Issues**

- Username / account enumeration
- Brute force and credential stuffing
- Broken session management (fixation, hijacking)
- JWT vulnerabilities (none algorithm, weak secrets, etc.)
- Weak or missing MFA
- Flawed password reset / account recovery flows

**Relevant study notes:**

- [Authentication Vulnerabilities](bscp-study-notes/authentication-vulnerabilities.md)
- [Broken Authentication](cwes-study-notes/broken-authentication.md)
- [Attacking Authentication Mechanisms](cwee-study-notes/attacking-authentication-mechanisms.md)
- [Session Security](cwes-study-notes/session-security.md)
- [Login Brute Forcing](cwes-study-notes/login-brute-forcing.md)

## A08:2025 - Software or Data Integrity Failures
Critical software and data lack proper integrity verification.

**Typical Exploits/Issues**

- Insecure deserialization leading to object manipulation
- Unrestricted file uploads allowing malicious content
- Lack of integrity checks on software updates or data
- Tampering with serialized data or uploaded files

**Relevant study notes:**

- [File Upload Attacks](cwes-study-notes/file-upload-attacks.md)

## A09:2025 - Security Logging and Alerting Failures
Security-relevant events are not adequately logged, monitored or alerted on.

**Typical Exploits/Issues**

- Insufficient logging of authentication, authorization, and security events
- Lack of monitoring and alerting for suspicious activities
- Logs missing critical context for incident response

## A10:2025 - Mishandling of Exceptional Conditions
Errors and exceptional conditions are handled in ways that create vulnerabilities or leak information.

**Typical Exploits/Issues**

- Verbose error messages leaking stack traces, database details, or internal paths
- Improper exception handling that enables control flow bypasses
- Failure to handle edge cases or malformed input securely (e.g. null bytes, oversized inputs)
