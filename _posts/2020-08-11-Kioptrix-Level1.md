---
layout: post
title: Kioptrix Level 1
---
[Kioptrix Level 1](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/#)

## Find IP of box
nmap -T5 <LHOST>/24
Kioptrix is running on: x.x.x.104  

## Init NMap Scan
nmap -sC -sV -oA nmap x.x.x.104
 - Default Scripts
 - Service Detection
 - Output all formats
