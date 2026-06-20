# Web Fundamentals

## HTTP Response Codes

### Classes

| Class | Description |
|-------|-------------|
| 1xx | Provides information, does not affect request processing |
| 2xx | Indicates a successful request |
| 3xx | Indicates server redirection of the client |
| 4xx | Signifies improper client requests (e.g., nonexistent resource or bad format) |
| 5xx | Indicates an issue with the HTTP server |

### Specific Codes

| Code | Name | Description |
|------|------|-------------|
| 100 | Continue | Request can continue |
| 200 | OK | Request successful |
| 201 | Created | Resource created |
| 204 | No Content | Request successful, no content |
| 301 | Moved Permanently | Resource moved permanently |
| 302 | Found | Resource temporarily moved |
| 400 | Bad Request | Invalid request |
| 401 | Unauthorized | Authentication required |
| 403 | Forbidden | Access denied |
| 404 | Not Found | Resource not found |
| 405 | Method Not Allowed | Known by the server but has been disabled and cannot be used |
| 408 | Request Timeout | Sent on an idle connection by some servers, even without any previous request by the client |
| 500 | Internal Server Error | Server error |
| 502 | Bad Gateway | Invalid gateway response |
| 503 | Service Unavailable | Server temporarily unavailable |
| 504 | Gateway Timeout | The server is acting as a gateway and cannot get a response in time |

## HTTP Headers

[Complete list of standard headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers)

### Request Headers

Contain more information about the resource to be fetched, or about the client requesting the resource.

| Header | Example | Description |
|--------|---------|-------------|
| Accept | `text/html,application/xhtml+xml` | Media types acceptable for the response |
| Authorization | `Bearer eyJhbGciOiJIUzI1NiJ9...` | Credentials for authentication |
| User-Agent | `Mozilla/5.0 (Windows NT 10.0; Win64; x64)` | Client software details |
| Host | `developer.mozilla.org` | Domain name of the server |
| Referer | `https://example.com/page` | URL of the page that initiated the request |
| Cookie | `sessionId=abc123; user=john` | Cookies previously set by the server, sent with the request |

[Full list of request headers](https://datatracker.ietf.org/doc/html/rfc7231#section-5)

### Response Headers

Hold additional information about the response, like its location or about the server providing it.

| Header | Example | Description |
|--------|---------|-------------|
| Location | `https://developer.mozilla.org/` | URL for redirection |
| Server | `Apache/2.4.41 (Unix)` | Server software information |
| Allow | `GET, POST, HEAD` | Allowed HTTP methods |
| Date | `Tue, 09 Sep 2025 09:09:00 GMT` | Date and time of the response |
| Set-Cookie | `sessionId=abc123; Path=/; HttpOnly` | Defines a cookie to be stored by the client |
| WWW-Authenticate |     | Authentication method and realm for access |

[Full list of response headers](https://datatracker.ietf.org/doc/html/rfc7231#section-7)

### Security Headers

Response headers that enforce browser security policies to protect websites from attacks.

| Header | Example | Description |
|--------|---------|-------------|
| Content-Security-Policy | `script-src 'self'` | Sets rules for allowed resource sources, preventing XSS by restricting scripts to trusted domains. |
| Strict-Transport-Security | `max-age=31536000` | Forces HTTPS connections, blocking plaintext HTTP to prevent traffic sniffing. |
| Referrer-Policy | `origin` | Controls `Referer` header, limiting sensitive URL exposure during navigation. |

[OWASP secure response headers](https://owasp.org/www-project-secure-headers/)

### Representation/Entity Headers

Contain information about the body of the resource, like its MIME type, or encoding/compression applied. Common to both request and response.

| Header | Example | Description |
|--------|---------|-------------|
| Content-Type | `text/html; charset=UTF-8` | MIME type and character encoding of the body |
| Content-Encoding | `gzip` | Compression method applied to the body |
| Content-Language | `en-US` | Language of the resource content |
| Content-Location | `/documents/foo.json` | Alternate location for the returned data |
| Boundary | `----WebKitFormBoundary7MA4YWxkTrZu0gW` | Delimiter for separating parts in multipart messages, e.g., form data or file uploads |
| Media-Type | `multipart/form-data` | Specifies the MIME type of the message body, often used with multipart data |

### Payload Headers

Contain representation-independent information about payload data, including content length and the encoding used for transport.

| Header | Example | Description |
|--------|---------|-------------|
| Content-Length | `348` | Size of the response body in bytes |
| Transfer-Encoding | `chunked` | Encoding used for data transfer |
| Trailer | `Expires` | Headers sent after the chunked response |

## HTTP Request Methods

| Method | Description |
|--------|-------------|
| GET | Requests a resource |
| POST | Submits data to create/update a resource |
| PUT | Updates a resource with provided data |
| DELETE | Removes a specified resource |
| HEAD | Requests headers only, no body |
| OPTIONS | Lists allowed methods for a resource |
| PATCH | Partially updates a resource |
| TRACE | Echoes the received request for debugging |

## Browser DevTools

| Shortcut | Description |
|----------|-------------|
| `[CTRL+SHIFT+I]` or `[F12]` | Show devtools |
| `[CTRL+SHIFT+E]` | Show Network tab |
| `[CTRL+SHIFT+K]` | Show Console tab |

**Tip:** Use the network tab to observe dynamic content in action

## Web Apps

### Web Stack Combinations

| Combinations | Components                          |
|--------------|-------------------------------------|
| LAMP         | Linux, Apache, MySQL, and PHP       |
| WAMP         | Windows, Apache, MySQL, and PHP     |
| WINS         | Windows, IIS, .NET, and SQL Server  |
| MAMP         | macOS, Apache, MySQL, and PHP       |
| XAMPP        | Cross-Platform, Apache, MySQL, and PHP/PERL |

### OWASP Top Ten Vulnerabilities

| No. | Vulnerability                           |
|-----|-----------------------------------------|
| 1   | Broken Access Control                   |
| 2   | Cryptographic Failures                   |
| 3   | Injection                                |
| 4   | Insecure Design                          |
| 5   | Security Misconfiguration                |
| 6   | Vulnerable and Outdated Components       |
| 7   | Identification and Authentication Failures |
| 8   | Software and Data Integrity Failures     |
| 9   | Security Logging and Monitoring Failures |
| 10  | Server-Side Request Forgery (SSRF)       |

- [OWASP top 10 vulnerabilities](https://owasp.org/www-project-top-ten/)
- [OWASP web application security testing guide](https://github.com/OWASP/wstg/tree/master/document/4-Web_Application_Security_Testing)
- [OWASP cheat sheet series](https://cheatsheetseries.owasp.org/index.html)

### URL Encoding Reference

| Character | URL Encoded |
|-----------|-------------|
| space     | %20         |
| !         | %21         |
| "         | %22         |
| #         | %23         |
| $         | %24         |
| %         | %25         |
| &         | %26         |
| '         | %27         |
| (         | %28         |
| )         | %29         |

### Practicing Web Application Development Steps

| Step | To-Do                                  |
|------|----------------------------------------|
| 1    | Set up a VM with a web server          |
| 2    | Create an HTML page                    |
| 3    | Design it with CSS                     |
| 4    | Add some simple functions with JavaScript |
| 5    | Program a simple web application       |
| 6    | Connect your web application to the database |
| 7    | Experiment with APIs                   |
| 8    | Test your application for various vulnerabilities and security holes |
| 9    | Try to adjust your code and configurations to close the vulnerabilities |

### Tips

- Review source code for sensitive data exposure

### Helpful Links

- [W3Schools](https://www.w3schools.com/)
- [Front end practice sandbox 1](https://html-css-js.com/)
- [Front end practice sandbox 2](https://html6.com/editor/)
- [Front end learn test and share](https://codepen.io/)
- [Exploit DB](https://www.exploit-db.com/)
- [Rapid7 DB](https://www.rapid7.com/db/)
- [National Vulnerability Database](https://nvd.nist.gov/)
- [Common Vulnerability Scoring System: User Guide](https://www.first.org/cvss/user-guide)

## Using Web Proxies

### Tools

- [Burp Suite](https://portswigger.net/burp)
- [OWASP ZAP](https://www.zaproxy.org/)
- [Foxy Proxy](https://addons.mozilla.org/en-US/firefox/addon/foxyproxy-standard/)
- [Proxychains](https://github.com/haad/proxychains)

### Setup

- Configure proxy settings/install and configure foxy proxy
- Add certificate

### Tool Functions 

| Function                  | Burp                                      | ZAP                                      |
|---------------------------|-------------------------------------------|------------------------------------------|
| Intercepting Requests     | Proxy tab, Intercept sub-tab, toggle on   | Toggle green button or [Ctrl+B]          |
| Intercept Response        | Proxy > Options, enable Intercept Response | Automatically enabled with intercept     |
| Automatic Modification    | Proxy > Options > Match and Replace, add rule | [Ctrl+R] or Replacer in Options          |
| Repeating Requests        | [Ctrl+R] or right-click, send to Repeater | Right-click, select Open/Resend          |
| URL Encoding              | Right-click, Convert Selection > URL Encode, or [Ctrl+U] | Auto-encodes request data before sending |
| Decoding                  | Decoder tab                              | [Ctrl+E]                                 |
| Fuzzing                   | [https://academy.hackthebox.com/module/110/section/1054](https://academy.hackthebox.com/module/110/section/1054) | [https://academy.hackthebox.com/module/110/section/1056](https://academy.hackthebox.com/module/110/section/1056) |
| Web Scanner               | [https://academy.hackthebox.com/module/110/section/1084](https://academy.hackthebox.com/module/110/section/1084) | [https://academy.hackthebox.com/module/110/section/1086](https://academy.hackthebox.com/module/110/section/1086) |
| Extensions                | [https://portswigger.net/bappstore](https://portswigger.net/bappstore) | [https://www.zaproxy.org/addons/](https://www.zaproxy.org/addons/) |
| DOM Invader               | Available in integrated browser https://portswigger.net/burp/documentation/desktop/tools/dom-invader | Burp Only |

### Burp Shortcuts

| Shortcut      | Description         |
|---------------|---------------------|
| [CTRL+R]      | Send to repeater    |
| [CTRL+SHIFT+R]| Go to repeater      |
| [CTRL+I]      | Send to intruder    |
| [CTRL+SHIFT+I]| Go to intruder      |
| [CTRL+U]      | URL encode          |
| [CTRL+SHIFT+U]| URL decode          |

### ZAP Shortcuts

| Shortcut      | Description             |
|---------------|-------------------------|
| [CTRL+B]      | Toggle intercept on/off |
| [CTRL+R]      | Go to replacer          |
| [CTRL+E]      | Go to encode/decode/hash|

### Firefox Shortcuts

| Shortcut      | Description           |
|---------------|-----------------------|
| [CTRL+SHIFT+R]| Force Refresh Page    |
| [F12]         | Open Developer Tools  |
| [CTRL+SHIFT+I]| Open Inspector        |
| [CTRL+SHIFT+E]| Open Network Panel    |
| [CTRL+SHIFT+J]| Open Console          |
| [CTRL+U]      | View Page Source      |