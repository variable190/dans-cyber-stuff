# FTP

## Overview
FTP (File Transfer Protocol) is used to transfer files between a client and server. In bug bounty hunting, it can be leveraged for reconnaissance (e.g., discovering open FTP servers), exploitation (e.g., uploading malicious files), or data exfiltration.

## Basic Commands
| Command | Description | Example |
|---------|-------------|---------|
| `ftp <hostname>` | Connect to an FTP server | `ftp ftp.example.com` |
| `open <hostname> <port>` | Open a connection to a specific host and port | `open ftp.example.com 21` |
| `user <username> <password>` | Log in with username and password | `user admin password123` |
| `anonymous` | Attempt anonymous login (no credentials) | `anonymous` |
| `ls` or `dir` | List files in the current directory | `ls` |
| `cd <directory>` | Change to a specified directory | `cd /public` |
| `get <filename>` | Download a file from the server | `get sensitive.txt` |
| `put <filename>` | Upload a file to the server | `put shell.php` |
| `binary` | Switch to binary mode for non-text files | `binary` |
| `ascii` | Switch to ASCII mode for text files | `ascii` |
| `bye` or `quit` | Disconnect from the FTP server | `bye` |

**Tip:** `ftp ftp://username:password\!@localhost` local connection with known credentials, after connection through ssh for example.

## Commonly Used Flags
| Flag | Purpose |
|------|----------|
| `-v` | Verbose output, showing detailed connection and transfer info. |
| `-i` | Turns off interactive prompting during multiple file transfers. |
| `-n` | Disables auto-login upon initial connection. |
| `-p` | Enables passive mode to traverse firewalls (e.g., `ftp -p ftp.example.com`). |

## Exploitation Tips
- **Anonymous Access**: Test `anonymous` login to upload files or exfiltrate data if permitted (username: anonymous and password: user@domain.com when prompted).
- **Brute Forcing**: Use tools like Hydra (`hydra -l user -P pass.txt ftp://ftp.example.com`) to guess credentials.
- **File Upload**: Upload malicious scripts (e.g., PHP shells) if write access is granted.

