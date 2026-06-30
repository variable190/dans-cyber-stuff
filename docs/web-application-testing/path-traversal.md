# Path Traversal

## Overview

Path traversal (also known as directory traversal) vulnerabilities allow an attacker to access files and directories that are stored outside the web root folder. By manipulating variables that reference files with "dot-dot-slash" (`../`) sequences and similar variations, an attacker can traverse the file system and access sensitive files such as configuration files, source code, or operating system files.

This can lead to information disclosure and, in some cases, be chained with other flaws for remote code execution.

## Attack Surface

- File download or view features that accept filenames or paths as parameters (e.g., `?file=report.pdf`, `?image=logo.png`)
- Language or theme selectors that include files dynamically
- Logging or report generation features
- Any functionality that constructs file paths from user input without proper sanitization

## Identification

- Look for parameters that appear to reference files, paths, or resources.
- Test basic traversal payloads against suspected parameters.
- Use fuzzing with dedicated wordlists to discover vulnerable parameters and successful traversals.
- Observe different responses or status codes when requesting non-existent vs. existing files outside the intended directory.
- Check for additional requests in proxy history that might load resources via file paths.

## Exploitation

### Basic Testing Techniques

- Try accessing sensitive files without traversal first: `/etc/passwd` or `C:\Windows\win.ini`
- Test common traversal sequences:
  - `../../../etc/passwd` (Linux)
  - `..\..\..\windows\win.ini` (Windows equivalent)
- Include the required base folder when the application expects it: `filename=/var/www/images/../../../etc/passwd`
- Use null byte termination (in older systems) to bypass appended extensions: `../../../etc/passwd%00.png`

### Bypasses and Variations

- Use multiple traversal sequences or encoding to bypass filters:
  - `....//....//....//....//etc/passwd`
  - URL-encoded versions: `%2e%2e%2f%2e%2e%2f%2e%2e%2f%2e%2e%2f%65%74%63%2f%70%61%73%73%77%64`
- Path truncation techniques (less common in modern systems): append many `./` after a non-existing directory to exceed path limits.

### Useful Commands for Confirmation

After successful traversal:

- On Linux: target `/etc/passwd`, `/etc/shadow`, application config files, source code.
- On Windows: `C:\Windows\win.ini`, `C:\boot.ini`, web.config, or application binaries.

## Impact

- Disclosure of sensitive system files, application source code, database credentials, or user data.
- Potential for further exploitation (e.g., reading private keys, configuration that reveals other attack surfaces).
- In combination with file write or other features, could lead to code execution.

## Prevention

- Avoid using user-supplied input to construct file paths when possible.
- Use a whitelist of allowed files or directories.
- Canonicalize (normalize) the path and validate that the resulting path is within the intended base directory.
- Implement proper access controls and run the application with the least privileges required.
- Use modern frameworks that handle file serving securely.

## Tools & Payloads

- Burp Suite Intruder with path traversal wordlists (e.g., from SecLists Fuzzing/LFI or custom lists)
- ffuf or similar for parameter and payload fuzzing
- Common test files:
  - Linux: `/etc/passwd`, `/etc/shadow`, application config files
  - Windows: `win.ini`, `boot.ini`

**Example fuzzing command (adapt as needed):**
```bash
ffuf -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://target/index.php?file=../../../../FUZZ' -fs 2287
```

See also the dedicated [File Inclusion](file-inclusion.md) page for more advanced LFI/RFI techniques and wrappers that often build on path traversal.
