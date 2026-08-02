# Tech Interview Prep

### SQL Injection

**What is it?**  
SQL Injection happens when user-supplied data is inserted into a SQL query without proper separation of code and data. The database then executes the attacker’s input as part of the query.

**How it works**  
1. Application takes user input (e.g. username, search term).  
2. Input is concatenated directly into a SQL string.  
3. Attacker crafts input that changes the meaning of the query.  
4. Database runs the modified query.

**Common variants**  
- **Error-based**: Forces the database to return error messages that leak data.  
- **Union-based**: Uses `UNION SELECT` to pull data from other tables.  
- **Boolean-based blind**: Asks true/false questions and observes differences in the response.  
- **Time-based blind**: Uses `SLEEP()` or similar to measure response time.  
- **Out-of-band**: Forces the database to make a DNS/HTTP request to an attacker-controlled server.  
- **Second-order**: Payload is stored and only executed later in a different query.

**Impact**  
Full database compromise, data theft, authentication bypass, remote code execution (via certain database features).

**Testing**  
- Classic payloads (`' OR 1=1--`, etc.)  
- SQLMap with tamper scripts for WAF bypass  
- Manual testing for each variant  
- Look for differences in response length, timing, or content

**Prevention**  
- Parameterised queries / prepared statements (best defence)  
- Use of ORMs that parameterise by default  
- Strict input validation and allowlisting  
- Least privilege database accounts  
- Disable unnecessary database features

**Key points to remember**  
- Parameterised queries completely separate data from code.  
- Even if input is validated, parameterisation is still required.  
- Blind techniques are slower but work when no errors or data are returned.

---

### Cross-Site Scripting (XSS)

**What is it?**  
XSS occurs when an application includes untrusted data in a web page without proper validation or escaping, allowing an attacker’s JavaScript to run in a victim’s browser.

**How it works**  
1. Attacker injects malicious JavaScript into a page.  
2. Victim visits the page (or is tricked into visiting it).  
3. Browser executes the attacker’s script in the context of the vulnerable site.  
4. Script can steal cookies, modify the page, make requests as the user, etc.

**Common variants**  
- **Reflected**: Payload is in the URL and reflected immediately in the response.  
- **Stored**: Payload is saved in the database and shown to other users later.  
- **DOM-based**: Vulnerability exists entirely in client-side JavaScript (no server reflection needed).

**Impact**  
Session hijacking, account takeover, keylogging, phishing, malware distribution, defacement.

**Testing**  
- Basic payloads: `<script>alert(1)</script>`  
- Event handlers: `onerror=alert(1)`, `onload=...`  
- Filter bypass techniques (encoding, case variation, SVG, etc.)  
- DOM sinks (document.write, innerHTML, location, etc.)

**Prevention**  
- Context-aware output encoding (HTML, JavaScript, URL, CSS contexts are different)  
- Content Security Policy (CSP) – strong defence when properly configured  
- Input validation and sanitisation  
- Use of modern frameworks that auto-escape by default

**Key points**  
- HttpOnly cookies prevent XSS from stealing the session cookie, but do not stop other XSS impact.  
- CSP is one of the best modern defences if implemented strictly.  
- DOM XSS is often missed because it doesn’t appear in server responses.

---

### Insecure Direct Object Reference (IDOR) / Broken Object Level Authorisation

**What is it?**  
The application uses a user-supplied identifier (ID, username, filename, etc.) to access an object without checking whether the current user is allowed to access that specific object.

**How it works**  
1. Application receives an identifier from the user (e.g. `?id=123`).  
2. It fetches the object with that ID.  
3. No check is performed to confirm the user owns or has permission for that object.  
4. Attacker changes the ID to another user’s object.

**Common variants**  
- Horizontal (accessing other users at the same privilege level)  
- Vertical (accessing higher-privilege objects)  
- API-based BOLA (Broken Object Level Authorisation)  
- Mass assignment (binding unexpected fields)

**Impact**  
Unauthorised viewing, modification, or deletion of other users’ data. Often leads to full account takeover when combined with other issues.

**Testing**  
- Change sequential or predictable IDs  
- Test both horizontal and vertical access  
- Look for UUID or GUID that can still be enumerated  
- Try adding unexpected parameters (role, isAdmin, etc.)

**Prevention**  
- Always perform authorisation checks on the server side for every object access  
- Prefer indirect references or random UUIDs + proper authorisation  
- Never rely on client-side checks alone  
- Use access control frameworks that enforce least privilege

**Key points**  
- This is one of the most common and highest-impact bugs in modern APIs.  
- Even random IDs are not enough if there is no authorisation check.  
- Always test every endpoint that takes an ID.

---

### Server-Side Request Forgery (SSRF)

**What is it?**  
The application fetches a remote resource based on user-controlled input without proper validation, allowing an attacker to make the server request internal or external resources.

**How it works**  
1. Application takes a URL or host from the user.  
2. Server makes a request to that URL.  
3. Attacker supplies an internal address or cloud metadata endpoint.  
4. Server returns the response (or leaks information via timing/errors).

**Common variants**  
- Basic SSRF (response is returned to attacker)  
- Blind SSRF (no direct response, but side effects exist)  
- Protocol smuggling (gopher, dict, file://, etc.)  
- Cloud metadata access (AWS, GCP, Azure instance metadata)

**Impact**  
Internal network scanning, access to internal services, credential theft from cloud metadata, remote code execution in some cases.

**Testing**  
- Internal IP ranges (127.0.0.1, 169.254.169.254, 10.x.x.x, etc.)  
- Cloud metadata endpoints  
- Different protocols and ports  
- DNS rebinding and time-based techniques for blind cases

**Prevention**  
- Strict allowlisting of allowed domains/IPs  
- Network segmentation (server cannot reach internal network)  
- Disable unused URL schemes  
- Use a dedicated proxy or library that enforces restrictions

**Key points**  
- Cloud metadata is one of the highest-value targets.  
- Blind SSRF is harder but still very useful.  
- Always test both the response and side effects (DNS, timing, etc.).

---

### File Upload Vulnerabilities

**What is it?**  
The application allows users to upload files without proper validation of type, content, or storage location.

**How it works**  
1. User uploads a file.  
2. Application stores it somewhere accessible.  
3. Attacker uploads a malicious file (webshell, polyglot, etc.).  
4. Attacker requests the file and executes it.

**Common variants**  
- Unrestricted file type  
- Extension blacklist bypass (double extension, null byte, case variation)  
- Content-type spoofing  
- Path traversal in filename  
- Polyglot files (valid as multiple formats)

**Impact**  
Remote code execution, stored XSS, denial of service, malware hosting.

**Testing**  
- Try different extensions and content types  
- Double extensions (shell.php.jpg)  
- Null bytes and special characters  
- Large files and zip bombs  
- Check if the uploaded file is executable or rendered as HTML

**Prevention**  
- Validate both content type and actual file content (magic bytes)  
- Store files outside the web root  
- Randomise filenames  
- Disable script execution in upload directories  
- Scan with antivirus / malware detection

**Key points**  
- Never trust the Content-Type header or file extension alone.  
- Storing outside the web root is one of the strongest controls.  
- Even “safe” formats can be abused if they are processed (e.g. image processing libraries).

---

### Path Traversal / Local File Inclusion (LFI)

**What is it?**  
User input is used to construct a file path without proper sanitisation, allowing access to files outside the intended directory.

**How it works**  
1. Application takes a filename or path from the user.  
2. It concatenates it into a file system call.  
3. Attacker uses `../` sequences (or encodings) to escape the intended directory.  
4. Sensitive files are read or included.

**Common variants**  
- Classic directory traversal (`../../etc/passwd`)  
- Encoding bypasses (URL encoding, double encoding, Unicode)  
- Null byte injection (older systems)  
- PHP wrappers (`php://filter`, `php://input`, `data://`)  
- LFI to RCE (via log poisoning or session files)

**Impact**  
Source code disclosure, configuration file access, credential theft, remote code execution.

**Testing**  
- `../` sequences and encoded versions  
- PHP filter wrappers to base64-encode files  
- Log poisoning (inject PHP into logs then include them)  
- Session file inclusion

**Prevention**  
- Never use user input directly in file paths  
- Canonicalise and validate the final path  
- Use allowlists of permitted files  
- Disable dangerous PHP wrappers if possible

**Key points**  
- Even if `../` is blocked, encoding and wrappers often still work.  
- LFI is frequently chained with other issues to achieve RCE.  
- Always test for both reading and inclusion.

---

### XML External Entity (XXE)

**What is it?**  
The XML parser is configured to process external entity declarations supplied by the attacker.

**How it works**  
1. Application accepts XML input.  
2. Parser resolves external entities.  
3. Attacker defines an entity that points to a local file or internal URL.  
4. When the entity is referenced, the content is included in the response or leaked out-of-band.

**Common variants**  
- Classic XXE (file content returned in response)  
- Blind XXE (out-of-band via DNS/HTTP)  
- XXE to SSRF  
- Denial of service (billion laughs attack)

**Impact**  
Local file disclosure, internal network scanning, denial of service, remote code execution in rare cases.

**Testing**  
- External entity declarations  
- Parameter entities  
- Out-of-band techniques (Burp Collaborator, Interactsh)  
- Different XML parsers and content types

**Prevention**  
- Disable external entity processing and DTD processing  
- Prefer safer data formats (JSON)  
- Use hardened XML parsers with secure defaults

**Key points**  
- Many modern frameworks disable XXE by default, but older or custom parsers often do not.  
- Blind XXE is still very useful when direct response is not available.  
- Always test both classic and out-of-band methods.

---

### Insecure Deserialisation

**What is it?**  
Untrusted data is deserialised into objects, allowing an attacker to control object properties or execute code via gadget chains.

**How it works**  
1. Application deserialises data from the user (cookie, parameter, API body).  
2. Attacker crafts a malicious serialised object.  
3. During deserialisation, dangerous methods are called (gadget chains).  
4. This often leads to remote code execution.

**Common variants**  
- Java (ysoserial gadgets)  
- PHP (object injection)  
- .NET  
- Python pickle  
- Node.js

**Impact**  
Remote code execution, privilege escalation, denial of service, authentication bypass.

**Testing**  
- Identify serialised data (base64, Java serialised streams, etc.)  
- Use known gadget chains for the language  
- Look for type confusion or property overriding

**Prevention**  
- Avoid deserialising untrusted data  
- Use allowlisting of permitted classes  
- Integrity checks / signing of serialised data  
- Prefer safer formats (JSON with strict schemas)

**Key points**  
- This is one of the highest-impact vulnerabilities when present.  
- Detection often starts with recognising serialised data formats.  
- Language-specific tools (ysoserial, phpggc, etc.) are essential.

---

### JWT Attacks

**What is it?**  
Weaknesses in how JSON Web Tokens are created, signed, or validated.

**How it works**  
1. Application issues a JWT after login.  
2. Client sends the JWT on subsequent requests.  
3. Attacker modifies the token (algorithm, claims, signature).  
4. Server accepts the modified token due to weak validation.

**Common variants**  
- Algorithm “none”  
- Algorithm confusion (RS256 → HS256)  
- Weak secret / brute-forceable HMAC key  
- kid header injection  
- Claim manipulation (role, exp, etc.)

**Impact**  
Authentication bypass, privilege escalation, account takeover.

**Testing**  
- Change `"alg": "none"` and remove signature  
- Switch RS256 to HS256 and sign with the public key  
- Brute-force weak secrets  
- Manipulate claims (role, admin, expiry)

**Prevention**  
- Explicitly whitelist allowed algorithms  
- Use strong secrets or asymmetric keys properly  
- Validate all claims (exp, nbf, aud, etc.)  
- Never trust the algorithm header blindly

**Key points**  
- Algorithm confusion is still surprisingly common.  
- Always check both the signature and the claims.  
- Libraries that force algorithm selection are much safer.

---

### Business Logic Flaws

**What is it?**  
Abuse of intended application functionality that does not rely on a classic technical vulnerability (injection, XSS, etc.).

**How it works**  
The application correctly implements individual features but fails to enforce business rules across steps or states.

**Common variants**  
- Race conditions (limit overruns, double spending)  
- Price / quantity manipulation  
- Workflow bypass (skipping steps)  
- Negative values or boundary abuse  
- Coupon / discount stacking

**Impact**  
Financial loss, data integrity issues, unauthorised actions, privilege escalation.

**Testing**  
- Parallel requests (Burp Turbo Intruder)  
- Reordering or skipping steps in multi-step processes  
- Edge cases (zero, negative, very large values)  
- Replaying or reusing tokens / coupons

**Prevention**  
- Server-side validation of every step and final state  
- Proper locking / atomic operations  
- Rate limiting and concurrency controls  
- Clear business rule enforcement

**Key points**  
- These bugs often require understanding the application’s intended flow.  
- They are frequently missed by automated scanners.  
- High impact when found in financial or privilege-related features.

---

This level of depth matches the CSRF section.  
Would you like me to continue with any remaining topics (e.g. Command Injection, Mass Assignment, GraphQL) in the same style, or start turning these into a full revision document?

---

### CSRF Revision Cheat Sheet

**What is it?** 
Cross-Site Request Forgery = A malicious site tricks your browser into sending a request to a vulnerable site where you are logged in.  
The browser automatically includes your session cookie, so the target site thinks the request is legitimate.

---

**Why Cookies Matter**
- Session cookie is automatically sent with every request to the domain.
- Attacker does not need to steal the cookie — the browser does the work for them.

---

**Cookie Attributes (Set by Server in Set-Cookie Header)**

| Attribute      | Purpose                                      | Common Values                  | CSRF Impact |
|----------------|----------------------------------------------|--------------------------------|-------------|
| **SameSite**   | Controls cross-site sending                  | Lax (default), Strict, None    | Main defence. Lax/Strict blocks most CSRF |
| **HttpOnly**   | Prevents JavaScript access                   | Present or absent              | Protects against XSS stealing cookie |
| **Secure**     | Only send over HTTPS                         | Present or absent              | Required for SameSite=None |

**Best combination for session cookie**: `HttpOnly; Secure; SameSite=Lax` (or Strict)

---

**Important Request Headers (Added Automatically by Browser)**

| Header               | What it tells the server                     | Usefulness for CSRF |
|----------------------|----------------------------------------------|---------------------|
| **Origin**           | Only scheme + domain                         | Very reliable |
| **Referer**          | Full URL of the triggering page              | Useful but often missing |
| **Sec-Fetch-Site**   | same-origin / same-site / cross-site         | One of the strongest |
| **Sec-Fetch-Mode**   | How request was made (cors, navigate, etc.)  | Good supporting info |
| **X-CSRF-Token**     | Anti-CSRF token value                        | Strong when used |

---

**Main Defences (in order of strength)**

1. **SameSite=Lax/Strict** on session cookie
2. **CSRF Token** (double-submit cookie or header token)
3. **Origin / Sec-Fetch-Site** validation on server
4. Proper error handling and rate limiting

---

**How a CSRF Token Usually Works**
- Server sets a random `csrf_token` cookie
- Legitimate page includes the same value in form field or header
- Server checks they match
- Attacker cannot read the token (SameSite + same-origin policy)

---

**Edge Cases & Limitations**
- SameSite=None + Secure is needed for some third-party flows (weaker)
- Older browsers may not support SameSite or Sec-Fetch headers
- Referer can be blocked by privacy settings
- Some legitimate cross-site flows (OAuth, payments) need careful handling

---

**Tip**  
Strongest modern setup:  
`SameSite=Lax` on session cookie + CSRF token + `Origin`/`Sec-Fetch-Site` check.

---

### Quick Overviews – “Nice to Have” Skills

**Vulnerability Management**
This is the process of identifying, prioritising, tracking, and remediating vulnerabilities across an organisation’s systems. It usually involves scanning, risk scoring (e.g. CVSS), assigning ownership, tracking remediation progress, and reporting. In a bug bounty triage role it is useful because you already understand how vulnerabilities are prioritised and managed after they are validated.

**Software QA (Quality Assurance)**
Software QA focuses on finding defects in software through testing (manual and automated). It involves writing test cases, checking edge cases, regression testing, and ensuring software behaves as expected. This experience is valuable in triage because good QA skills help you systematically reproduce issues and spot incomplete or low-quality reports.

**SAST and DAST**
- **SAST** (Static Application Security Testing) analyses source code or bytecode without running the application. It is good at finding coding issues early but can produce false positives.
- **DAST** (Dynamic Application Security Testing) tests the running application from the outside (like a black-box scanner). It finds issues that only appear at runtime but has less context about the code.

Both are useful background knowledge because many clients use these tools, and understanding their strengths and limitations helps when triaging reports that may overlap with scanner findings.

--- 

## Questions

**OWASP Top 10 (2025) + Core Vulnerabilities**  

### A01 – Broken Access Control

1. What is the difference between horizontal and vertical privilege escalation?  
Horizontal privilege escalation is accessing another user’s data at the same privilege level. Vertical privilege escalation is accessing higher-privilege functionality or data (e.g. admin features).

2. How would you test for IDOR / BOLA in an API?  
Create two accounts, authenticate as one, and try to access the other user’s objects by changing IDs in API requests. Check whether the server enforces ownership or role checks.

3. Why is SSRF now included under Broken Access Control in the 2025 list?  
SSRF allows an attacker to make the server access internal resources it should not be able to reach, which is fundamentally an access control failure. In 2025 it was therefore moved under Broken Access Control.

### A02 – Security Misconfiguration

1. Give three common examples of security misconfigurations.  
Default credentials left enabled, unnecessary services or ports open, overly permissive CORS, verbose error messages, and missing security headers.

2. How can default credentials or unnecessary services lead to compromise?  
Default credentials give attackers an easy entry point. Unnecessary services increase the attack surface and may contain unpatched vulnerabilities.

3. What is the risk of verbose error messages?  
Verbose error messages can leak stack traces, database details, file paths, or internal IP addresses that help an attacker.

### A03 – Software Supply Chain Failures

1. What does “Software Supply Chain Failures” cover beyond just outdated libraries?  
It covers vulnerable or malicious third-party libraries, compromised build systems, CI/CD pipelines, and distribution mechanisms — not just outdated packages.

2. How can a compromised CI/CD pipeline be more dangerous than a vulnerable dependency?  
A compromised CI/CD pipeline can inject malicious code into every build and affect all downstream users, whereas a single vulnerable dependency usually has more limited impact.

3. What controls help reduce supply chain risk?  
Dependency scanning (SCA), signed packages, locked dependency versions, secure CI/CD practices, and allowlisting of approved components.

### A04 – Cryptographic Failures

1. What is the difference between encryption in transit and encryption at rest?  
Encryption in transit protects data while it is moving (e.g. TLS). Encryption at rest protects data while it is stored (e.g. database or disk encryption).

2. Why is storing passwords with a weak hashing algorithm a cryptographic failure?  
Weak hashing (e.g. MD5 or unsalted SHA-1) allows attackers to crack passwords efficiently using rainbow tables or brute force.

3. Give an example of a common cryptographic mistake in web applications.  
Using outdated algorithms (MD5, SHA-1), hard-coded keys, or transmitting sensitive data over HTTP instead of HTTPS.

### A05 – Injection

1. Why are parameterised queries the strongest defence against SQL injection?  
Parameterised queries separate the SQL code from the data. The database treats user input strictly as data and never as executable code.

2. What is the difference between error-based, boolean-based, and time-based SQL injection?  
Error-based returns database errors that leak information. Boolean-based observes true/false differences in responses. Time-based measures delays caused by SLEEP() or similar functions.

3. How can command injection occur even if the application is not directly executing shell commands?  
If user input is passed into a function that eventually executes a system command (e.g. via a library or OS call), command injection can still occur.

### A06 – Insecure Design

1. What is the main difference between Insecure Design and Implementation flaws?  
Insecure Design means the application was designed without proper security controls. Implementation flaws mean the design was correct but the code was written incorrectly.

2. Give an example of a business logic flaw that would fall under Insecure Design.  
Allowing users to apply the same discount code unlimited times, or letting a user complete a multi-step process while skipping important validation steps.

3. How does threat modelling help prevent insecure design issues?  
Threat modelling identifies potential threats and required controls early in the design phase so they can be built in rather than bolted on later.

### A07 – Authentication Failures

1. What is credential stuffing and how is it different from brute force?  
Credential stuffing uses previously breached username/password pairs. Brute force systematically guesses passwords, often against a single account.

2. Why is a weak password reset flow a serious authentication failure?  
A weak password reset flow can allow an attacker to take over accounts by guessing or manipulating reset tokens, or by resetting another user’s password.

3. What risks do poorly implemented JWT authentication introduce?  
Weaknesses include accepting the “none” algorithm, algorithm confusion attacks, weak secrets, and failing to validate important claims (exp, iss, etc.).

### A08 – Software or Data Integrity Failures

1. What is insecure deserialisation and why is it dangerous?  
Insecure deserialisation occurs when untrusted data is converted back into objects. Attackers can craft malicious objects that execute code during deserialisation.

2. How can an attacker abuse an unsigned software update mechanism?  
If updates are not signed or the signature is not properly verified, an attacker can distribute a malicious update that users will trust and install.

3. Why is integrity protection important in CI/CD pipelines?  
Without integrity controls, an attacker who compromises the pipeline can inject malicious code that is then trusted and deployed.

### A09 – Security Logging & Alerting Failures

1. Why is lack of logging considered a security risk?  
Without proper logs it is very difficult to detect, investigate, or respond to attacks. Attackers can operate for long periods without being noticed.

2. What should be logged for security-relevant events?  
Successful and failed logins, access to sensitive data, privilege changes, input validation failures, and administrative actions.

3. How can poor alerting allow attacks to go unnoticed for long periods?  
If alerts are missing or ignored, active attacks (e.g. brute force or data exfiltration) can continue for days or weeks without response.

### A10 – Mishandling of Exceptional Conditions

1. What kinds of issues fall under “Mishandling of Exceptional Conditions”?  
Improper error handling, unhandled edge cases, resource exhaustion, and failures that reveal sensitive information or leave the application in an insecure state.

2. How can improper error handling lead to information disclosure?  
Detailed error messages can disclose stack traces, database structure, file paths, or internal logic that helps an attacker.

3. Give an example of how failing to handle edge cases can create a vulnerability.  
Failing to handle very large inputs, negative values, or unexpected characters can lead to crashes, logic bypasses, or injection vulnerabilities.

---

### Core Vulnerabilities

**IDOR / Broken Object Level Authorisation**

1. How would you test for IDOR in a multi-user application?  
Create two accounts. Authenticate as User A and attempt to access User B’s resources by changing object IDs in requests. Observe whether access is granted.

2. Why are sequential IDs particularly risky?  
Sequential IDs are easy to enumerate, so an attacker can simply increment or decrement the ID to access other users’ data.

3. Can using UUIDs alone prevent IDOR? Why or why not?  
No. UUIDs only make enumeration harder. Without a proper authorisation check, an attacker who obtains a valid UUID can still access the object.

**SQL Injection**

1. Explain how a parameterised query prevents SQL injection.  
The query structure is sent separately from the data. User input is bound as parameters and never interpreted as SQL code.

2. What is second-order SQL injection?  
Second-order SQL injection occurs when malicious input is stored in the database and later used in a different query without proper handling.

3. When would you use time-based blind SQL injection?  
When the application does not return useful errors or data differences, but you can still detect a delay caused by a time-based payload.

**XSS (Cross-Site Scripting)**

1. What is the difference between Reflected, Stored, and DOM-based XSS?  
Reflected XSS is returned immediately in the response. Stored XSS is saved and later shown to other users. DOM-based XSS occurs entirely in client-side JavaScript.

2. How does Content Security Policy (CSP) help prevent XSS?  
CSP restricts which scripts are allowed to run. By blocking inline scripts and untrusted sources, most XSS payloads are prevented from executing.

3. Why does setting the HttpOnly flag on cookies not fully stop XSS impact?  
HttpOnly stops JavaScript from reading the cookie, but the attacker can still perform actions as the user (e.g. make requests, change data, or display phishing content).

**CSRF**

1. How does the SameSite cookie attribute help prevent CSRF?  
SameSite=Lax or Strict prevents the browser from sending the cookie on cross-site requests, so the attacker’s forged request has no session.

2. What is the difference between the Origin and Referer headers?  
Origin only contains the scheme and domain. Referer contains the full URL of the previous page. Origin is more reliable and privacy-friendly.

3. Why can’t an attacker simply remove security headers in a CSRF attack?  
The browser automatically adds Origin and Sec-Fetch headers. An attacker cannot remove or spoof them from a normal cross-site request.

**SSRF**

1. What makes cloud metadata endpoints such a high-value target in SSRF attacks?  
Cloud metadata endpoints often contain temporary credentials or sensitive configuration that can give the attacker access to the cloud environment.

2. What is the difference between basic and blind SSRF?  
Basic SSRF returns the response to the attacker. Blind SSRF does not return the response directly; the attacker must use side channels (DNS, timing, etc.).

3. How would you prevent SSRF in an application that needs to fetch external URLs?  
Use a strict allowlist of permitted domains or IPs, block internal ranges, and avoid letting user input control the full URL.

**File Upload**

1. Why is checking only the file extension insufficient protection?  
Attackers can bypass extension checks with double extensions, null bytes, or by changing the Content-Type header. The actual content must also be validated.

2. What is a polyglot file and why is it useful to attackers?  
A polyglot file is valid as more than one file type (e.g. both an image and a script). It can bypass filters that only check one aspect of the file.

3. What is the safest way to store user-uploaded files?  
Store uploaded files outside the web root, give them random names, serve them from a separate domain, and never execute user-uploaded content.

**JWT Attacks**

1. What is the “none” algorithm attack?  
The attacker changes the algorithm to “none” and removes the signature. If the server accepts it, the token is trusted without verification.

2. Explain the algorithm confusion attack (RS256 to HS256).  
The attacker changes the algorithm from RS256 (asymmetric) to HS256 (symmetric) and signs the token using the public key as the HMAC secret. If the server does not restrict algorithms, it accepts the token.

3. Why should the algorithm not be taken from the JWT header?  
The algorithm header is controlled by the user. The server must ignore it and enforce a fixed, expected algorithm instead.