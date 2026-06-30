# Reconnaissance

## Overview

- Work from quietest to loudest (passive, active, internal)
- Ensure correct rate limiting of scans
- Ensure any requests contain required headers to notify the target per target scope

### Web Reconnaissance Goals

- **Identify Assets**: Discovering all associated domains, subdomains, and IP addresses provides a map of the target's online presence.
- **Uncover Hidden Information**: Web reconnaissance aims to uncover directories, files, and technologies that are not readily apparent and could serve as entry points for an attacker.
- **Analyse the Attack Surface**: By identifying open ports, running services, and software versions, you can assess the potential vulnerabilities and weaknesses of the target.
- **Gather Intelligence**: Collecting information about employees, email addresses, and technologies used can aid in social engineering attacks or identifying specific vulnerabilities associated with certain software.

## Passive Reconnaissance

| Technique            | Description            | Example                  | Tools           | Risk          |
|-----------------------|------------------------|--------------------------|-----------------|---------------|
| Search Engine Queries | Uncovers public data.  | Google for employee names.| Google, Shodan  | Very Low      |
| WHOIS Lookups         | Retrieves domain details. | whois on example.com.   | whois, online   | Very Low      |
| DNS Analysis          | Identifies DNS records.| dig for subdomains.      | dig, dnsenum    | Very Low      |
| Web Archive Analysis  | Reviews historical sites. | Wayback for old pages.  | Wayback Machine | Very Low      |
| Social Media Analysis | Gathers social profiles.| LinkedIn for employees.  | LinkedIn, OSINT | Very Low      |
| Code Repositories     | Analyses public code.  | GitHub for credentials.  | GitHub, GitLab  | Very Low      |

## Active Reconnaissance

| Technique         | Description            | Example                  | Tools           | Risk          |
|-------------------|------------------------|--------------------------|-----------------|---------------|
| Port Scanning     | Identifies open ports. | Nmap scans ports 80, 443.| Nmap, Masscan   | High          |
| Vulnerability Scanning | Probes for vulnerabilities. | Nessus checks for XSS.   | Nessus, Nikto   | High          |
| Network Mapping   | Maps network topology. | Traceroute tracks hops.  | Traceroute, Nmap| Medium-High   |
| Banner Grabbing   | Retrieves service banners. | curl checks HTTP version.| Netcat, curl    | Low           |
| OS Fingerprinting | Detects OS type.       | Nmap uses -O.            | Nmap, Xprobe2   | Low           |
| Service Enumeration | Identifies service versions. | Nmap -sV on port 80. | Nmap            | Low           |
| Web Spidering     | Crawls website structure. | Burp Spider maps pages.  | Burp, ZAP Spider| Low-Medium    |

## Internal Scans

Once initial foothold gained (ssh creds for example)
```bash
netstat -tulpn | grep LISTEN
nmap localhost
```

## Methods and Tools

### nmap

#### Scan Types

Common Nmap scan types and flags with examples:

| Scan Type | Description | Flag |
|-----------|-------------|------|
| TCP SYN Scan | Stealthy half-open scan for TCP ports. Example: `nmap -sS -T4 192.168.1.1` | -sS |
| TCP Connect Scan | Full TCP connection scan. Example: `nmap -sT -p 1-1000 192.168.1.1` | -sT |
| UDP Scan | Scan for UDP ports. Example: `nmap -sU -sV 192.168.1.1` | -sU |
| Ping Scan | Host discovery without port scanning. Example: `nmap -sn 192.168.1.0/24` | -sn (or -sP) |
| Version Detection | Identify service versions on open ports. Example: `nmap -sV -p 80,443 192.168.1.1` | -sV |
| OS Detection | Fingerprint operating system. Example: `nmap -O 192.168.1.1` | -O |
| Script Scan | Run NSE scripts for additional info. Example: `nmap -sC 192.168.1.1` | -sC |
| Aggressive Scan | Combines version, OS, script, and traceroute. Example: `nmap -A 192.168.1.1` | -A |
| Idle/Zombie Scan | Stealth scan using spoofed IP. Example: `nmap -sI zombie.host 192.168.1.1` | -sI |

#### Commonly Used Flags

Additional flags to customise Nmap scans:

| Flag | Purpose |
|------|----------|
| `-p` | Defines a port range or specific ports to scan (e.g., `-p 1-1000` for ports 1-1000, `-p 80,443` for specific ports). |
| `-T<0-5>` | Sets timing template (0=paranoid to 5=insane) to control scan speed (e.g., `-T4` for faster scans). |
| `-A` | Enables aggressive scan features (version detection, OS detection, scripting, traceroute). |
| `-oN` | Outputs scan results to a normal file (e.g., `-oN scan.txt`). |
| `-v` | Increases verbosity for detailed output during the scan. |
| `-Pn` | Treats all hosts as online, skipping host discovery (useful for firewalls). |
| `-iL` | Reads targets from a file (e.g., `-iL targets.txt` for a list of IPs). |

### WHOIS

```bash
whois example.com
```
**Note**: WHOIS data can be inaccurate or intentionally obscured, so verify from multiple sources. Privacy services may mask the true owner.

### DNS

```bash
dig example.com A
```
This retrieves the A record (hostname to IPv4 address). The output includes the IP address and query details.

#### DNS Record Types

| Record Type | Full Name               | Description                       |
|-------------|-------------------------|-----------------------------------|
| A           | Address Record          | Maps a hostname to its IPv4 address. |
| AAAA        | IPv6 Address Record     | Maps a hostname to its IPv6 address. |
| CNAME       | Canonical Name Record   | Creates an alias for a hostname, pointing to another hostname. |
| MX          | Mail Exchange Record    | Specifies the mail server(s) responsible for handling email. |
| NS          | Name Server Record      | Delegates a DNS zone to an authoritative name server. |
| TXT         | Text Record             | Stores arbitrary text information, often for verification or policies. |
| SOA         | Start of Authority Record | Specifies administrative details about a DNS zone. |
| SRV         | Service Record          | Defines hostname and port for specific services. |
| PTR         | Pointer Record          | Used for reverse DNS lookups, mapping an IP to a hostname. |

#### DNS Tools

| Tool                | Key Features                     | Use Cases                      |
|---------------------|----------------------------------|--------------------------------|
| dig                 | Versatile DNS lookup, multiple query types | Manual queries, zone transfers, troubleshooting |
| nslookup            | Simple DNS lookup for A, AAAA, MX | Basic resolution, mail checks  |
| host                | Concise DNS lookup               | Quick A, AAAA, MX checks       |
| dnsenum             | Automated enumeration, brute-forcing | Subdomain discovery, DNS data  |
| fierce              | Subdomain enumeration, wildcard detection | DNS recon, target identification |
| dnsrecon            | Multi-technique enumeration, various outputs | Comprehensive subdomain analysis |
| theHarvester        | OSINT from DNS and other sources | Email, employee data collection |
| Online DNS Lookup   | User-friendly lookup interfaces  | Quick lookups, domain checks   |

### Subdomains

#### Subdomain Enumeration 

| Approach | Examples |
|----------|----------|
| Passive Enumeration | Certificate Transparency (CT) logs, search engine queries |
| Active Enumeration | Brute-forcing, DNS zone transfers |

#### Brute-Force Enumeration Tools

| Tool      | Description                          |
|-----------|--------------------------------------|
| dnsenum   | Comprehensive DNS enumeration        |
| fierce    | User-friendly subdomain discovery    |
| dnsrecon  | Versatile multi-technique enumeration|
| amass     | Actively maintained subdomain tool   |
| assetfinder | Lightweight subdomain finder       |
| puredns   | Powerful DNS brute-forcing           |

**Examples:**

```bash
dnsenum --enum inlanefreight.com:55209 -f /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -r
```
**Note:** -r enables recursive subdomain brute-forcing.

```bash
ffuf -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt:FUZZ -u http://FUZZ:inlanefreight.htb:55291 
```

```bash
gobuster dns -do http://inlanefreight.htb:55291 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt
```

#### DNS Zone Transfers

```bash
dig @nsztm1.digi.ninja zonetransfer.me axfr
```
- dig: DNS lookup utility.
- @ns1.example.com: Specifies the nameserver to query (e.g., ns1.example.com).
- example.com: Domain to perform the lookup on.
- axfr: Requests a zone transfer, retrieving all DNS records for the domain.

**Note**
- Many DNS servers restrict zone transfers to authorised servers.
- zonetransfer.me is a service specifically setup to demonstrate the risks of zone transfers so that the dig command will return the full zone record.

### Virtual Hosts

#### Virtual Host Discovery Tools

| Tool        | Description         | Features                |
|-------------|---------------------|-------------------------|
| [gobuster](tools-and-protocols/gobuster.md)    | Multi-purpose brute-forcer | Fast, custom wordlists  |
| Feroxbuster | Rust-based fuzzer   | Recursion, filters      |
| ffuf        | Fast web fuzzer     | Custom input, filtering |

**Examples:**
```bash
gobuster vhost -u http://192.0.2.1 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt --append-domain
gobuster vhost -u http://web1337.inlanefreight.htb:8888 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -t 60 --append-domain
```

**Common flags:**

| Flag | Description                     |
|------|---------------------------------|
| -t   | Increase threads for faster scanning |
| -k   | Ignore SSL/TLS certificate errors |
| -o   | Save output to a file            |

### Certificate Transparency (CT) Logs
CT logs record SSL/TLS certificates, revealing subdomains.

#### Searching CT Logs

| Tool   | Key Features                     | Use Cases                     | Pros             | Cons                  |
|--------|----------------------------------|--------------------------------|------------------|-----------------------|
| crt.sh | User-friendly interface, domain search, SAN details | Quick subdomain checks, certificate history | Free, no registration | Limited filtering     |
| Censys | Advanced filtering, IP/certificate search | In-depth analysis, misconfigurations | Extensive data, API | Requires registration |

**Example:**
```bash
curl -s "https://crt.sh/?q=facebook.com&output=json" | jq -r '.[].name_value' | sed 's/\*\.//g' | sort -u
```

### Fingerprinting 

#### Techniques

| Technique             | Description                                      |
|-----------------------|--------------------------------------------------|
| Banner Grabbing       | Analyses server banners for software and version details. |
| Analysing HTTP Headers| Examines headers (e.g., Server, X-Powered-By) for tech info. |
| Probing for Responses | Sends crafted requests to elicit unique tech-specific responses. |
| Analysing Page Content| Reviews page elements (e.g., copyright) for tech clues. |

#### Tools

| Tool     | Description                       | Features                          |
|----------|-----------------------------------|-----------------------------------|
| Wappalyzer| Browser extension for tech profiling | Identifies CMSs, frameworks, analytics |
| BuiltWith | Web tech profiler with reports    | Free/paid plans, detailed stacks  |
| WhatWeb  | Command-line website fingerprinting| Uses signature database           |
| Nmap     | Versatile network scanner         | NSE for specialised fingerprinting|
| Netcraft  | Web security and fingerprinting   | Tech, hosting, security reports   |
| wafw00f  | WAF identification tool           | Detects WAF type and configuration|

#### Banner Grabbing

```bash
curl -I inlanefreight.com
```
**Find CMS info:**
```bash
curl -s http://app.inlanefreight.local/index.php | grep '<meta name="generator"'
```

#### Wafw00f

```bash
pip3 install git+https://github.com/EnableSecurity/wafw00f
wafw00f inlanefreight.com
```

#### Nikto

```bash
sudo apt update && sudo apt install -y perl
git clone https://github.com/sullo/nikto
cd nikto/program
chmod +x ./nikto.pl
nikto -h inlanefreight.com -Tuning b
```
- -h specifies the target host.
- -Tuning b flag tells Nikto to only run the Software Identification modules.

### robots.txt

#### robots.txt Structure
The robots.txt file is a plain text file in the website's root, using records separated by blank lines.

| Component   | Description                                      |
|-------------|--------------------------------------------------|
| User-agent  | Specifies crawler or bot the rules apply to (e.g., * for all, Googlebot, Bingbot). |
| Directives  | Instructions for the specified user-agent.       |

#### Common Directives

| Directive   | Description                          | Example                  |
|-------------|--------------------------------------|--------------------------|
| Disallow    | Blocks bot from paths or patterns.   | Disallow: /admin/        |
| Allow       | Permits specific paths despite Disallow. | Allow: /public/      |
| Crawl-delay | Sets delay (seconds) between requests. | Crawl-delay: 10       |
| Sitemap     | Provides XML sitemap URL.            | Sitemap: sitemap.xml    |

### Well-Known URIs
The .well-known standard (RFC 8615) centralizes website metadata in /.well-known/.

#### Examples

| URI Suffix         | Description                            | Status     | Reference                                   |
|--------------------|----------------------------------------|------------|---------------------------------------------|
| security.txt       | Contact info for vulnerability reports | Permanent  | RFC 9116                                    |
| change-password    | URL for password change                | Provisional| https://w3c.github.io/webappsec-change-password-url/ |
| openid-configuration | OpenID Connect configuration        | Permanent  | http://openid.net/specs/openid-connect-discovery-1_0.html |
| assetlinks.json    | Verifies digital asset ownership       | Permanent  | https://github.com/google/digitalassetlinks/blob/master/well-known/specification.md |
| mta-sts.txt        | SMTP MTA-STS security policy           | Permanent  | RFC 8461                                    |

[All well-known URIs](https://www.iana.org/assignments/well-known-uris/well-known-uris.xhtml)

### Web Crawling

Web crawling maps a website's structure by following links. Analyse `robots.txt` for hidden directories.

#### Popular Web Crawlers

| Tool            | Description                       |
|-----------------|-----------------------------------|
| Burp Suite Spider | Active crawler for mapping and vulnerability discovery. |
| OWASP ZAP       | Open-source scanner with spider for vulnerability checks. |
| Scrapy          | Python framework for custom, scalable crawling.         |
| Apache Nutch    | Scalable Java crawler for large-scale recon.            |

#### Scrapy:

Using custom Scrapy spider, ReconSpider, output will be in JSON:

```bash
pip3 install scrapy
wget -O ReconSpider.zip https://academy.hackthebox.com/storage/modules/144/ReconSpider.v1.2.zip
unzip ReconSpider.zip 
python3 ReconSpider.py http://inlanefreight.com
```

| Key          | Description                             |
|--------------|-----------------------------------------|
| emails       | Lists email addresses found on the domain. |
| links        | Lists URLs of links found within the domain. |
| external_files | Lists URLs of external files such as PDFs. |
| js_files     | Lists URLs of JavaScript files used by the website. |
| form_fields  | Lists form fields found on the domain (empty in this example). |
| images       | Lists URLs of images found on the domain. |
| videos       | Lists URLs of videos found on the domain (empty in this example). |
| audio        | Lists URLs of audio files found on the domain (empty in this example). |
| comments     | Lists HTML comments found in the source code. |

**Searching Results:**
```bash
cat results.json | jq '.emails'
cat results.json | jq '.comments'
```

#### Spider code example

```python
import scrapy

class ExampleSpider(scrapy.Spider):
    name = "example"
    start_urls = ['http://example.com/']

    def parse(self, response):
        for link in response.css('a::attr(href)').getall():
            if any(link.endswith(ext) for ext in self.interesting_extensions):
                yield {"file": link}
            elif not link.startswith("#") and not link.startswith("mailto:"):
                yield response.follow(link, callback=self.parse)
```

**Extract links:**
```bash
jq -r '.[] | select(.file != null) | .file' example_data.json | sort -u
```


### Search Engine Discovery

#### Google Dorking

| Operator      | Description                       | Example                         | Example Description                          |
|---------------|-----------------------------------|---------------------------------|----------------------------------------------|
| site:         | Limits results to a domain.       | site:example.com               | Find all pages on example.com.               |
| inurl:        | Finds term in URL.                | inurl:login                    | Search for login pages.                      |
| filetype:     | Searches for specific file types. | filetype:pdf                   | Find downloadable PDFs.                      |
| intitle:      | Finds term in title.              | intitle:"confidential report"  | Look for pages titled "confidential report". |
| intext:       | Searches term in body text.       | intext:"password reset"        | Identify pages with "password reset".        |
| cache:        | Shows cached webpage version.     | cache:example.com              | View cached content of example.com.          |
| link:         | Finds pages linking to URL.       | link:example.com               | Identify sites linking to example.com.       |
| related:      | Finds similar websites.           | related:example.com            | Discover sites like example.com.             |
| info:         | Shows webpage summary.            | info:example.com               | Get details about example.com.               |
| define:       | Provides word/phrase definitions. | define:phishing                | Get "phishing" definitions.                  |
| numrange:     | Searches within number range.     | site:example.com numrange:1000-2000 | Find numbers 1000-2000 on example.com. |
| allintext:    | Requires all words in body.       | allintext:admin password reset | Find pages with "admin" and "password reset". |
| allinurl:     | Requires all words in URL.        | allinurl:admin panel           | Find URLs with "admin" and "panel".          |
| allintitle:   | Requires all words in title.      | allintitle:confidential report 2023 | Find titles with "confidential report 2023". |
| AND           | Requires all terms.               | site:example.com AND (inurl:admin OR inurl:login) | Find admin/login on example.com. |
| OR            | Includes any of the terms.        | "linux" OR "ubuntu" OR "debian" | Search for Linux, Ubuntu, or Debian pages.   |
| NOT           | Excludes specified term.          | site:bank.com NOT inurl:login  | Find bank.com pages excluding login.         |
| * (wildcard)  | Represents any character/word.    | site:socialnetwork.com filetype:pdf user* manual | Find user manuals in PDFs. |
| ... (range)   | Finds results in numerical range. | site:ecommerce.com "price" 100..500 | Find products priced 100-500. |
| " " (quotes)  | Searches exact phrase.            | "information security policy" | Find exact "information security policy".   |
| - (minus)     | Excludes terms.                   | site:news.com -inurl:sports   | Find news.com pages excluding sports.        |

[Google Hacking Database](https://www.exploit-db.com/google-hacking-database)

### Web Archives

Web archives like the Wayback Machine store historical website snapshots:

| Feature | Description | Use Case in Reconnaissance |
|---------|-------------|----------------------------|
| Historical Snapshots | View past versions of websites. | Identify past content or functionality. |
| Hidden Directories | Explore removed or hidden directories. | Discover sensitive information or backups. |
| Content Changes | Track changes in content. | Assess security posture evolution. |

### Automating Recon

These frameworks provide a complete suite of tools for web reconnaissance.

| Framework   | Description                                      |
|-------------|--------------------------------------------------|
| FinalRecon  | Python tool for SSL, Whois, headers, and crawling. |
| Recon-ng    | Python framework for DNS, subdomains, and exploits.|
| theHarvester| Python tool for emails, subdomains, and banners.  |
| SpiderFoot  | OSINT tool for IPs, domains, emails, and scanning.|
| OSINT Framework | Collection of tools for social media and records. |

#### FinalRecon

```bash
git clone https://github.com/thewhiteh4t/FinalRecon.git
cd FinalRecon
pip3 install -r requirements.txt
chmod +x ./finalrecon.py
```

##### Common Flags/Options

| Option         | Argument    | Description                         |
|----------------|-------------|-------------------------------------|
| -h, --help     |             | Show the help message and exit.     |
| --url          | URL         | Specify the target URL.             |
| --headers      |             | Retrieve header information.        |
| --sslinfo      |             | Get SSL certificate information.    |
| --whois        |             | Perform a Whois lookup.             |
| --crawl        |             | Crawl the target website.           |
| --dns          |             | Perform DNS enumeration.            |
| --sub          |             | Enumerate subdomains.               |
| --dir          |             | Search for directories.             |
| --wayback      |             | Retrieve Wayback URLs.              |
| --ps           |             | Perform a fast port scan.           |
| --full         |             | Perform a full reconnaissance scan. |