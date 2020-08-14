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

Replace the '1' and the text after in the URL with `<script>alert(1)</script>` and it executes successfully

![XSS Check](/images/KioptrixL3/XSS1.JPG)

### SQLI Check

We can also check for and SQL injection vulnerability by replacing the 1 with `' or '1'='1` 

![SQLI Check](/images/KioptrixL3/SQLI1.JPG)

### SQLI Exploitation
#### Number of Columns

Continuing with the SQLI attack, we need to find the number of columns in the table. To do this we can enter `order by x--` where x is an integer. Start at 1 and increment by 1 until we get `unkown column`

When x=6, it works:

![SLQI Column correct](/images/KioptrixL3/SQLI2.JPG)

But if we enter 7, it fails:

![SQLI Column failure](/images/KioptrixL3/SQLI3.JPG)

Therefore, we know there are 6 columns in the table

#### Vulnerable Columns

To find these, after `id = 1` we enter `union select 1,2,3,4,5,6` and the numbers that appear in the 'Sub Gallery' section are new to the page and therefore the vulnerable columns

![SQLI Vulnerable Columns](/images/KioptrixL3/SQLI4.JPG)

#### SQL Version

This is simple, just replace either the 2 or 3 from the `union select 1,2,3,4,5,6` injection previously

We are using version 5.0.51a-3ubuntu5.4 which is MySQL

#### Database Name

Replace the `version` from the previous injection with `database`

The database name is gallery

#### User

Replace `database` with `user`

The user is root@localhost

#### Table Name

Replace `user()` with `group_concat(table_name)` and append ` from information_schema.tables where table_schema=database()--` at the end of the URL

This shows all of the table names we have access to

dev_accounts looks most promising so we'll start there

#### Column Name

Replace table from `group_concat(table_name)` with column and 
