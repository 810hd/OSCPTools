# Library \(L\)

_TryHackMe_ – [**Library**](https://tryhackme.com/room/bsidesgtlibrary) Write-Up

_topics_: web application security, open ports, password attacks, Linux privilege escalation, python, file manipulation,

1. **Plan**
2. **Enumeration**
3. **Local Privilege Escalation \(Exploitation\)**
4. **Root Privilege Escalation**

new tools: hydra\(officially\)

tools: firefox browser, terminal, nmapAutomator, dirsearch, ssh, sudo

## 1\) Plan

Not much info given about this. Boot2root for FIT and bsides guatemala CTF.

## 2\) Enumeration

following `./nmapAutomator.sh <ip> Basic`

![](.gitbook/assets/0%20%282%29.png)

and following `python3 dirsearch.py -u http://10.10.228.158/ -e php`

![](.gitbook/assets/1%20%281%29.png)

it looks like this machine is running an Ubuntu **Apache** Web Server version **2.4.18** on port 80 with ssh OpenSSH running on **ssh** port 22, version **7.2p2**. There are three subdirectories we have access to \(/images, /index.html, /robots.txt\).

## 3\) Local Privilege Escalation \(Exploitation\)

Using searchsploit with the services and version number return some potential exploits

![](.gitbook/assets/2.png)![](.gitbook/assets/3.png)

but we need a foothold somehow. None of the subdirectories seem to allow a path traversal attack or reveal anything interesting other than the robots.txt, which says **User-agent: rockyou** which is a hint towards the rockyou wordlist. There is nothing on the page except a username **meliodas** and some comments, you cannot post a comment or review them.

There is nothing else on this webpage that would appear to reveal an attack vector. We must go with the only two hints we have, meliodas and rockyou. SSH is open on port 22 and meliodas:rockyou is a username and password combination, lets try to break in with hydra.

`hydra -t 20 -l meliodas -P <rockyou.txt> ssh://10.10.228.158`

Trying this yields a way in. Hydra returns that the password **iloveyou1** is for the user **meliodas**.

![](.gitbook/assets/4.png)

## 4\) Root Privilege Escalation

First things first with root privilege escalation, `sudo -l`

![](.gitbook/assets/5.png)

returns this interesting information. The only sudo command the user can run is `sudo python bak.py`_._ Running the command returns an error that the user cannot run python as sudo. Using the full file path `sudo python /home/meliodas/bak.py`, runs the command which does nothing.

There is nothing against removing the file and changing the contents to include a python shell. `rm bak.py` and `echo 'import pty; pty.spawn("/bin/sh")' > /home/meliodas/bak.py` and run the command again to achieve a root shell.

![](.gitbook/assets/6.png)

