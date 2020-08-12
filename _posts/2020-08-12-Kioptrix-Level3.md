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
