---
layout: single
title: Forest - Hack The Box
excerpt: "Esta máquina es fácil, aunque necesita mucho fuzzing para encontrar el archivo .mysql_history. Con esa contraseña me conecté por SSH y me conecté a MySQL. Encontré otra credencial, me conecté por SSH y pude ejecutar wine como root. Me creé un shell con msfvenom, lo ejecuté y obtuve una consola como root."
date: 2025-07-16
classes: wide
header:
  teaser: /assets/images/htb-writeup-forest/sonrisu_portada.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:
  - nmap
  - searchsploit
  - netcat
  - msfvenom
---

![](/assets/images/htb-writeup-forest/sonrisu_portada.png)

Esta máquina es fácil, aunque necesita mucho fuzzing para encontrar el archivo .mysql_history. Con esa contraseña me conecté por SSH y me conecté a MySQL. Encontré otra credencial, me conecté por SSH y pude ejecutar wine como root. Me creé un shell con msfvenom, lo ejecuté y obtuve una consola como root.

## Portscan


![](/assets/images/htb-writeup-forest/sunrise.png)


## ENCONTRA 4 PUERTOS

- 22
- 80
- 3306
- 8080

![](/assets/images/htb-writeup-forest/sunrise2.png)


## NO VEMOS NADA RELEVANTE HABLA DE DE EN LOS ENLACES PODEMOS CONTACTAR A SOPORTE ASI ES QUE DEJAMOS


- PUERTO 8080

![](/assets/images/htb-writeup-forest/sunrise3.png)


vamos que el que esta coriendo un  weborf 0.12.2 buscamos en searchsploit y encontre un directory traversal	

![](/assets/images/htb-writeup-forest/sunrise4.png)

y encontramos una archivo .mysql_history

![](/assets/images/htb-writeup-forest/sunrise5.png)


nos conectamos por ssh  con esa credenciales y como recordamos estaba el puerto 3306 abierto veamos si las credenciales funciaban
y resulto ser corecto buscare para ver que encuentro talvez encuentro algo interesante

![](/assets/images/htb-writeup-forest/sunrise6.png)


y vemos otras credenciales y intentamos conectamos por ssh con esas credenciales

![](/assets/images/htb-writeup-forest/sunrise7.png)

gane acceso por ssh y veo que tengo permisos SUID y peudo ejecutar como root wine


![](/assets/images/htb-writeup-forest/sunrise10.png)

como podemos ejecutar cualquier programa como root y ejecutamos un smfvenom que nos madara un consola interactiva como root

![](/assets/images/htb-writeup-forest/sunrise9.png)

nos ponemos en escucha por el puero 80 descargo el smfvenom que cree y lo ejecutanos


![](/assets/images/htb-writeup-forest/sunrise1O.png)


y tenemos una consola como root
