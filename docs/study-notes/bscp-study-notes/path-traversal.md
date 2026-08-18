# Path Traversal

`..\..\..\windows\win.ini` windows equivalvent to `../../../etc/passwd`

## Testing

- Try without path traversal `/etc/passwd`
- Find additional requests in the target tab
- Try `Fuzzing - Path Traversal` word list in intruder
- Try including the required base folder `filename=/var/www/images/../../../etc/passwd`
- Try null byte if user-supplied filename required `filename=../../../etc/passwd%00.png`