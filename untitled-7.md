# Anonforce \(L\)

_TryHackMe_ – [**Anonforce**](https://tryhackme.com/room/bsidesgtanonforce) Write-Up

_topics_: web application attacks \(FTP\), password cracking

1. **Plan**
2. **Enumeration**
3. **Password Cracking/Privilege Escalation**

new tools:

tools: terminal, nmapAutomator, firefox browser

## 1. Plan

This box is not a walkthrough. It is described as a boot2root CTF box asking for only the user and root flag.

Because we do not know anything about this box, there is not much to plan for except to parse through enumeration.

## 2. Enumeration

using nmapAutomator, we can implement basic scans:

`./nmapAutomator.sh 10.10.92.214 Basic` which returns that port 21/22 are open, FTP and SSH. The basic scan returns the version numbers and software running on those ports, as well as revealing that anonymous FTP login \(FTP code 230\) is allowed. Because it is running FTP, it must be running software on that port with that protocol. Navigating to **ftp://10.10.92.214/** displays the entire system directory, including /home where user.txt files are usually found.

 PORT STATE SERVICE VERSION

 21/tcp open ftp vsftpd 3.0.3

 22/tcp open ssh OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 \(Ubuntu Linux; protocol 2.0\)

navigating to /_home_/_melodias_ reveals the user.txt, very straightforward.

## 3. Password Cracking/Privilege Escalation

There exists a particular directory on **ftp://10.10.92.214/** called ‘notread’ which is obviously a red flag and something to search in. Cd’ing to this reveals backup PGP key and a private key.

Once we download the files, we can try to decrypt them with the gpg command.

`gpg –import private.asc` asks for a password, so perhaps johntheripper can crack the hash of this file, enabling us to decrypt the keys. First we have to import the private.asc key and convert it with gpg2john.

`gpg --import /media/sf_shareFolder/private.asc`

`gpg2john /media/sf_shareFolder/private.asc > hash`

`john hash`

`john –show hash`

`gpg --import /media/sf_shareFolder/private.asc (enter password)`

`gpg --decrypt /media/sf_shareFolder/backup.pgp`

once GPG decrypts the key, a password hash for root and for the user melodias appears. Because there are hundreds of different types of hashes, we need to identify which hashes we have and crack them with hashcat.

We can identify the types of hash by the $n$ preceding the bunch of characters. The root hash has $6$ syntax while the user hash has $1$ syntax. They are revealed to be **sha512crypt $6$, SHA512 \(Unix\)** and **md5crypt, MD5 \(Unix\), Cisco-IOS $1$ \(MD5\)** respectively from the website \(https://hashcat.net/wiki/doku.php?id=example\_hashes\). Now we can crack these hashes and SSH into the machine as root.

Hashcat and john would not work on a VM. Attempting to break them in Manjaro returned the error: clGetPlatformIDs\(\): CL\_PLATFORM\_NOT\_FOUND\_KHR which I did not have time to troubleshoot. I attempted to list the hashes into online crackers but they could not recognize the syntax:

**root:$6$07nYFaYf$F4VMaegmz7dKjsTukBLh6cP01iMmL7CiQDt1ycIm6a.bsOIBp0DwXVb9XI2EtULXJzBtaMZMNd2tV4uob5RVM0:18120:0:99999:7:::**

**melodias:$1$xDhc6S6G$IQHUW5ZtMkBQ5pUMjEQtL1:18120:0:99999:7:::**

so I was too lazy to try and install OpenCL so hashcat would work. Anyway I just looked up what the cracked hash was and will revist proper ways to crack hashes. Cracking the hash reveals the root password and enables us to SSH into the machine as root.

