# Módulo 09 — DNF5 y transacciones

- Estado: En curso
- Fecha: 2026-09-25
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: DNF5 es el equivalente completo a
  `pacman` (resuelve dependencias, descarga, gestiona repos) — a
  diferencia de `rpm` (módulo 08), que es solo la capa de bajo nivel.

## Concepto diferencial

**DNF5** es una reescritura completa en C++ (el `dnf` clásico era
Python) — Fedora lo adoptó como gestor de paquetes por defecto desde
Fedora 41.

La diferencia real más marcada frente a pacman: **DNF mantiene un
historial de transacciones con undo/rollback real**
(`dnf history undo <id>`, `dnf history rollback <id>`). pacman no tiene
esto nativo — en Arch, deshacer una transacción requiere snapshots de
Btrfs/Snapper (ya visto en `arch-linux-desktop`, módulos 21-24) o
revertir manualmente paquete por paquete con `pacman -U` sobre el caché.

## Práctica guiada

```bash
sudo dnf5 history list
sudo dnf install tree
sudo dnf5 history list
sudo dnf5 history info <id-de-la-transaccion-nueva>
sudo dnf5 history undo <id-de-la-transaccion-nueva>
rpm -q tree
```

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
