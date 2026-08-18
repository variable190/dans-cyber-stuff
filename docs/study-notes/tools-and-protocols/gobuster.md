# Gobuster

## Overview

Gobuster is a fast and flexible tool for brute-forcing directories, files, DNS names, and virtual hosts, widely used in bug bounty reconnaissance.

## Basic Commands

| Command | Description | Example |
|---------|-------------|---------|
| `gobuster dir -u <url> -w <wordlist>` | Brute-force directories and files on a web server. | `gobuster dir -u http://example.com -w /usr/share/wordlists/dirb/common.txt` |
| `gobuster dns -d <domain> -w <wordlist>` | Enumerate subdomains via DNS brute-forcing. | `gobuster dns -d example.com -w subdomains.txt` |
| `gobuster vhost -u <url> -w <wordlist>` | Discover virtual hosts on a target IP. | `gobuster vhost -u http://10.10.10.10 -w vhosts.txt` |

## Commonly Used Flags

| Flag | Purpose |
|------|---------|
| `-u` | Specifies the target URL or domain (e.g., `-u http://example.com`). |
| `-w` | Defines the wordlist file for brute-forcing (e.g., `-w /usr/share/wordlists/dirb/common.txt`). |
| `-t` | Sets the number of concurrent threads (e.g., `-t 50` for faster scanning). |
| `-x` | Specifies file extensions to test (e.g., `-x php,txt` for `.php` and `.txt`). |
| `-k` | Ignores SSL/TLS errors (e.g., `-k` for self-signed certificates). |
| `-o` | Outputs results to a file (e.g., `-o results.txt`). |
| `-s` | Filters by HTTP status codes (e.g., `-s 200,204` to show only 200 or 204 responses). |
| `-e` | Expands results to show full URLs (e.g., `-e` for detailed output). |

## Advanced Usage

| Command | Description | Example |
|---------|-------------|---------|
| `gobuster dir -u <url> -w <wordlist> -x php,html -t 50 -k` | Directory brute-forcing with extensions, threads, and SSL bypass. | `gobuster dir -u https://example.com -w common.txt -x php,html -t 50 -k` |
| `gobuster dns -d <domain> -w <wordlist> -t 100 -o subdomains.txt` | Aggressive subdomain enumeration with high threads and output. | `gobuster dns -d example.com -w subdomains-top1million-5000.txt -t 100 -o subdomains.txt` |
| `gobuster vhost -u <url> -w <wordlist> -t 30 -s 200` | Virtual host scanning with thread count and status filter. | `gobuster vhost -u http://10.10.10.10 -w vhosts.txt -t 30 -s 200` |

## Wordlists

- **Directories/Files**: `/usr/share/wordlists/dirb/common.txt`, `/opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt`
- **Subdomains**: `/opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt`
- **Virtual Hosts**: Custom lists like `vhosts.txt` with common names (e.g., `admin`, `dev`).

## Tips

- Use `-t 10-100` based on target sensitivity to avoid detection.
- Combine with Nmap to verify open ports before scanning.
- Check `robots.txt` for hints to seed wordlists.

## Links

- Gobuster: [https://github.com/OJ/gobuster](https://github.com/OJ/gobuster)
- SecLists: [https://github.com/danielmiessler/SecLists](https://github.com/danielmiessler/SecLists)