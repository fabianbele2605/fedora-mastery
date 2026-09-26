# Módulo 20 — Servidor web desde terminal y Cockpit

- Estado: En curso
- Fecha: 2026-09-26
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: administrar un servicio real
  (`httpd`, ya corriendo desde el módulo 17) desde Cockpit, comparando
  cada acción contra su equivalente en terminal. Arranca la Fase 4.

## Concepto diferencial

Arch no tiene una capa de administración gráfica nativa — todo pasa
por `systemctl`/`journalctl` en terminal. Cockpit expone los mismos
mecanismos de systemd (unidades, estado, logs) en una interfaz web,
sin agregar una capa de gestión propia: cada botón de Cockpit dispara
el comando equivalente por detrás, verificable siempre desde la CLI.

## Práctica guiada

```bash
systemctl is-enabled httpd
systemctl is-active httpd
journalctl -u httpd --no-pager | tail -10
```

Luego, desde Cockpit → "Servicios": buscar `httpd`, revisar su estado,
habilitarlo para el arranque (`enable`) desde la interfaz, y comparar
los logs mostrados ahí contra `journalctl -u httpd`. Confirmar el
cambio de habilitación con `systemctl is-enabled httpd` por CLI.

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
