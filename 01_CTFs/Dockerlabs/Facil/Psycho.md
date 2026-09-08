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
En el `html` del sitio principal parece haber un error de código dinamico

## 2. 🔓 Explotación (Foothold)
*(¿Cómo logramos entrar? ¿Qué vulnerabilidad o script usamos?)*


## 3. 🚀 Escalada de Privilegios
*(¿Cómo pasamos de ser un usuario normal a ser Administrador o Root?)*


## 4. 🚩 Banderas (Flags)
- **User:** 
- **Root**: 