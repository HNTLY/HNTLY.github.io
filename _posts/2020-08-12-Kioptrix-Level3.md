---
layout: post
title: Kioptrix Level 3
---
[Kioptrix Level 3](https://www.vulnhub.com/entry/kioptrix-level-12-3,24/)

## Find IP of box
Our IP: x.x.x.49

`nmap -sP <LHOST>/24`

Kioptrix is running on: x.x.x.16  

## /etc/hosts

As mentioned in the Description of the VM on VulnHub, we must add our victim machine IP to our /etc/hosts file

`x.x.x.16 kioptrix3.com`

## Init NMap Scan

`nmap -sC -sV -oA nmap x.x.x.16`

 - Default Scripts
 - Service Detection
 - Output all formats
 
 ![Initial NMap Scan](/images/KioptrixL3/NMap1.JPG)
 
 There are only 2 ports open: 22 (ssh), 80 (http)

## HTTP - Port 80
### Gobuster

`gobuster dir -u http://x.x.x.16 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -o gobuster.txt -x php,html,txt,xml`

![Initial Gobuster Scan](/images/KioptrixL3/gobuster1.JPG)

Navigate to gallery if we use the sorting options drop down menu in the album, the URL changes to contain parameters such as id=1, which suggest we may be able to use XSS.

### XSS Check

Replace the '1' and the text after in the URL with <script>alert(1)</script> and it executes successfully

![XSS Check](/images/KioptrixL3/XSS1.JPG)

### SQLI Check

We can also check for and SQL injection vulnerability by replacing the 1 with [' or '1'='1] without the square brackets

![SQLI Check](/images/KioptrixL3/SQLI1.JPG)
