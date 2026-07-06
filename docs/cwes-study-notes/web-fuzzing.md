# Web Fuzzing

## What is Web Fuzzing?

Web fuzzing injects large input sets into web applications to find vulnerabilities, hidden resources, or errors.

| Target                  | Description                     |
|-------------------------|---------------------------------|
| Hidden Directories/Files| Discover unlinked paths         |
| Insecure APIs           | Identify unprotected endpoints  |
| SQL Injection           | Find injectable query points    |
| XSS Vulnerabilities     | Detect script injection flaws   |
| Command Injection       | Uncover system command exploits |

## Installing Tools

### Prerequisits

Installing Go, Python and PIPX

```bash
sudo apt update
sudo apt install -y golang
sudo apt install -y python3 python3-pip
sudo apt install pipx
pipx ensurepath
sudo pipx ensurepath --global
```

### Tools

| Tool       | Description                        | Use Cases                          |
|------------|------------------------------------|------------------------------------|
| FFUF       | Fast Go-based web fuzzer for enumeration. | Directory/file enumeration, parameter discovery, brute-force attacks |
| Gobuster   | Simple, fast web directory fuzzer. | Content discovery, DNS subdomain enumeration, WordPress detection |
| FeroxBuster| Rust-based recursive content discovery tool. | Recursive scanning, unlinked content discovery, high-performance scans |
| wfuzz/wenum| Versatile Python fuzzer for parameter testing. | Directory/file enumeration, parameter discovery, brute-force attacks |

```bash
go install github.com/ffuf/ffuf/v2@latest
go install github.com/OJ/gobuster/v3@latest
curl -sL https://raw.githubusercontent.com/epi052/feroxbuster/main/install-nix.sh | sudo bash -s $HOME/.local/bin
pipx install git+https://github.com/WebFuzzForge/wenum
pipx runpip wenum install setuptools
```

## Miscellaneous Commands

| Command                           | Description                          |
|-----------------------------------|--------------------------------------|
| `sudo sh -c 'echo "SERVER_IP academy.htb" >> /etc/hosts'` | Add DNS entry to hosts file |
| `echo "SERVER_IP academy.htb" \| sudo tee -a /etc/hosts` | Add DNS entry to hosts file |
| `for i in $(seq 1 1000); do echo $i >> ids.txt; done` or ```seq 0 1000 > numbers.txt``` | Create numerical wordlist (1-1000) |
| `curl http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=key' -H 'Content-Type: application/x-www-form-urlencoded'` | Send POST request with data  |

## SecLists Wordlists

| Wordlist                                      | Description                       |
|-----------------------------------------------|-----------------------------------|
| `/usr/share/seclists/Discovery/Web-Content/common.txt` | Common directory/file names       |
| `/usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt` | Extensive directory names  |
| `/usr/share/seclists/Discovery/Web-Content/raft-large-directories.txt` | Large directory collection  |
| `/usr/share/seclists/Discovery/Web-Content/big.txt` | Comprehensive directory/files  |

## Web Fuzzing Tips

| Tip                    | Explanation                          |
|------------------------|--------------------------------------|
| Choose Wordlists       | Select relevant wordlists for target |
| Combine Wordlists      | Use multiple wordlists for breadth   |
| Customize Wordlists    | Tailor based on target knowledge     |
| Monitor Performance    | Adjust for resource-intensive lists  |
| Use Community Resources| Leverage updated community wordlists |

## Parameter Fuzzing

- Altering a product ID in a shopping cart URL could reveal pricing errors or unauthorized access to other users' orders.
- Modifying a hidden parameter in a request might unlock hidden features or administrative functions.
- Injecting malicious code into a search query could expose vulnerabilities like Cross-Site Scripting (XSS) or SQL Injection (SQLi).

### Rest API Parameter Types

| Parameter Type       | Description                                      | Example                          |
|----------------------|--------------------------------------------------|----------------------------------|
| Query Parameters     | Appended to URL after ? for filtering/sorting/pagination | /users?limit=10&sort=name |
| Path Parameters      | Embedded in URL for specific resources           | /products/{id}pen_spark          |
| Request Body Parameters | Sent in body of POST/PUT/PATCH for create/update | { "name": "New Product", "price": 99.99 } |

### GraphQL 

#### Components

| Component      | Description                              | Example                  |
|----------------|------------------------------------------|--------------------------|
| Field          | Retrieves specific data (e.g., name, email) | name, email              |
| Relationship   | Indicates connections between data types | posts                    |
| Nested Object  | Field returning another object for deeper traversal | posts { title, body }    |
| Argument       | Modifies query/field behavior (e.g., filter, sort) | posts(limit: 5)          |

**Example**

```json
query {
  user(id: 123) {
    name
    email
    posts(limit: 5) {
      title
      body
    }
  }
}
```

#### Mutations

| Component  | Description                                      | Example                          |
|------------|--------------------------------------------------|----------------------------------|
| Operation  | Action to perform (e.g., createPost, updateUser) | createPost                       |
| Argument   | Input data for the operation (e.g., title, body) | title: "New Post", body: "This is the content of the new post" |
| Selection  | Fields to retrieve in response (e.g., id, title) |                                   |

**Example**

```json
mutation {
  createPost(title: "New Post", body: "This is the content of the new post") {
    id
    title
  }
}
```

### Discovering Parameters

- API documentation (Rest/graphql)
- WSDL Analysis (SOAP)
- Introspection (graphql)
- Network traffic analysis
- Parameter name fuzzing

### Fuzzing APIs

```bash
git clone https://github.com/PandaSt0rm/webfuzz_api.git
cd webfuzz_api
python3 -m venv .venv
source .venv/bin/activate
pip3 install -r requirements.txt
python3 api_fuzzer.py http://IP:PORT
```

## FFUF

### Flags

| Flag    | Use                            | Example                     |
|---------|--------------------------------|-----------------------------|
| -u      | Specify target URL             | -u http://example.com/FUZZ  |
| -w      | Set wordlist file              | -w wordlist.txt             |
| -ic     | Ignore wordlist comments       | -ic                         |
| -H      | Add custom HTTP headers        | -H "Authorization: Bearer token" |
| -X      | Set HTTP method                | -X POST                     |
| -e      | Extend wordlist with extensions | -e .php,.html              |
| -s      | Enable silent mode             | -s                          |
| -v      | Increase verbosity             | -v                          |
| -t      | Set number of threads          | -t 50                       |
| -k      | Ignore SSL/TLS errors          | -k                          |
| -o      | Output results to file         | -o results.txt              |
| -timeout | Set request timeout in seconds | -timeout 30                |
| -recursion | Enable recursive directory scanning | -recursion          |
| -recursion-depth | Set maximum recursion depth   | -recursion-depth 2  |
| -s      | Filter by status codes         | -s 200,404                  |
| -mc     | Match by status codes          | -mc 200                     |
| -ml     | Match by line count            | -ml 50                      |
| -mw     | Match by word count            | -mw 100                     |
| -ms     | Match by size in bytes         | -ms 1024                    |
| -fc     | Filter by status codes         | -fc 404                     |
| -fl     | Filter by line count           | -fl 0                       |
| -fw     | Filter by word count           | -fw 0                       |
| -fs     | Filter by size in bytes        | -fs 512                     |
| -ac     | Automatically calibrate filtering | -ac                      |

| Command                           | Description                     |
|-----------------------------------|---------------------------------|
| `ffuf -u http://example.com/FUZZ` | Basic URL fuzzing               |
| `ffuf -u http://example.com/FUZZ -w wordlist.txt` | Fuzz with wordlist |
| `ffuf -u http://example.com/FUZZ -w wordlist.txt -ic` | Ignore wordlist comments |
| `ffuf -u http://example.com/FUZZ -w wordlist.txt -c` | Colorize output |
| `ffuf -u http://example.com/FUZZ -w wordlist.txt -mc 200` | Filter by status code |
| `ffuf -u http://example.com/FUZZ -w wordlist.txt -mr "Welcome"` | Filter by regex pattern |
| `ffuf -u http://example.com/FUZZ -w wordlist.txt -e .php,.html` | Add extensions |
| `ffuf -u http://example.com/FUZZ -w wordlist.txt -t 50` | Set threads (50) |
| `ffuf -u http://example.com/FUZZ -w wordlist.txt -x http://127.0.0.1:8080` | Use proxy |
| `ffuf -u http://94.237.53.81:52991/post.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "y=FUZZ" -w /usr/share/seclists/Discovery/Web-Content/common.txt -mc 200 -v` |  POST parameter fuzzing, match status code 200 |

### Examples

| Command | Description |
|---------|-------------|
| `ffuf -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -ic -u http://IP:PORT/FUZZ -t 80` | Directory fuzzing, set to 80 threads |
| `ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -ic -u http://IP:PORT/w2ksvrus/FUZZ -e .php,.html,.txt,.bak,.js -v -rate 500` | File fuzzing, verbose output, limit to 500 request per second |
| `ffuf -w /usr/share/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -ic -v -u http://IP:PORT/FUZZ -e .php,.txt,.html -recursion -recursion-depth 2` | Recursive fuzzing, limit depth to 2 |
| `ffuf -u http://STMIP:STMPO/FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt -recursion -e .php,.txt,.html` | Recursive directory fuzzing |
| `ffuf -u http://STMIP:STMPO/admin/panel.php?accessID=FUZZ -w /usr/share/seclists/Discovery/Web-Content/common.txt -fw 8` | Value fuzzing |
| `ffuf -u http://fuzzing_fun.htb:STMPO -w /usr/share/seclists/Discovery/Web-Content/common.txt -H 'Host: FUZZ.fuzzing_fun.htb:STMPO' -ac` | Vhost fuzzing |


## Gobuster

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

| Command                           | Description                     |
|-----------------------------------|---------------------------------|
| `gobuster dir -u http://example.com -w wordlist.txt` | Directory fuzzing |
| `gobuster dir -u http://example.com -w wordlist.txt -x .php,.html` | Fuzz with extensions |
| `gobuster dir -u http://example.com -w wordlist.txt -s 200` | Filter by status code (only includes passed status code) |
| `gobuster dir -u http://example.com -w wordlist.txt -b 200` | Filter out status code (excludes passed status code) |
| `gobuster dir -u http://example.com -w wordlist.txt --exclude-length 0,404` | Filter out lengths (0 and 404 bytes filtered out) |
| `gobuster dir -u http://example.com -w wordlist.txt -t 50` | Set threads (50) |
| `gobuster dir -u http://example.com -w wordlist.txt -o results.txt` | Save output |
| `gobuster dns -d example.com -w subdomains.txt` | DNS subdomain fuzzing |
| `gobuster dns -d example.com -w subdomains.txt -i` | Show IPs for subdomains |
| `gobuster dns -d example.com -w subdomains.txt -z` | Silent mode    |

### Examples

| Command | Description |
|---------|-------------|
| `gobuster vhost -u http://inlanefreight.htb:44915 -w /usr/share/seclists/Discovery/Web-Content/common.txt --append-domain` | vhost fuzzing, `--append-domain` instructs Gobuster to append the base domain (inlanefreight.htb) to each word in the wordlist |
| `gobuster dns -d inlanefreight.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt` | Subdomain fuzzing |

## Wenum

| Flag | Description | Example Scenario |
|------|-------------|------------------|
| --hc (hide code)     | Exclude responses that match the specified status codes                     | After fuzzing, the server returned many 400 Bad Request errors Use --hc 400 to hide them and focus on other responses |
| --sc (show code)     | Include only responses that match the specified status codes                | You are only interested in successful requests (200 OK) Use --sc 200 to filter the results accordingly |
| --hl (hide length)   | Exclude responses with the specified content length (in lines)              | The server returns verbose error messages with many lines Use --hl with a high value to hide these and focus on shorter responses |
| --sl (show length)   | Include only responses with the specified content length (in lines)         | You suspect a specific response with a known line count is related to a vulnerability Use --sl to pinpoint it |
| --hw (hide word)     | Exclude responses with the specified number of words                        | The server includes common phrases in many responses Use --hw to filter out responses with those word counts |
| --sw (show word)     | Include only responses with the specified number of words                   | You are looking for short error messages Use --sw with a low value to find them |
| --hs (hide size)     | Exclude responses with the specified response size (in bytes or characters) | The server sends large files for valid requests Use --hs to filter out these large responses and focus on smaller ones |
| --ss (show size)     | Include only responses with the specified response size (in bytes or characters) | You are looking for a specific file size Use --ss to find it |
| --hr (hide regex)    | Exclude responses whose body matches the specified regular expression       | Filter out responses containing the "Internal Server Error" message Use --hr "Internal Server Error" |
| --sr (show regex)    | Include only responses whose body matches the specified regular expression  | Filter for responses containing the string "admin" using --sr "admin" |
| --filter/--hard-filter | General-purpose filter to show/hide responses or prevent their post-processing using a regular expression | --filter "Login" will show only responses containing "Login", while --hard-filter "Login" will hide them and prevent any plugins from processing them |

| Command                           | Description                     |
|-----------------------------------|---------------------------------|
| `wenum -w /usr/share/seclists/Discovery/Web-Content/common.txt --hc 404 -u "http://94.237.53.81:52991/get.php?x=FUZZ"` | Parameter fuzzing, exclude 404 |
| `wenum -c -z file,wordlist.txt -d 'username=FUZZ&password=secret'` | Fuzz POST data |
| `wenum -c -z file,wordlist.txt -b 'session=12345'` | Use cookie     |
| `wenum -c -z file,wordlist.txt -H 'User-Agent: Wenum'` | Add custom header |
| `wenum -c -z file,wordlist.txt -t 50` | Set threads (50) |
| `wenum -c -z file,wordlist.txt -u http://example.com/FUZZ -X PUT` | Fuzz with PUT method |
| `wenum -c -z file,wordlist.txt --hl 50` | Filter by content length  |
| 

## Feroxbuster

| Flag | Description | Example Scenario |
|------|-------------|------------------|
| --dont-scan (Request) | Exclude specific URLs or patterns from being scanned (even if found in links during recursion) | You know the /uploads directory contains only images, so you can exclude it using --dont-scan /uploads |
| -S, --filter-size    | Exclude responses based on their size (in bytes) You can specify single sizes or comma-separated ranges | You've noticed many 1KB error pages Use -S 1024 to exclude them |
| -X, --filter-regex   | Exclude responses whose body or headers match the specified regular expression | Filter out pages with a specific error message using -X "Access Denied" |
| -W, --filter-words   | Exclude responses with a specific word count or range of word counts        | Eliminate responses with very few words (e.g., error messages) using -W 0-10 |
| -N, --filter-lines   | Exclude responses with a specific line count or range of line counts        | Filter out long, verbose pages with -N 50- |
| -C, --filter-status  | Exclude responses based on specific HTTP status codes This operates as a denylist | Suppress common error codes like 404 and 500 using -C 404,500 |
| --filter-similar-to  | Exclude responses that are similar to a given webpage                       | Remove duplicate or near-duplicate pages based on a reference page using --filter-similar-to error.html |
| -s, --status-codes   | Include only responses with the specified status codes This operates as an allowlist (default: all) | Focus on successful responses using -s 200,204,301,302 |

| Command                           | Description                     |
|-----------------------------------|---------------------------------|
| `feroxbuster -u http://example.com -w wordlist.txt` | Basic URL fuzzing |
| `feroxbuster -u http://example.com -w wordlist.txt -e` | Include extensions |
| `feroxbuster -u http://example.com -w wordlist.txt -x 404` | Exclude 404 responses |
| `feroxbuster -u http://example.com -w wordlist.txt -t 50` | Set threads (50) |
| `feroxbuster -u http://example.com -w wordlist.txt --depth 3` | Set recursion depth (3) |
| `feroxbuster -u http://example.com -w wordlist.txt -o results.txt` | Save output |
| `feroxbuster -u http://example.com -w wordlist.txt --no-recursion` | Disable recursion |
| `feroxbuster -u http://example.com -w wordlist.txt --url-redirect` | Follow redirects |

## Web API Types

| Type   | Description                     | Features                           |
|--------|---------------------------------|------------------------------------|
| REST   | Uses HTTP methods for resources | JSON/XML, stateless, scalable, caching |
| SOAP   | XML-based protocol for services | XML, stateful/stateless, WS-Security |
| GraphQL| Query language for APIs         | JSON, single endpoint, flexible queries |

## REST Fuzzing Tips

| Tip                    | Explanation                          |
|------------------------|--------------------------------------|
| Test All Methods       | Check GET, POST, PUT, DELETE for vulns |
| Validate Input Fields  | Fuzz inputs with unexpected data     |
| Examine Errors         | Analyze messages for info disclosure |
| Test Authentication    | Check for weak auth controls         |
| Explore Rate Limits    | Test throttling for bypasses         |
| Use Comprehensive Payloads | Test SQLi, XSS payloads          |
| Check Representations  | Test JSON/XML for consistency flaws  |

## SOAP Fuzzing Tips

| Tip                    | Explanation                          |
|------------------------|--------------------------------------|
| Analyze WSDL Files     | Understand service operations        |
| Validate XML Schema    | Test for validation flaws            |
| Check XML Injection    | Fuzz for injection vulnerabilities   |
| Test SOAP Headers      | Identify header misconfigurations    |
| Evaluate WS-Security   | Verify security implementations      |
| Test Transport Security| Ensure HTTPS enforcement             |
| Examine SOAP Faults    | Check for info leakage in faults     |

## GraphQL Fuzzing Tips

| Tip                    | Explanation                          |
|------------------------|--------------------------------------|
| Test Query Depth       | Check handling of complex queries    |
| Validate Input Types   | Fuzz arguments for validation flaws  |
| Examine Query Aliasing| Test aliased queries for leakage      |
| Check Introspection    | Ensure no sensitive schema exposure  |
| Assess Authorization   | Verify query access controls         |
| Evaluate Rate Limiting | Test handling of excessive queries   |
| Fuzz Mutations         | Test data-altering queries for flaws |