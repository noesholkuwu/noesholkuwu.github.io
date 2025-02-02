---
layout: single
title: underpass - Hack The Box
excerpt: "Underpass es una máquina de nivel fácil que, para explotarla, necesité hacer mucho fuzzing. Las credenciales venían en el archivo install, y tuve que seguir buscando para encontrar algo más, ya que el panel de autenticación daba problemas. Después de obtener las credenciales y autenticarme, el hash estaba en user"
date: 2020-10-03
classes: wide
header:
  teaser: /assets/images/htb-writeup-underpass/portunderpass.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - nmap
  - dirseach
  - john
---

![](/assets/images/htb-writeup-underpass/portunderpass2.png)

Underpass es una máquina de nivel fácil que, para explotarla, necesité hacer mucho fuzzing. Las credenciales venían en el archivo install, y tuve que seguir buscando para encontrar algo más, ya que el panel de autenticación daba problemas. Después de obtener las credenciales y autenticarme, el hash estaba en user

## Portscan



```
  33   │ ┌──(noesholk㉿noes)-[~]
  34   │ └─$ sudo nmap -p- --open -sCV 10.10.11.48
  35   │ Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-01 06:43 UTC
  36   │ Nmap scan report for 10.10.11.48
  37   │ Host is up (0.15s latency).
  38   │ Not shown: 62741 closed tcp ports (reset), 2792 filtered tcp ports (no-response)
  39   │ Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
  40   │ PORT   STATE SERVICE VERSION
  41   │ 22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.
       │ 0)
  42   │ | ssh-hostkey: 
  43   │ |   256 48:b0:d2:c7:29:26:ae:3d:fb:b7:6b:0f:f5:4d:2a:ea (ECDSA)
  44   │ |_  256 cb:61:64:b8:1b:1b:b5:ba:b8:45:86:c5:16:bb:e2:a2 (ED25519)
  45   │ 80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
  46   │ |_http-title: Apache2 Ubuntu Default Page: It works
  47   │ |_http-server-header: Apache/2.4.52 (Ubuntu)
  48   │ Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  49   │ 
  50   │ Service detection performed. Please report any incorrect results at https://nmap
       │ .org/submit/ .
  51   │ Nmap done: 1 IP address (1 host up) scanned in 114.64 seconds
```

vemos una pagina apache nada del otro mundo


```
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ sudo nmap -sU --top-ports 200  10.10.11.48
   3   │ Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-01 07:48 UTC
   4   │ Nmap scan report for 10.10.11.48
   5   │ Host is up (0.26s latency).
   6   │ Not shown: 197 closed udp ports (port-unreach)
   7   │ PORT     STATE         SERVICE
   8   │ 161/udp  open          snmp
   9   │ 1812/udp open|filtered radius
  10   │ 1813/udp open|filtered radacct
  11   │ 
  12   │ Nmap done: 1 IP address (1 host up) scanned in 235.30 seconds
```

como no encontre nada en la pagina use fuzing contra el servidor y igual no encontre ningun resultado y como la maquina esta hecha para ser explotada era lo mas normal ver el protocolo `sU` bueno no es por presumir pero es por experiencia si no encuentran nada en la pagina y fuzing y la maquina tiene que sre explotada es mas evidente que debe estar la zona por haci decir vulnerable, esta en auditorias siempre echar un vistazo en protocolo udp

```
  15   │ ┌──(noesholk㉿noes)-[~]
  16   │ └─$ snmpbulkwalk -v2c -c public 10.10.11.48
  17   │ iso.3.6.1.2.1.1.1.0 = STRING: "Linux underpass 5.15.0-126-generic #136-Ubuntu SM
       │ P Wed Nov 6 10:38:22 UTC 2024 x86_64"
  18   │ iso.3.6.1.2.1.1.2.0 = OID: iso.3.6.1.4.1.8072.3.2.10
  19   │ iso.3.6.1.2.1.1.3.0 = Timeticks: (2882248) 8:00:22.48
  20   │ iso.3.6.1.2.1.1.4.0 = STRING: "steve@underpass.htb"
  21   │ iso.3.6.1.2.1.1.5.0 = STRING: "UnDerPass.htb is the only daloradius server in th
       │ e basin!"
  22   │ iso.3.6.1.2.1.1.6.0 = STRING: "Nevada, U.S.A. but not Vegas"
  23   │ iso.3.6.1.2.1.1.7.0 = INTEGER: 72
  24   │ iso.3.6.1.2.1.1.8.0 = Timeticks: (1) 0:00:00.01
  25   │ iso.3.6.1.2.1.1.9.1.2.1 = OID: iso.3.6.1.6.3.10.3.1.1
  26   │ iso.3.6.1.2.1.1.9.1.2.2 = OID: iso.3.6.1.6.3.11.3.1.1
  27   │ iso.3.6.1.2.1.1.9.1.2.3 = OID: iso.3.6.1.6.3.15.2.1.1
  28   │ iso.3.6.1.2.1.1.9.1.2.4 = OID: iso.3.6.1.6.3.1
  29   │ iso.3.6.1.2.1.1.9.1.2.5 = OID: iso.3.6.1.6.3.16.2.2.1
  30   │ iso.3.6.1.2.1.1.9.1.2.6 = OID: iso.3.6.1.2.1.49
  31   │ iso.3.6.1.2.1.1.9.1.2.7 = OID: iso.3.6.1.2.1.50
  32   │ iso.3.6.1.2.1.1.9.1.2.8 = OID: iso.3.6.1.2.1.4
  33   │ iso.3.6.1.2.1.1.9.1.2.9 = OID: iso.3.6.1.6.3.13.3.1.3
  34   │ iso.3.6.1.2.1.1.9.1.2.10 = OID: iso.3.6.1.2.1.92
  35   │ iso.3.6.1.2.1.1.9.1.3.1 = STRING: "The SNMP Management Architecture MIB."
  36   │ iso.3.6.1.2.1.1.9.1.3.2 = STRING: "The MIB for Message Processing and Dispatchin
       │ g."
  37   │ iso.3.6.1.2.1.1.9.1.3.3 = STRING: "The management information definitions for th
       │ e SNMP User-based Security Model."
  38   │ iso.3.6.1.2.1.1.9.1.3.4 = STRING: "The MIB module for SNMPv2 entities"
  39   │ iso.3.6.1.2.1.1.9.1.3.5 = STRING: "View-based Access Control Model for SNMP."
  40   │ iso.3.6.1.2.1.1.9.1.3.6 = STRING: "The MIB module for managing TCP implementatio
       │ ns"
  41   │ iso.3.6.1.2.1.1.9.1.3.7 = STRING: "The MIB module for managing UDP implementatio
       │ ns"
  42   │ iso.3.6.1.2.1.1.9.1.3.8 = STRING: "The MIB module for managing IP and ICMP imple
       │ mentations"
  43   │ iso.3.6.1.2.1.1.9.1.3.9 = STRING: "The MIB modules for managing SNMP Notificatio
       │ n, plus filtering."
  44   │ iso.3.6.1.2.1.1.9.1.3.10 = STRING: "The MIB module for logging SNMP Notification
       │ s."
  45   │ iso.3.6.1.2.1.1.9.1.4.1 = Timeticks: (1) 0:00:00.01
  46   │ iso.3.6.1.2.1.1.9.1.4.2 = Timeticks: (1) 0:00:00.01
  47   │ iso.3.6.1.2.1.1.9.1.4.3 = Timeticks: (1) 0:00:00.01
  48   │ iso.3.6.1.2.1.1.9.1.4.4 = Timeticks: (1) 0:00:00.01
  49   │ iso.3.6.1.2.1.1.9.1.4.5 = Timeticks: (1) 0:00:00.01
  50   │ iso.3.6.1.2.1.1.9.1.4.6 = Timeticks: (1) 0:00:00.01
  51   │ iso.3.6.1.2.1.1.9.1.4.7 = Timeticks: (1) 0:00:00.01
  52   │ iso.3.6.1.2.1.1.9.1.4.8 = Timeticks: (1) 0:00:00.01
  53   │ iso.3.6.1.2.1.1.9.1.4.9 = Timeticks: (1) 0:00:00.01
  54   │ iso.3.6.1.2.1.1.9.1.4.10 = Timeticks: (1) 0:00:00.01
  55   │ iso.3.6.1.2.1.25.1.1.0 = Timeticks: (2883253) 8:00:32.53
  56   │ iso.3.6.1.2.1.25.1.2.0 = Hex-STRING: 07 E9 02 01 08 0C 2B 00 2B 00 00 
  57   │ iso.3.6.1.2.1.25.1.3.0 = INTEGER: 393216
  58   │ iso.3.6.1.2.1.25.1.4.0 = STRING: "BOOT_IMAGE=/vmlinuz-5.15.0-126-generic root=/d
       │ ev/mapper/ubuntu--vg-ubuntu--lv ro net.ifnames=0 biosdevname=0
  59   │ "
  60   │ iso.3.6.1.2.1.25.1.5.0 = Gauge32: 0
  61   │ iso.3.6.1.2.1.25.1.6.0 = Gauge32: 217
  62   │ iso.3.6.1.2.1.25.1.7.0 = INTEGER: 0
  63   │ iso.3.6.1.2.1.25.1.7.0 = No more variables left in this MIB View (It is past the
       │  end of the MIB tree)
```

`iso.3.6.1.2.1.1.5.0 = STRING: "UnDerPass.htb is the only daloradius server in the basin!"`

analizando llege a la conclucion que debe aver una aplicaion detras del sirvidor aremos un fuzing con daloradius

## fuzzing

```
   4   │ ┌──(noesholk㉿noes)-[~]
   5   │ └─$ dirsearch -u http://10.10.11.48/daloradius   
   6   │ /usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pk
       │ g_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pk
       │ g_resources.html
   7   │   from pkg_resources import DistributionNotFound, VersionConflict
   8   │ 
   9   │   _|. _ _  _  _  _ _|_    v0.4.3                                                
       │                                            
  10   │  (_||| _) (/_(_|| (_| )                                                         
       │                                            
  11   │                                                                                 
       │                                            
  12   │ Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist
       │  size: 11460
  13   │ 
  14   │ Output File: /home/noesholk/reports/http_10.10.11.48/_daloradius_25-02-01_08-34-
       │ 06.txt
  15   │ 
  16   │ Target: http://10.10.11.48/
  17   │ 
  18   │ [08:34:06] Starting: daloradius/                                                
       │                                            
  19   │ [08:34:12] 200 -  221B  - /daloradius/.gitignore                            
  20   │ [08:34:47] 301 -  319B  - /daloradius/app  ->  http://10.10.11.48/daloradius/app
       │ /
  21   │ [08:34:57] 200 -   24KB - /daloradius/ChangeLog                             
  22   │ [08:35:07] 200 -    2KB - /daloradius/docker-compose.yml                    
  23   │ [08:35:07] 301 -  319B  - /daloradius/doc  ->  http://10.10.11.48/daloradius/doc
       │ /
  24   │ [08:35:07] 200 -    2KB - /daloradius/Dockerfile                            
  25   │ [08:35:32] 301 -  323B  - /daloradius/library  ->  http://10.10.11.48/daloradius
       │ /library/
  26   │ [08:35:33] 200 -   18KB - /daloradius/LICENSE                               
  27   │ [08:36:04] 200 -   10KB - /daloradius/README.md                             
  28   │ [08:36:11] 301 -  321B  - /daloradius/setup  ->  http://10.10.11.48/daloradius/s
       │ etup/
  29   │                                                                              
  30   │ Task Completed                                                                  
       │                                            
  31   │                                                                                 
       │                                            
  32   │ ┌──(noesholk㉿noes)-[~]
  33   │ └─$ dirsearch -u http://10.10.11.48/daloradius/app
  34   │ /usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pk
       │ g_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pk
       │ g_resources.html
  35   │   from pkg_resources import DistributionNotFound, VersionConflict
  36   │ 
  37   │   _|. _ _  _  _  _ _|_    v0.4.3                                                
       │                                            
  38   │  (_||| _) (/_(_|| (_| )                                                         
       │                                            
  39   │                                                                                 
       │                                            
  40   │ Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist
       │  size: 11460
  41   │ 
  42   │ Output File: /home/noesholk/reports/http_10.10.11.48/_daloradius_app_25-02-01_08
       │ -40-32.txt
  43   │ 
  44   │ Target: http://10.10.11.48/
  45   │ 
  46   │ [08:40:32] Starting: daloradius/app/                                            
       │                                            
  47   │ [08:41:25] 301 -  326B  - /daloradius/app/common  ->  http://10.10.11.48/dalorad
       │ ius/app/common/
  48   │ [08:42:59] 301 -  325B  - /daloradius/app/users  ->  http://10.10.11.48/daloradi
       │ us/app/users/
  49   │ [08:43:00] 302 -    0B  - /daloradius/app/users/  ->  home-main.php         
  50   │ [08:43:00] 200 -    2KB - /daloradius/app/users/login.php                   
  51   │                                                                              
  52   │ Task Completed                                                                  
       │                                            
  53   │                                                                                 
       │                   
  54   │ 
  55   │ 
  56   │ ┌──(noesholk㉿noes)-[~]
  57   │ └─$ dirsearch -u http://10.10.11.48/daloradius/app
  58   │ /usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pk
       │ g_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pk
       │ g_resources.html
  59   │   from pkg_resources import DistributionNotFound, VersionConflict
  60   │ 
  61   │   _|. _ _  _  _  _ _|_    v0.4.3                                                
       │                                            
  62   │  (_||| _) (/_(_|| (_| )                                                         
       │                                            
  63   │                                                                                 
       │                                            
  64   │ Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist
       │  size: 11460
  65   │ 
  66   │ Output File: /home/noesholk/reports/http_10.10.11.48/_daloradius_app_25-02-01_08
       │ -40-32.txt
  67   │ 
  68   │ Target: http://10.10.11.48/
  69   │ 
  70   │ [08:40:32] Starting: daloradius/app/                                            
       │                                            
  71   │ [08:41:25] 301 -  326B  - /daloradius/app/common  ->  http://10.10.11.48/dalorad
       │ ius/app/common/
  72   │ [08:42:59] 301 -  325B  - /daloradius/app/users  ->  http://10.10.11.48/daloradi
       │ us/app/users/
  73   │ [08:43:00] 302 -    0B  - /daloradius/app/users/  ->  home-main.php         
  74   │ [08:43:00] 200 -    2KB - /daloradius/app/users/login.php
  75   │ 
  76   │ 
  77   │ ┌──(noesholk㉿noes)-[~]
  78   │ └─$ dirsearch -u http://10.10.11.48/daloradius/doc/install
  79   │ /usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pk
       │ g_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pk
       │ g_resources.html
  80   │   from pkg_resources import DistributionNotFound, VersionConflict
  81   │ 
  82   │   _|. _ _  _  _  _ _|_    v0.4.3                                                
       │                                            
  83   │  (_||| _) (/_(_|| (_| )                                                         
       │                                            
  84   │                                                                                 
       │                                            
  85   │ Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 25 | Wordlist
       │  size: 11460
  86   │ 
  87   │ Output File: /home/noesholk/reports/http_10.10.11.48/_daloradius_doc_install_25-
       │ 02-01_09-11-08.txt
  88   │ 
  89   │ Target: http://10.10.11.48/
  90   │ 
  91   │ [09:11:08] Starting: daloradius/doc/install/                                    
       │                                            
  92   │ [09:12:55] 200 -    8KB - /daloradius/doc/install/INSTALL
```

Subí cómo lo hice, ya que estoy haciendo de nuevo la máquina porque no tenía imágenes y, como no sabía de dónde sacarlas, tuve que hacerla dos veces para subirla a la página. Igual encontré una dirección donde nos podemos autenticar y unas credenciales.

![](/assets/images/htb-writeup-underpass/auntenticacion.png)

![](/assets/images/htb-writeup-underpass/install.png)

![](/assets/images/htb-writeup-underpass/install2.png)

encontramos un hash

```
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ hashcat hash /usr/share/wordlists/rockyou.txt 
   3   │ hashcat (v6.2.6) starting in autodetect mode
   4   │ 
   5   │ OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, LLVM 18.1.8,
       │  SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
   6   │ ================================================================================
       │ ============================================================
   7   │ * Device #1: cpu-haswell-AMD Athlon 3000G with Radeon Vega Graphics, 3157/6379 M
       │ B (1024 MB allocatable), 1MCU
   8   │ 
   9   │ The following 11 hash-modes match the structure of your input hash:
  10   │ 
  11   │       # | Name                                                       | Category
  12   │   ======+============================================================+==========
       │ ============================
  13   │     900 | MD4                                                        | Raw Hash
  14   │       0 | MD5                                                        | Raw Hash
  15   │      70 | md5(utf16le($pass))                                        | Raw Hash
  16   │    2600 | md5(md5($pass))                                            | Raw Hash 
       │ salted and/or iterated
  17   │    3500 | md5(md5(md5($pass)))                                       | Raw Hash 
       │ salted and/or iterated
  18   │    4400 | md5(sha1($pass))                                           | Raw Hash 
       │ salted and/or iterated
  19   │   20900 | md5(sha1($pass).md5($pass).sha1($pass))                    | Raw Hash 
       │ salted and/or iterated
  20   │    4300 | md5(strtoupper(md5($pass)))                                | Raw Hash 
       │ salted and/or iterated
  21   │    1000 | NTLM                                                       | Operating
       │  System
  22   │    9900 | Radmin2                                                    | Operating
       │  System
  23   │    8600 | Lotus Notes/Domino 5                                       | Enterpris
       │ e Application Software (EAS)
  24   │ 
  25   │ Please specify the hash-mode with -m [hash-mode].
  26   │ 
  27   │ Started: Sat Feb  1 22:12:14 2025
  28   │ Stopped: Sat Feb  1 22:12:27 2025
  29   │                                                                                 
       │                                                
  30   │ ┌──(noesholk㉿noes)-[~]
  31   │ └─$ hashcat -m 0 hash /usr/share/wordlists/rockyou.txt
  32   │ hashcat (v6.2.6) starting
  33   │ 
  34   │ OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, LLVM 18.1.8,
       │  SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
  35   │ ================================================================================
       │ ============================================================
  36   │ * Device #1: cpu-haswell-AMD Athlon 3000G with Radeon Vega Graphics, 3157/6379 M
       │ B (1024 MB allocatable), 1MCU
  37   │ 
  38   │ Minimum password length supported by kernel: 0
  39   │ Maximum password length supported by kernel: 256
  40   │ 
  41   │ Hashes: 1 digests; 1 unique digests, 1 unique salts
  42   │ Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
  43   │ Rules: 1
  44   │ 
  45   │ Optimizers applied:
  46   │ * Zero-Byte
  47   │ * Early-Skip
  48   │ * Not-Salted
  49   │ * Not-Iterated
  50   │ * Single-Hash
  51   │ * Single-Salt
  52   │ * Raw-Hash
  53   │ 
  54   │ ATTENTION! Pure (unoptimized) backend kernels selected.
  55   │ Pure kernels can crack longer passwords, but drastically reduce performance.
  56   │ If you want to switch to optimized kernels, append -O to your commandline.
  57   │ See the above message to find out about the exact limits.
  58   │ 
  59   │ Watchdog: Temperature abort trigger set to 90c
  60   │ 
  61   │ Host memory required for this attack: 0 MB
  62   │ 
  63   │ Dictionary cache built:
  64   │ * Filename..: /usr/share/wordlists/rockyou.txt
  65   │ * Passwords.: 14344392
  66   │ * Bytes.....: 139921507
  67   │ * Keyspace..: 14344385
  68   │ * Runtime...: 2 secs
  69   │ 
  70   │ 412dd4759978acfcc81deab01b382403:underwaterfriends        
  71   │                                                           
  72   │ Session..........: hashcat
  73   │ Status...........: Cracked
  74   │ Hash.Mode........: 0 (MD5)
  75   │ Hash.Target......: 412dd4759978acfcc81deab01b382403
  76   │ Time.Started.....: Sat Feb  1 22:14:36 2025 (3 secs)
  77   │ Time.Estimated...: Sat Feb  1 22:14:39 2025 (0 secs)
  78   │ Kernel.Feature...: Pure Kernel
  79   │ Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
  80   │ Guess.Queue......: 1/1 (100.00%)
  81   │ Speed.#1.........:  1485.0 kH/s (0.15ms) @ Accel:512 Loops:1 Thr:1 Vec:8
  82   │ Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
  83   │ Progress.........: 2984448/14344385 (20.81%)
  84   │ Rejected.........: 0/2984448 (0.00%)
  85   │ Restore.Point....: 2983936/14344385 (20.80%)
  86   │ Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
  87   │ Candidate.Engine.: Device Generator
  88   │ Candidates.#1....: underwear63 -> underfalsehope
  89   │ Hardware.Mon.#1..: Util:100%
  90   │ 
  91   │ Started: Sat Feb  1 22:13:11 2025
  92   │ Stopped: Sat Feb  1 22:14:40 2025
```

## ssh

```
username - svcMosh
password - underwaterfriends
```





```
   1   │ ssh svcMosh@underpass.htb
   2   │ svcMosh@underpass.htb's password: 
   3   │ Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-126-generic x86_64)
   4   │ 
   5   │  * Documentation:  https://help.ubuntu.com
   6   │  * Management:     https://landscape.canonical.com
   7   │  * Support:        https://ubuntu.com/pro
   8   │ 
   9   │  System information as of Sat Feb  1 11:44:17 PM UTC 2025
  10   │ 
  11   │   System load:  0.2               Processes:             334
  12   │   Usage of /:   60.4% of 6.56GB   Users logged in:       2
  13   │   Memory usage: 25%               IPv4 address for eth0: 10.10.11.48
  14   │   Swap usage:   0%
  15   │ 
  16   │   => There is 1 zombie process.
  17   │ 
  18   │ 
  19   │ Expanded Security Maintenance for Applications is not enabled.
  20   │ 
  21   │ 0 updates can be applied immediately.
  22   │ 
  23   │ Enable ESM Apps to receive additional future security updates.
  24   │ See https://ubuntu.com/esm or run: sudo pro status
  25   │ 
  26   │ 
  27   │ The list of available updates is more than a week old.
  28   │ To check for new updates run: sudo apt update
  29   │ Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your 
       │ Internet connection or proxy settings
  30   │ 
  31   │ 
  32   │ Last login: Sat Feb  1 23:25:30 2025 from 127.0.0.1
  33   │ svcMosh@underpass:~$ 
  34   │ 
  35   │ 
  36   │ 
  37   │ 
  38   │ 
  39   │ 
  40   │ Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-126-generic x86_64)
  41   │ 
  42   │  * Documentation:  https://help.ubuntu.com
  43   │  * Management:     https://landscape.canonical.com
  44   │  * Support:        https://ubuntu.com/pro
  45   │ 
  46   │  System information as of Sat Feb  1 11:57:30 PM UTC 2025
  47   │ 
  48   │   System load:  0.04              Processes:             247
  49   │   Usage of /:   61.0% of 6.56GB   Users logged in:       2
  50   │   Memory usage: 25%               IPv4 address for eth0: 10.10.11.48
  51   │   Swap usage:   0%
  52   │ 
  53   │   => There is 1 zombie process.
  54   │ 
  55   │ 
  56   │ Expanded Security Maintenance for Applications is not enabled.
  57   │ 
  58   │ 0 updates can be applied immediately.
  59   │ 
  60   │ Enable ESM Apps to receive additional future security updates.
  61   │ See https://ubuntu.com/esm or run: sudo pro status
  62   │ 
  63   │ 
  64   │ The list of available updates is more than a week old.
  65   │ To check for new updates run: sudo apt update
  66   │ Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your 
       │ Internet connection or proxy settings
  67   │ 
  68   │ 
  69   │ 
  70   │ root@underpass:~#
```

no pude explicar bien ya que no tenia images ni nada de la pagina tube que volver a hacerla recien estoy acadando en la 4:04 am y ma tengo que que levantarme tempranocreo que no subire imagen de portada talvez despues
