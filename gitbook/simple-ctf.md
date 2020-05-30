# Simple CTF \(L\)

_TryHackMe_ – [**SimpleCTF**](https://tryhackme.com/room/easyctf) Write-Up

_topics_: web application security, password cracking, fixing public exploits, ftp, ssh, salted hashes

1. **Plan**
2. **Enumeration**
3. **Local Privilege Escalation \(Exploitation\)**
4. **Root Privilege Escalation**

new tools: hash-identifier, gunzip, dcode.fr

tools: nmapAutomator, dirsearch, hydra, searchsploit, python, vim, sudo

## Plan

Not much to plan here, description is "Beginner CTF"

## Enumeration

`export ip=10.10.20.223`

Starting `./nmapAutomator.sh $ip Basic`

![](.gitbook/assets/image.png)

We see that **FTP**, **HTTP** and **SSH** are open on ports **21**, **80** and **2222** respectively. 

Port 80 is running **Apache 2.4.18** lets dirsearch that

`python3 dirsearch.py -u $ip -e php,html,txt`

![](.gitbook/assets/image%20%2826%29.png)

A few interesting things here, a subdirectory and a text file. Lets inspect the txt file and enumerate _/simple_

Inspecting the text file reveals this is a **CUPS server**, reveals the username **mike** and that it was accessed March 19, 2003..

Inspecting _/simple_ reveals **CMS** website, **version 2.2.8**

Inspecting the FTP server seemingly reveals nothing

{% hint style="info" %}
**Remember from the initial scan that Anonymous login is permitted.** 
{% endhint %}

![](.gitbook/assets/image%20%2837%29.png)

#### Lets: 

1. Check for vulnerabilities on the **http** server with an `nmap` scan
2. try `searchsploit` with the **CMS** **version 2.2.8**
3. Login to the **FTP** server as Anonymous
4. enumerate _/simple_
5. Brute forcing the **ssh** port with the username **mike** we found in _/robots.txt_ using `hydra`

### HTTP Server

`./nmapAutomator.sh $ip Vulns`

![A few potential vulnerabilities](.gitbook/assets/image%20%2811%29.png)

### Searchsploit the Website

`searchsploit CMS Simple 2.2`

![Looks vulnerable to SQL injection \(2.2.8\)](.gitbook/assets/image%20%2853%29.png)

### FTP Anonymous Login

`ftp <ip>`

![A username and sensitive message revealed](.gitbook/assets/image%20%286%29.png)

After logging in to the FTP server as anonymous, we have discovered an additional username **mitch** and that we have a **weak password**.

### Subdirectories

Further enumeration of _/simple_

`python3 dirsearch.py -u http://10.10.20.223/simple -e php,html,txt`

![](.gitbook/assets/image%20%285%29.png)

### Brute Forcing SSH

`hydra -t 20 -l mike -P rockyou.txt ssh://10.10.20.223:2222`

![Hydra is taking awhile, lets move on while it runs](.gitbook/assets/image%20%2841%29.png)

There is a lot of information here, one of these will most likely be our **attack vector.** We have two usernames, five or six potential CVEs to use and we have access to five subdirectories. I'm guessing we'll be able to find **ssh** creds through brute forcing with usernames, on _login.php_ or the **CMS Simple SQL injection.** _****_

## Local Privilege Escalation

### SQL Injection

Looking up **CMS Simple 2.2.8** exploit yields **CVE-2019-9053** and an accompanying python script with usage instructions.

Running the script 

`python SQL.py -u http://10.10.218.67/simple --crack -w /usr/share/dirb/wordlists/others/best110.txt`

{% hint style="info" %}
This script gave me errors on the Kali 2020 machine from THM. I ended up booting Kali 2018, got the termcolor error, `pip install termcolor` then run above command
{% endhint %}

![](.gitbook/assets/image%20%2832%29.png)

Per `hash-identifer`, we have an **MD5** hashed password. 

### Decrypting Hashes

Notice the word _Salt_ in the first line, this indicates we have a **salted hash**, a hash accompaned with a salt. A **salt** is random data that is used as an additional input to a one-way function that hashes data, a password or passphrase

![Learned about salted hashes with some help from the THM discord](.gitbook/assets/image%20%2814%29.png)

Putting both hash and salt in the hash decoding website [dcode.fr](https://www.dcode.fr/md5-hash)

![Brute forcing SSH is unnecessary now that we have a password](.gitbook/assets/image%20%2812%29.png)

![and we have a password &amp; low privilege shell](.gitbook/assets/image%20%288%29.png)

### Brute Forcing SSH

Brute forcing **SSH** with `hydra`, using the username mitch instead of mike also yields the password

`gunzip < /usr/share/wordlists/rockyou.txt.gz >> rockyou.txt`

`hydra -P rockyou.txt -l mitch ssh://10.10.218.67:2222`

![](.gitbook/assets/image%20%2827%29.png)

## Root Privilege Escalation

First things first `sudo -l`

![](.gitbook/assets/image%20%2855%29.png)

So any user can use sudo privileges on vim

`sudo /usr/bin/vim`

And as we know to spawn a shell in vim, `:!bash`

![](.gitbook/assets/image%20%284%29.png)

