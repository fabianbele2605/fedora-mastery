# Módulo 04 — Administración desde Cockpit

- Estado: En curso
- Fecha: 2026-09-25
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: comparar cada panel de Cockpit
  (cuentas, registros, recursos, actualizaciones) contra su comando de
  terminal equivalente, para no aprender Cockpit como caja negra.

## Concepto diferencial

Cockpit no reemplaza los mecanismos del sistema, los expone visualmente:

- **Cuentas**: sin almacén propio — lee/escribe `/etc/passwd`/`/etc/shadow`
  directamente. Crear un usuario desde la UI es el mismo efecto que
  `useradd`/`passwd` por CLI.
- **Registros**: interfaz visual sobre `journalctl` — cada filtro de la UI
  (servicio, prioridad, rango de tiempo) tiene un flag equivalente.
- **Recursos** (histórico de CPU/memoria/red): depende de **`cockpit-pcp`**
  (Performance Co-Pilot). Sin ese paquete, Cockpit solo muestra el
  snapshot instantáneo — antes de asumir un bug, hay que verificar si el
  paquete está instalado.
- **Actualizaciones**: capa visual sobre `dnf check-update`/`dnf upgrade`.
  El aviso "Actualizaciones de seguridad disponibles" que vimos desde el
  módulo 01 tiene que poder cruzarse con la salida real de `dnf`.

## Práctica guiada

```bash
getent passwd fbeleno
sudo tail -5 /etc/passwd
rpm -q cockpit-pcp
sudo dnf check-update
journalctl --since "-15 min" -p err
```

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
