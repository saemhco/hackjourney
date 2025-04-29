# Manual de Pentesting - JuiceShop

## 📜 Información General

- **Objetivo:** Realizar un pentest sobre la máquina _JuiceShop_ (IP: `10.10.30.134`).
    
- **Puerto expuesto:** `3000/tcp` (HTTP / aplicación web).
    
- **Tecnología detectada:** OWASP Juice Shop (aplicación vulnerable para prácticas).
    

---

## 🛡️ Fases del Pentesting

### 1. Reconocimiento

#### 🔍 Escaneo de Puertos

```
sudo nmap 10.10.30.134 -n -Pn -sS -sC -sV --min-rate 5000 -oN juice_scan.txt
```

- Puerto TCP 3000 abierto.
    
- Servicio responde con características HTTP (`X-Recruiting: /#/jobs`, "OWASP Juice Shop" en HTML).
    

#### 📂 Fuzzing (Enumeración de Directorios)

```
ffuf -u http://10.10.30.134:3000/FUZZ -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 40
```

- Se encontraron rutas importantes:
    
    - `/search`, `/profile`, `/basket`, `/chatbot`, `/login`, `/register`, `/products`, etc.
        

---

### 2. Análisis de Vulnerabilidades

#### 🛠️ Pruebas iniciales en la web

- Acceso a la página principal con productos de jugos.
    
- Activación automática de vulnerabilidades como:
    
    - **Error Handling** (Errores no controlados).
        
    - **Exposed Metrics** (EndPoints de métricas expuestas).
        

#### 📋 Enumeración Manual

- Carga de pruebas en formularios.
    
- Detección de inyección SQL en el login.
    

---

### 3. Explotación

#### 🔑 SQL Injection (Bypass de Login)

- Usuario:
    

```
' OR 1=1--
```

- Contraseña:
    

```
Cualquier valor
```

- Resultado: **Acceso exitoso** como `admin@juice-sh.op`.
    

---

#### 👤 Control de Cuentas

- Se accedió al perfil de usuario:
    
    - Cambio de nombre de usuario.
        
    - Opciones de carga de archivos e imagenes.
        
- Se accedió a "Cesta de Compras":
    
    - Edición de cantidades.
        
- Se accedió a "Chatbot":
    
    - Inyección de mensajes personalizados.
        

---

### 4. Post-Explotación

#### 🔥 Comportamientos detectados

- Cambios de nombre de usuario.
    
- Subida de archivos (posible vector de ataque).
    
- Manipulación de la cesta de compras.
    
- Comunicación con el bot.
    

---

## 📌 Observaciones

- Durante el fuzzing (`ffuf`) se dispararon alertas automáticas en la aplicación.
    
- En Juice Shop, el escaneo puede ser detectado y el progreso "reparado" o "reseteado".
    
- Se recomienda pausar los escaneos tras descubrir los endpoints clave.
    

---

# ✅ Conclusión del Trabajo

Se logró:

- Acceso no autorizado mediante SQLi.
    
- Manipulación de datos de usuario.
    
- Enumeración de rutas y directorios internos.
    
- Identificación de vulnerabilidades de mala gestión de errores y exposición de métricas.
    

---

# 📌 Recomendaciones Siguientes

- Intentar File Upload Bypass.
    
- Revisar seguridad de JWT si se encuentra uso.
    
- Buscar IDOR (Insecure Direct Object References).
    
- Explorar posibles Command Injections en formularios de carga.
    

---

_(Nota: Las capturas de pantalla están disponibles y documentadas aparte si se requiere evidencia visual.)_