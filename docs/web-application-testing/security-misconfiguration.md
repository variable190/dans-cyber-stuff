# Security Misconfiguration

## Overview

Security misconfiguration occurs when security settings are not properly defined or are left at insecure defaults. This broad category includes exposed debug interfaces, unnecessary services, default credentials, verbose error messages, and unsafe feature configurations (e.g., allowing dangerous PHP wrappers, template engines with code execution, or SSI directives).

Misconfigurations often enable or amplify other vulnerabilities.

## Attack Surface

- Application and server configuration files
- Debug endpoints, admin interfaces, or backup files left accessible
- Default or overly permissive settings in frameworks, web servers, databases, and cloud environments
- Features like Server-Side Template Injection (SSTI), SSI, or XSLT that are enabled when not needed
- API versions and inventory that are not properly managed

## Identification

- Look for verbose error messages, stack traces, or debug information.
- Test for common misconfigurations (default credentials, exposed .git, .env, backup files).
- Check security headers (or lack thereof).
- Test whether dangerous features (template engines, SSI, PHP wrappers) are enabled.
- Enumerate API versions and functionality differences.
- Look for insecure headers that may enable other attacks (e.g., missing anti-CSRF or CORS misconfig).

## Exploitation

### Common Misconfigurations

- Unnecessary services or modules enabled.
- Default accounts or weak configurations.
- Debug modes left on in production.
- Files containing sensitive information left readable.

### Insecure Headers / Feature Abuse

- Test for SQLi, command injection, or CSRF where headers or features are misconfigured.
- Check file uploads for unconstrained sizes or dangerous file types.

### Server-Side Template Injection (SSTI) as Misconfiguration

Template engines (Jinja, Twig, etc.) can be abused when user input is passed directly into templates.

Test string: `${{<%[%'"}}%\` .`

Jinja (Python) examples:
```jinja
{{ config.items() }}
{{ self.__init__.__globals__.__builtins__.open("/etc/passwd").read() }}
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

Twig (PHP):
```twig
{{ "/etc/passwd"|file_excerpt(1,-1) }}
{{ ['id'] | filter('system') }}
```

Tools: SSTImap

### Server-Side Includes (SSI)

Typical extensions: .shtml, .shtm, .stm

Useful directives:
- `<!--#exec cmd="whoami" -->`
- `<!--#include virtual="index.html" -->`

### XSLT Injection

Can be used for information disclosure, LFI, or RCE depending on the processor:
```xslt
<xsl:value-of select="system-property('xsl:version')" />
<xsl:value-of select="unparsed-text('/etc/passwd', 'utf-8')" />
<xsl:value-of select="php:function('system','id')" />
```

## Impact

- Information disclosure (source code, credentials, internal paths)
- Remote code execution via enabled dangerous features
- Amplification of other vulnerabilities
- Unauthorised access to administrative functionality
- Compliance and reputational damage

## Prevention

- Follow the principle of least functionality: disable unused features, modules, and services.
- Use secure defaults and harden configurations (disable dangerous PHP settings like `allow_url_include`, restrict template engines).
- Implement proper error handling that does not leak sensitive information.
- Regularly review and inventory exposed endpoints and API versions.
- Use infrastructure-as-code and automated configuration scanning.
- Apply security headers consistently (CSP, X-Frame-Options, etc.).
- Keep all components (OS, frameworks, libraries) patched.

## Tools & Payloads

- Configuration scanners and vulnerability scanners (Nikto, etc.)
- SSTImap for template injection testing
- Manual testing with Burp for headers, error messages, and feature abuse
- ffuf or gobuster for discovering exposed files and directories

**Example SSTI test command:**
```bash
python3 sstimap.py -u http://target/index.php?name=test -S id
```

See [File Inclusion](file-inclusion.md) (for PHP wrappers and config checks) and [SSRF](ssrf.md) for related misconfiguration vectors.

### Attacking Common Applications and Platforms

Many applications have well-known misconfigurations or default setups that can be exploited:

**Enumeration and Discovery**
- Add relevant vhosts: `printf "%s\t%s\n" "$IP" "target.htb" | sudo tee -a /etc/hosts`
- Port and service discovery: `nmap -p 80,443,8000,8080,... --open ...`
- Visual reconnaissance: eyewitness or aquatone on discovered hosts.

**WordPress**
- `wpscan --url <target> --enumerate ap,u`
- XML-RPC password attacks and plugin/theme enumeration.
- Common shells and brute force tools.

**Joomla / Drupal / Tomcat / Jenkins / Splunk / Others**
- Specific scanners (droopescan, mgr_brute, etc.).
- Default credential lists and Metasploit modules.
- Script console or manager interfaces for code execution (Groovy payloads, WAR uploads).

**General Web Shells**
- Simple PHP reverse shells and command execution via GET parameters.

Cheat sheets and specific tool commands for these platforms are included in the relevant sections.

### GraphQL and Web Service/API Attacks

**GraphQL Basics**
- Introspection queries reveal schema: query `__schema { types { name } }`
- Common issues: broken auth, excessive data exposure, injection via arguments.

**Testing Approach**
- Use introspection to map the API.
- Look for authorisation bypasses on objects/fields.
- Test for injection in resolvers.

**Other API/Web Service Issues**
- Broken authentication, object level authorisation, resource consumption limits.
- Improper inventory (old API versions exposed).
- Unsafe consumption of downstream APIs.

These often stem from or lead to security misconfigurations.

### Web Fuzzing

Fuzzing is a key technique for discovering hidden resources, parameters, and vulnerabilities:
- Directories/files, parameters, injection points (SQLi, XSS, command injection).
- Tools: ffuf, gobuster, wfuzz (after installing prerequisites like Go/Python).
- Wordlists from SecLists for paths, parameters, LFI, etc.

Incorporate fuzzing into testing of all other categories above.
