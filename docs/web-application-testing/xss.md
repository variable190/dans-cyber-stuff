# XSS

## What is XSS?

Cross-Site Scripting (XSS) vulnerabilities allow attackers to inject malicious scripts into web pages viewed by other users. The victim's browser executes the attacker-controlled code in the context of the vulnerable site, enabling cookie theft, session hijacking, phishing, defacement, or actions on behalf of the victim.

## Exploits

- [Reflected XSS](xss/reflected.md)
- [Stored XSS](xss/stored.md)
- [DOM-based XSS](xss/dom-based.md)
- [Phishing / Credential Theft with XSS](xss/phishing.md)
- [Blind XSS](xss/blind.md)

## Impact

- Session hijacking via cookie theft
- Phishing and credential theft
- Actions performed with the victim's privileges (e.g., changing settings, making purchases)
- Keylogging or screen scraping
- Defacement or malware delivery
- Bypassing CSRF protections in some cases when combined with other attacks

## Prevention

- Encode/escape all user-controlled output according to context (HTML, JavaScript, CSS, URLs, attributes).
- Use modern frameworks that provide automatic escaping (React, Angular, etc.).
- Implement a strong Content-Security-Policy (CSP) as defense in depth.
- Sanitize rich HTML input with well-maintained libraries (e.g., DOMPurify).
- Avoid inserting user input into dangerous contexts (innerHTML without sanitization, `eval()`, etc.).
- Validate and encode on both client and server.

## Tools & Payloads

- **Manual testing:** Burp Repeater, browser dev tools
- **Automated:** XSStrike, XSSer, Burp Pro scanner, ZAP
- **Payload collections:**
  - PayloadsAllTheThings XSS section
  - PayloadBox XSS lists
- **Example basic test:**
  ```js
  <script>alert(window.origin)</script>
  ```

**Note:** XSS can appear in HTTP headers (User-Agent, Cookie) when those values are reflected on the page. Test comprehensively.

See the CSRF and Session Security notes for chaining techniques (e.g., XSS + CSRF).
