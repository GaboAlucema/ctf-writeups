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

## 3. Filtrado Avanzado en FFUF (Limpieza de Basura)

Cuando el servidor miente y devuelve cientos de "falsos positivos" (ej. páginas de error personalizadas con código 200), usamos estas banderas para limpiar la terminal.

**Filtrar por Código HTTP:**
* **`-mc` (Match Code):** Muestra SOLAMENTE estos códigos.
  `... -mc 200,301`
* **`-fc` (Filter Code):** OCULTA estos códigos (Útil si te saturan los 403).
  `... -fc 403,302`

**Filtrar por Tamaño y Contenido (Para falsos positivos 200 OK):**
Si toda la basura que sale tiene, por ejemplo, un tamaño de 1042 bytes, lo filtramos así:
* **`-fs` (Filter Size):** Oculta por tamaño específico en bytes.
  `... -fs 1042`
* **`-fw` (Filter Words):** Oculta por cantidad de palabras.
  `... -fw 250`
* **`-fl` (Filter Lines):** Oculta por cantidad de líneas de código.
  `... -fl 35`