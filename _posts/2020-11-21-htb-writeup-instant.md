---
layout: single
title: instant - Hack The Box 
excerpt : "Instantes una máquina de Hack The Box centrada en ingeniería inversa y APIs. El flujo es Análisis de la APK Se decompila una aplicación móvil para obtener información como subdominios o claves.Enumeración: Se investigan los subdominios y endpoints de la API descubiertos. Explotación: Se abusa de vulnerabilidades en la API para ganar acceso inicial. Escalada Se buscan archivos sensibles en el sistema (como backups) para obtener privilegios más altos."
date: 2025-01-22
classes: wide
header:
  teaser: /assets/images/htb-writeup-buff/buff_logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - nmap
  - analyze apk 
  - gobuster
  - JWT token
---

![](/assets/images/htb-writeup-buff/buff_logo.png)

Como siempre, comencé con un escaneo inicial de Nmap para identificar puertos abiertos en el host. Una vez completado el escaneo, solo había dos puertos abiertos: 22 y 80.

```
# Nmap 7.94SVN scan initiated Mon Nov  4 21:46:38 2024 as: /usr/lib/nmap/nmap -T4 -p- -vvv -oN scans/instant-initial 10.10.11.37
Nmap scan report for 10.10.11.37
Host is up, received echo-reply ttl 63 (0.072s latency).
Scanned at 2024-11-04 21:46:38 EST for 16s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63
```

nmap -sC -sV -p22,80 -oN scans/instant-openports 10.10.11.37



```
# Nmap 7.94SVN scan initiated Mon Nov  4 21:47:17 2024 as: /usr/lib/nmap/nmap -sC -sV -p22,80 -oN scans/instant-openports 10.10.11.37
Nmap scan report for 10.10.11.37
Host is up (0.053s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 31:83:eb:9f:15:f8:40:a5:04:9c:cb:3f:f6:ec:49:76 (ECDSA)
|_  256 6f:66:03:47:0e:8a:e0:03:97:67:5b:41:cf:e2:c7:c7 (ED25519)
80/tcp open  http    Apache httpd 2.4.58
|_http-title: Did not follow redirect to http://instant.htb/
|_http-server-header: Apache/2.4.58 (Ubuntu)
Service Info: Host: instant.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

http://instant.htb/


![](/assets/images/htb-writeup-instant/inst.webp)

```
cat network_security_config.xml 

<?xml version="1.0" encoding="utf-8"?>

<network-security-config>

    <domain-config cleartextTrafficPermitted="true">

        <domain includeSubdomains="true">mywalletv1.instant.htb</domain>

        <domain includeSubdomains="true">swagger-ui.instant.htb</domain>

    </domain-config>

</network-security-config># 
```
gobuster dir -u "swagger-ui.instant.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -t 50

```
gobuster dir -u "swagger-ui.instant.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-20000.txt -t 50                                                               ⏎
/usr/lib/python3/dist-packages/dirsearch/dirsearch.py:23: DeprecationWarning: pkg_resources is deprecated as an API. See https://setuptools.pypa.io/en/latest/pkg_resources.html
  from pkg_resources import DistributionNotFound, VersionConflict

 

  _|. _ _  _  _  _ _|_    v0.4.3                                                                              

 (_||| _) (/_(_|| (_| )                                                                                       

                                                                                                              

Extensions: php, aspx, jsp, html, js | HTTP method: GET | Threads: 50 | Wordlist size: 11460

Output File: /home/kali/Instant/reports/_swagger-ui.instant.htb/_24-12-20_15-21-03.txt

Target: http://swagger-ui.instant.htb/

[15:21:03] Starting:                                                                                          
[15:21:18] 308 -  263B  - /apidocs  ->  http://swagger-ui.instant.htb/apidocs/
[15:21:36] 403 -  287B  - /server-status                                    
[15:21:36] 403 -  287B  - /server-status/                                   

                                                                             
Task Completed   
```


![](/assets/images/htb-writeup-instant/john.webp)



ahora capturamos con burp site




![](/assets/images/htb-writeup-instant/burp.webp)



Ahora podemos utilizar estas funcionalidades, probando la funcionalidad de la funcionalidad de la .view/logs. crea el archivo llamado 1.log en ./home/shirohige/logs/1.log.
vamos a leerlo usando "/read/log.




![](/assets/images/htb-writeup-instant/burp2.webp)




![](/assets/images/htb-writeup-instant/burp3.webp)


ya sabiendo que tenemos una forma de ver el id_rsa de usuario de la dirección de la dirección de usuario y cuando probamos ./view/logs. estábamos en el directorio de usuario de:shirohige.



ssh shirohige@10.10.11.37 -i id-rsa


![](/assets/images/htb-writeup-instant/ssh1.png)


Está encriptado, Para descifrarlo se puede reforzar con python con biblioteca de pycryptodome y lista de palabras o utilizar este github repo:




https://github.com/ItsWatchMakerr/SolarPuttyCracker



![](/assets/images/htb-writeup-instant/ssh2.webp)



La contraseña del usuario raíz está en un archivo descifrado, ahora permite ssh a máquina usando esto:

ssh root 10.10.11.37

![](/assets/images/htb-writeup-instant/ssh3.png)
