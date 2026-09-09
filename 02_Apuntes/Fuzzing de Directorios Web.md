# Fuzzing Web: Directorios y Subdominios

**Objetivo:** Descubrir rutas, archivos, directorios y subdominios ocultos en un servidor web mediante ataques de diccionario (fuerza bruta).

**Diccionarios recomendados:**
* **Para directorios:** `/usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt`
* **Para subdominios (DNS/VHosts):** `/usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt`

---

## Códigos de Estado HTTP Clave
Al fuzzear, la herramienta nos devuelve códigos. Los más importantes son:
* **200 OK:** ¡Éxito! El archivo o directorio existe y es accesible.
* **301/302 Redirect:** Existe, pero te está redirigiendo a otro lado.
* **403 Forbidden:** Existe, pero no tienes permisos para verlo (A veces evadible).
* **404 Not Found:** No existe (se suele filtrar automáticamente).
* **500 Internal Server Error:** Error del servidor (a veces útil si rompimos algo intencionalmente).

---

## 1. Fuzzing de Directorios y Archivos

### FFUF (El estándar actual)
La palabra `FUZZ` en mayúsculas indica dónde se inyectará el diccionario.

* **Escaneo básico:**
  `ffuf -w /ruta/al/diccionario.txt -u http://10.10.10.X/FUZZ`
* **Buscando extensiones específicas (.php, .txt, .bak):**
  `ffuf -w /ruta/al/diccionario.txt -u http://10.10.10.X/FUZZ -e .php,.txt,.bak`

### Gobuster (Alternativa sólida)
Si FFUF falla o necesitas confirmar resultados.

* **Escaneo básico:**
  `gobuster dir -u http://10.10.10.X -w /ruta/al/diccionario.txt`
* **Con extensiones:**
  `gobuster dir -u http://10.10.10.X -w /ruta/al/diccionario.txt -x php,txt,html`

---

## 2. Fuzzing de Subdominios (Virtual Hosts)

Utilizado cuando el objetivo esconde paneles (ej. `dev.target.htb` o `admin.target.htb`) bajo la misma dirección IP. 

**Requisito previo:** Debes registrar la IP y el dominio en tu archivo `/etc/hosts` de Linux antes de atacar.

* **Comando FFUF inyectando la cabecera Host (`-H`):**
  `ffuf -w /ruta/al/diccionario_dns.txt -u http://target.htb/ -H "Host: FUZZ.target.htb" -fs [TAMAÑO_BASE]`

*(Nota: En VHosts, casi siempre debes usar el filtro `-fs` porque el servidor devuelve `200 OK` por defecto a todo).*

---

## 3. Banderas (Flags) Esenciales en FFUF

FFUF es extremadamente personalizable. Dominar sus banderas te permite adaptar el ataque a cualquier servidor.

### 3.1 Banderas de Configuración Básica
* **`-w` (Wordlist):** Ruta del diccionario (admite múltiples diccionarios separados por comas).
* **`-u` (URL):** La URL objetivo (debe contener la palabra `FUZZ`).
* **`-c` (Color):** Colorea la salida en la terminal (facilita la lectura rápida).
* **`-t` (Threads):** Hilos concurrentes. Por defecto es 40. Si el servidor se cae o va lento, bájalo (ej. `-t 20`).
* **`-e` (Extensions):** Añade extensiones a cada palabra del diccionario (ej. `-e .php,.txt`).
* **`-v` (Verbose):** Muestra la URL completa de cada hallazgo (muy útil para hacer clic directamente).
* **`-o` (Output):** Guarda los resultados en un archivo (ej. `-o resultados.json`).

### 3.2 Banderas de Petición (Manipulación HTTP)
* **`-H` (Header):** Añade cabeceras personalizadas. Vital para VHosts o tokens (ej. `-H "Authorization: Bearer <token>"` o `-H "Host: FUZZ.target.htb"`).
* **`-X` (Method):** Cambia el método HTTP, que por defecto es GET (ej. `-X POST` o `-X PUT`).
* **`-d` (Data):** Envía datos en el cuerpo de la petición (ej. `-d "user=admin&pass=FUZZ"`).

### 3.3 Banderas de Filtrado (Limpieza de Basura)
La regla de oro: Las que empiezan con **`m` (Match)** muestran *solamente* eso. Las que empiezan con **`f` (Filter)** *ocultan* eso.

**Por Código HTTP (Code):**
* **`-mc`:** Muestra SOLAMENTE estos códigos (ej. `-mc 200,301`).
* **`-fc`:** OCULTA estos códigos (ej. `-fc 403,404`).

**Por Tamaño y Contenido (Size, Words, Lines):**
*Si el servidor devuelve siempre código 200 OK mintiendo, filtra los falsos positivos por la medida de la página de error.*
* **`-fs` / `-ms` (Size):** Filtra/Muestra por tamaño exacto en bytes (ej. `-fs 1042`).
* **`-fw` / `-mw` (Words):** Filtra/Muestra por cantidad de palabras (ej. `-fw 250`).
* **`-fl` / `-ml` (Lines):** Filtra/Muestra por cantidad de líneas de código (ej. `-fl 35`).

### 3.4 👑 Comando Maestro FFUF (Ejemplo de Combate)
Este es el comando estandarizado ideal para lanzar contra una web nueva. Utiliza color, busca extensiones clave, ignora los errores 404/403 y guarda un registro ordenado en pantalla:

`ffuf -c -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -u http://10.10.10.X/FUZZ -e .php,.txt,.bak -fc 404,403 -v`

---

## 4. Fuzzing de Parámetros (GET)
Se utiliza cuando encuentras una página válida (ej. `index.php`) pero necesitas descubrir si acepta variables ocultas para inyectar comandos (ej. `?page=`, `?id=`, `?cmd=`). 

**Caso A: Descubrir el NOMBRE del parámetro vulnerable (Ej. buscando LFI)**
Colocamos la palabra `FUZZ` donde iría el nombre de la variable y le asignamos un valor malicioso de prueba. 

* **Comando:** `ffuf -c -w /usr/share/seclists/Fuzzing/LFI/LFI-Jhaddix.txt -u "http://10.10.10.X/index.php?FUZZ=../../../../etc/passwd" -fs [TAMAÑO_BASE]` 

**Caso B: Descubrir el VALOR de un parámetro conocido** 
Si ya sabes que el parámetro es `?page=`, pero no sabes qué archivos existen, colocas el `FUZZ` en el valor. 

* **Comando:** `ffuf -c -w /usr/share/seclists/Fuzzing/LFI/LFI-gracefulsecurity-linux.txt -u "http://10.10.10.X/index.php?page=FUZZ" -fs [TAMAÑO_ERROR]` 

*(Nota: Al hacer fuzzing de parámetros en URLs complejas, es buena práctica encerrar la URL entre comillas dobles `""` en la terminal).*

## 5. Bonus: Wfuzz (La Navaja Suiza del Fuzzing)

Alternativa clásica a FFUF. Es ligeramente más lento, pero extremadamente versátil para manipular diccionarios al vuelo y atacar múltiples variables a la vez.

### FFUF vs Wfuzz (Comparativa Rápida)

| Característica | FFUF | Wfuzz |
| :--- | :--- | :--- |
| **Tecnología** | Golang (Ultrarrápido) | Python (Versátil) |
| **Palabra Clave** | `FUZZ` | `FUZZ`, `FUZ2Z`, `FUZ3Z` |
| **Uso Ideal** | Directorios, VHosts, fuerza bruta masiva | Parámetros complejos, APIs, codificación en vivo |

### Comandos Esenciales de Wfuzz

La lógica es similar, pero en Wfuzz el diccionario se indica con `-z file,ruta` y la URL siempre va al final.

* **Escaneo Básico (Directorios):**
  `wfuzz -c -z file,/usr/share/wordlists/dirb/common.txt http://10.10.10.X/FUZZ`
* **Inyección Múltiple (Atacar dos variables a la vez):**
  *(Ej. adivinar nombre de usuario y directorio al mismo tiempo).*
  `wfuzz -c -z file,usuarios.txt -z file,directorios.txt http://10.10.10.X/FUZZ/FUZ2Z`
* **Fuzzing de Parámetros (POST):**
  `wfuzz -c -z file,parametros.txt -d "FUZZ=test" http://10.10.10.X/api.php`
* **Magia de Encodings (Codificación al vuelo):**
  *(Convierte automáticamente las palabras del diccionario a base64 al atacar).*
  `wfuzz -c -z file,diccionario.txt,base64 http://10.10.10.X/index.php?page=FUZZ`

### Filtrado de Basura en Wfuzz
En lugar de `-f` (Filter) como en FFUF, Wfuzz utiliza `--h` (Hide) o `--s` (Show).

* **Ocultar por Códigos HTTP (`--hc`):**
  `... --hc 404,403`
* **Ocultar por cantidad de Palabras (`--hw`):**
  `... --hw 250`
* **Ocultar por Líneas de código (`--hl`):**
  `... --hl 35`