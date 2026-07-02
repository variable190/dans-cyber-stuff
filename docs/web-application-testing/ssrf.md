# SSRF

## What is SSRF?

Server-Side Request Forgery (SSRF) vulnerabilities allow an attacker to make the server-side application send HTTP (or other protocol) requests to an arbitrary destination of the attacker's choosing.

## Exploits

- [Basic SSRF to Internal Services](ssrf/basic-internal.md)
- [SSRF for Port Scanning and Metadata](ssrf/port-scanning.md)
- [Blind SSRF and Protocol Abuse](ssrf/blind-and-protocol.md)

## Impact

Access to internal resources, scan internal networks, interact with cloud metadata services, bypass access controls.

## Prevention

- Whitelist allowed domains and protocols.
- Disable unnecessary URL schemes.
- Use network segmentation and firewall rules.
- Validate and sanitize all user-supplied URLs.

## Tools & Payloads

- ffuf for port and endpoint fuzzing
- Gopherus for crafting advanced protocol payloads
- netcat or Python simple HTTP server for basic callbacks
- Burp Collaborator / Interactsh for OAST confirmation

**Example port scan command (adapt parameters):**
```bash
seq 1 10000 > ports.txt
ffuf -w ./ports.txt -u http://target/index.php -X POST -d "target=http://127.0.0.1:FUZZ" -fr "connection refused"
```

Additional internal service examples (such as Redis, SMTP, and other protocols) are covered in the original study notes.
