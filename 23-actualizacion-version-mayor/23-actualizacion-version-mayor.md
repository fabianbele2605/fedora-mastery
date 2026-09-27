# Módulo 23 — Actualizaciones entre versiones de Fedora Server

- Estado: En curso
- Fecha: 2026-09-27
- Versión de Fedora: Fedora Linux 44 → 45 (Server Edition)
- Objetivo diferencial frente a Arch: Fedora tiene versiones mayores
  discretas con ciclo de vida definido — Arch es rolling-release, no
  existe el concepto de "saltar de versión". El mecanismo
  (`dnf system-upgrade`) descarga todo primero y aplica los cambios en
  modo offline tras un reinicio, muy distinto a un `dnf upgrade` normal.

## Entorno y punto de restauración

Snapshot `22-antes-upgrade-mayor-f45` tomado antes de este módulo:
Fedora Server 44 completo, con Cockpit, `httpd` en 8585, contenedor
Podman rootless, SELinux enforcing, virtualización anidada habilitada.

## Concepto diferencial

`dnf system-upgrade` (plugin aparte de `dnf`) no actualiza en caliente:
descarga todos los paquetes de la nueva versión primero
(`system-upgrade download`), y recién al reiniciar entra en un modo
especial fuera de línea donde aplica la transacción completa
(`system-upgrade reboot`). Es una operación de alto riesgo real —
toca el sistema entero, no un solo servicio.

## Práctica guiada

```bash
cat /etc/fedora-release
sudo dnf install -y dnf-plugin-system-upgrade
sudo dnf system-upgrade download --releasever=45 -y
```

Luego, tras confirmar la descarga:

```bash
sudo dnf system-upgrade reboot
```

## Validación post-actualización (plan de reversión)

```bash
cat /etc/fedora-release
systemctl status cockpit.socket httpd podman
curl -s http://localhost:8585
sestatus
```

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
