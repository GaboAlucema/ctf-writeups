# 🎯 Máquina: Candy
**IP:** 
**OS:** 
**Dificultad:** 

---
## 1. 🔍 Enumeración
```bash
nmap -p- -sC -sV 172.17.0.2
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Home
|_http-server-header: Apache/2.4.58 (Ubuntu)
| http-robots.txt: 17 disallowed entries (15 shown)
| /joomla/administrator/ /administrator/ /api/ /bin/
| /cache/ /cli/ /components/ /includes/ /installation/
|_/language/ /layouts/ /un_caramelo /libraries/ /logs/ /modules/
|_http-generator: Joomla! - Open Source Content Management
MAC Address: 5A:9E:4B:9B:20:1C (Unknown)
```


## 2. 🔓 Explotación (Foothold)
*(¿Cómo logramos entrar? ¿Qué vulnerabilidad o script usamos?)*


## 3. 🚀 Escalada de Privilegios
*(¿Cómo pasamos de ser un usuario normal a ser Administrador o Root?)*


## 4. 🚩 Banderas (Flags)
- **User:** 
- **Root**: 