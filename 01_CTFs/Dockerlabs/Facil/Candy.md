# 🎯 Máquina: Candy
**IP:** 
**OS:** 
**Dificultad:** 

---
## 1. 🔍 Enumeración
```bash
nmap -p- -sC -sV 172.17.0.2
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.58 ((Ubuntu))
|_http-title: Home
|_http-server-header: Apache/2.4.58 (Ubuntu)
| http-robots.txt: 17 disallowed entries (15 shown)
| /joomla/administrator/ /administrator/ /api/ /bin/
| /cache/ /cli/ /components/ /includes/ /installation/
|_/language/ /layouts/ /un_caramelo /libraries/ /logs/ /modules/
|_http-generator: Joomla! - Open Source Content Management
MAC Address: 5A:9E:4B:9B:20:1C (Unknown)
```
Puerto 80: Servidor web Apache/2.4.58 (Ubuntu) corriendo el CMS Joomla. El archivo robots.txt revela el panel de login en /administrator/ y una ruta anómala interesante: `/un_caramelo`
Al revisar la página encontramos el siguiente html:
```html
<html lang="es"><head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Página en Construcción</title>
    <style>
        body {
            background-color: #f8f9fa;
            font-family: Arial, sans-serif;
            color: #343a40;
            display: flex;
            align-items: center;
            justify-content: center;
            height: 100vh;
            margin: 0;
        }

        .container {
            text-align: center;
        }

        h1 {
            font-size: 3rem;
            margin-bottom: 1rem;
            color: #dc3545;
        }

        p {
            font-size: 1.25rem;
            margin-bottom: 2rem;
        }

        .construction-icon {
            font-size: 6rem;
            color: #ffc107;
            margin-bottom: 1rem;
        }

        a {
            display: inline-block;
            margin-top: 1rem;
            padding: 0.75rem 1.5rem;
            font-size: 1rem;
            color: #fff;
            background-color: #007bff;
            text-decoration: none;
            border-radius: 0.25rem;
            transition: background-color 0.3s ease;
        }

        a:hover {
            background-color: #0056b3;
        }
    </style>
</head>
<body>
    <div class="container">
        <div class="construction-icon">🚧</div>
        <h1>Página en Construcción</h1>
	<!-- Creds
	admin:c2FubHVpczEyMzQ1
	-->
        <p>Estamos trabajando en algo increíble. ¡Vuelve pronto!</p>
        <a href="#">Volver al Inicio</a>
    </div>


</body></html>
```
Encontramos las credenciales de **admin** comentadas. Al revisar la página principal, vemos un portal de login. Probamos las credenciales pero no son correctas. Nos dimos cuenta que el link puede tener LFI:
```url
http://172.17.0.2/index.php/component/users/login?Itemid=101
```
Pero luego de estar probando distintas confi
## 2. 🔓 Explotación (Foothold)
*(¿Cómo logramos entrar? ¿Qué vulnerabilidad o script usamos?)*


## 3. 🚀 Escalada de Privilegios
*(¿Cómo pasamos de ser un usuario normal a ser Administrador o Root?)*


## 4. 🚩 Banderas (Flags)
- **User:** 
- **Root**: 