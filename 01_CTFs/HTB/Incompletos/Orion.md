# 🎯 Máquina: Orion
**IP:** 10.129.244.146
**OS:** Linux
**Dificultad:** Easy

---
## 1. 🔍 Enumeración
**nmap -p 22,80 -sC -sV 10.129.244.146**
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-05 14:44 -0400
Nmap scan report for 10.129.244.146
Host is up (0.15s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.15 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 3e:ea:45:4b:c5:d1:6d:6f:e2:d4:d1:3b:0a:3d:a9:4f (ECDSA)
|_  256 64:cc:75:de:4a:e6:a5:b4:73:eb:3f:1b:cf:b4:e3:94 (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://orion.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

### Hallazgos
* Formulario de contacto en la web el cual no lleva a nada
* la web corre bajo CraftCMS


## 2. 🔓 Explotación (Foothold)
*(¿Cómo logramos entrar? ¿Qué vulnerabilidad o script usamos?)*


## 3. 🚀 Escalada de Privilegios
*(¿Cómo pasamos de ser un usuario normal a ser Administrador o Root?)*


## 4. 🚩 Banderas (Flags)
- **User:** 
- **Root**: 