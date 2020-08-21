---
layout: post
title: Fristileaks 1-3
---
[Fristileaks 1-3](https://www.vulnhub.com/entry/fristileaks-13,133/)

## Setup for VMWare

Right click ovf file and open with VMWare. Import and click retry if a pop-up appears.

VM --> Settings --> Network Adaptor

Ensure the machine is on bridged and click Advanced. Enter 08:00:27:A5:A6:76 into the MAC address and click ok. It is now ready to load up.

## Find IP of box
Our IP: x.x.x.49

`nmap -sP <LHOST>/24`

Fristileaks is running x.x.x.23

## Init NMap Scan

`nmap -sC -sV -oA nmap x.x.x.23`

 - Default Scripts
 - Service Detection
 - Output all formats
 
![Initial NMap Scan](/images/Fristileaks1-3/NMap1.JPG) 

 There is one port running: 80 (http)
 
 As there was only one port open, I ran nmap -T5 -p- <LHOST> to scan all ports but to no avail
 
## HTTP
### Gobuster

`gobuster dir -u http://x.x.x.23 -w /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -o gobuster.txt -x php,html,txt,xml`

But the only directory we get that NMap didn't return is /images

### /robots.txt

Dissallowed entries: /cola  /sisi  /beer

If we naviagate to these directories, we get the same image in each

### Image Enum

I downloaded the images and used strings, exiftool and file but gained no information from these.

### /fristi

There's not much more to do now since we have no more ports to enum, no results from dir/gobuster, nothing in the source code. This leads me to believe we are looking for a web directory that isn't in a common wordlist. The image says 'drink fristi' so I googled fristi and realised it's a drink, just like the dissallowed directories in /robots.txt. If we navigate /fristi, we get a login page

The source code gives us a username: eezeepz

It also tells us that images are base64 encoded and at the bottom of the source code, we find what looks to be a base64 encoded string. Pasting this into [CyberChef](https://gchq.github.io/CyberChef/) and decoding from base64 gives us unreadable text but it contains a png header.

I created a file with the base64 text and ran the command `cat fristi.txt | base64 -d > fristi.png`

Here is the image we create:

![eezeepz password](/images/Fristileaks1-3/kekekePass.JPG)
