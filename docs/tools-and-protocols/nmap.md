# nmap

## Scan Types

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

## Commonly Used Flags

Additional flags to customize Nmap scans:

| Flag | Purpose |
|------|----------|
| `-p` | Defines a port range or specific ports to scan (e.g., `-p 1-1000` for ports 1-1000, `-p 80,443` for specific ports). |
| `-T<0-5>` | Sets timing template (0=paranoid to 5=insane) to control scan speed (e.g., `-T4` for faster scans). |
| `-A` | Enables aggressive scan features (version detection, OS detection, scripting, traceroute). |
| `-oN` | Outputs scan results to a normal file (e.g., `-oN scan.txt`). |
| `-v` | Increases verbosity for detailed output during the scan. |
| `-Pn` | Treats all hosts as online, skipping host discovery (useful for firewalls). |
| `-iL` | Reads targets from a file (e.g., `-iL targets.txt` for a list of IPs). |