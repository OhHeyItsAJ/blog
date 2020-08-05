---
layout: post
title: TryHackMe: Simple CTF
--- 
# Simple CTF
This box eas a great first step into the world of offensive security involving some automated exploits, Linux privilege abuse, and, as always, a good amount of enumeration. You can try it free yourself on [TryHackMe](https://tryhackme.com/room/easyctf)!
## Port Scan
- For the first TryHackMe question of how many ports are running under 1000, scan ports 1 through 1000 with`nmap -sS -p 1-1000 [easyCTF IP]`
![portscan1000](Pasted image 39.png)Here we have ports 21 and 80 open.
- We can further enumerate those services with `nmap -sS -sV -A -p 21,80 [easyCTF IP]`
![portscan1000version](Pasted image 40.png)We have FTP vsftpd 3.0.3 (with a #user of FTP) and Apache httpd 2.4.18 running on ubuntu
- For good measure, we can scan the rest of the ports with`nmap -sS -p- [easyCTF IP]`
![portscanall](Pasted image 41.png)We see port 2222 is also open, so we can further enumerate with `nmap -sS -sV -A -p 2222 [simnpleCTF IP]`
![portscanallversion](Pasted image 42.png)We see that the service version is SSH 7.2p2
## HTTP Enumeration
### Visiting the home page
- Just the default Apache page.
## Dirb
- Look for additional URLs with `dirb http://[easyCTF IP]/`
![dirb](Pasted image 44.png)This affords is the robots.txt file as well as the simple directory
![dirboutput](Pasted image 45.png)Further enumeration of the simple directory reveals additional directories
### Robots.txt
![robots](Pasted image 43.png)
We can see a CUPS server is running with a possible #user of mike
### Simple/
#### Root Directory
- Looking at the /simple path returns an installation page with the #artifact version of CMS Made Simple 2.2.8
![simpleroot](Pasted image 47.png)
#### Admin
- The simple/admin path returns the CMS Made Simple application
![simpleadmin](Pasted image 46.png)
---
## FTP -  This was a Rabbit Hole :/
- According to [this forum](https://forum.cmsmadesimple.org/viewtopic.php?f=8&t=74971), version numbers may be found in either root/version.php or lib/version.php
- From our port scan above we should be able to log in with the user FTP
	```
	telnet [Simple CTF IP] 21
	user FTP
	PASV
	LIST
	```
	Unfortunately, LIST times out and no files are returned. Doing some googling, we realize that this may be a firewall issue, which we have very little control over.
---
## Exploiting CMS Made Simple
- A quick google of CMS Made Simple 2.2.8 returns this [SQLi vulnerability](https://www.exploit-db.com/exploits/46635)
- We can download the exploit onto our machine with `wget https://www.exploit-db.com/raw/46635 -O 46635.py` 
- Using the usage instructions, we run the exploit and attempt to crack the password with `python 46635.py -u http://10.10.4.13/simple --crack -w /usr/share/wordlists/rockyou.txt`
![passwordcrack](Pasted image 48.png)This nets us a #user of ==mitch== and a cracked #password of ==secret==
## Logging in with SSH
- We can not try the creds for mitch with `ssh -p 2222 mitch@[Simple CTF IP]`
![sshlogin](Pasted image 49.png)This nets us the user flag!
- We also notice that there another user in the home directory
## Privilege Escalation Enumeration
- Running `sudo -l` reveals that we have no password sudo access to VIM
According to [GTFOBins](https://gtfobins.github.io/gtfobins/vim/?source=post_page---------------------------), we can break out of the shell with `sudo vim -c ':!/bin/sh'`
![privesc](Pasted image 50.png)This gets us the root flag!
![rootflag](Pasted image 51.png)