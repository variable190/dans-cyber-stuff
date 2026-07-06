# JavaScript Deobfuscation

## Terms

| Term | Description |
|------|-------------|
| Code minification | Reduces code to a single line often saved as .min.js. |
| Packing | Obfuscation recognizable from the six function arguments used in the initial function "function(p,a,c,k,e,d)". |
| JSFuck | Programming style based in the atomic parts of JS. Uses only ()+[]! (aa and jj encode are similar). |
| Base 64 | Contains only alpha-numeric, + and /. Padded with '='. |
| Hex | Encodes each char into its hex on the ASCII table. Contains 0-9 and a-f. |
| Ceasar | Shifts each char by a fixed number |
| Rot13 | Most common ceaser cipher, shifts by 13 |

## Commands

| Command | Description |
|---------|-------------|
| `curl http://SERVER_IP:PORT/`     | cURL GET request        |
| `curl -s http://SERVER_IP:PORT/ -X POST` | cURL POST request    |
| `curl -s http://SERVER_IP:PORT/ -X POST -d "param1=sample"` | cURL POST with data |
| `echo hackthebox | base64` | base64 encode |
| `echo ENCODED_B64 | base64 -d` | base64 decode |
| `echo hackthebox | xxd -p` | hex encode |
| `echo ENCODED_HEX | xxd -p -r` | hex decode |
| `echo hackthebox | tr 'A-Za-z' 'N-ZA-Mn-za-m'` | rot13 encode |
| `echo ENCODED_ROT13 | tr 'A-Za-z' 'N-ZA-Mn-za-m'` | rot13 decode |

## Useful Websites

| Site              | Description                          |
|-------------------|--------------------------------------|
| [JS Console](https://jsconsole.com/)      | Interactive JavaScript console for testing code. |
| [JS Minifier](https://www.toptal.com/developers/javascript-minifier) | Minifies JavaScript for performance optimization. |
| [BeatifyTools](https://beautifytools.com/javascript-obfuscator.php) | Obfuscates JavaScript to protect code. |
| [JS Obfuscator Tool](https://obfuscator.io/) | Tool to obfuscate JavaScript code securely. |
| [JSFuck](https://jsfuck.com/)             | Encodes JavaScript using only six characters. |
| [jjencode](https://utf-8.jp/public/jjencode.html) | Encodes JavaScript with JJEncode technique. |
| [aaencode](https://utf-8.jp/public/aaencode.html) | Encodes JavaScript with AAEncode method. |
| [Prettier](https://prettier.io/playground/) | Formats and beautifies JavaScript code. |
| [js-beautify](https://beautifier.io/)     | Beautifies minified or obfuscated JavaScript. |
| [UnPacker](https://matthewfl.com/unPacker.html) | Deobfuscates and unpacks JavaScript code. |
| [rot13.com](https://rot13.com/)           | Applies ROT13 encoding/decoding for text. |
| [Cipher Identifier](https://www.boxentriq.com/code-breaking/cipher-identifier) | Identifies cipher types in encoded text. |

## Misc

| Command/Action | Description |
|----------------|-------------|
| `ctrl+u` | Show HTML source code in Firefox |
| `ctrl+shift+z` | Open browser debugger tab |
| '{ }' button | Pretty print in debugger tab |
| Unpack | Replace eval function with console.log |