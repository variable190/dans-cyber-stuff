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
