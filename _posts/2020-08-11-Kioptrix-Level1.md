---
layout: post
title: Kioptrix Level 1
---
[Kioptrix Level 1](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/#)

## Find IP of box
Our IP: x.x.x.49

nmap -T5 <LHOST>/24
 
Kioptrix is running on: x.x.x.104  

## Init NMap Scan
nmap -sC -sV -oA nmap x.x.x.104
 - Default Scripts
 - Service Detection
 - Output all formats

![Initial NMap Scan](/images/KioptrixL1/Nmap1.JPG)

## HTTP - Port 80
### Gobuster
gobuster dir -u http://x.x.x.104 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -o gobuster.txt -x php,html,txt,xml
