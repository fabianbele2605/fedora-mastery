# Módulo 32 — Rollback de despliegues

- Estado: En curso
- Fecha: 2026-09-28
- Versión de Fedora: Fedora Silverblue 44.1.7
- Objetivo diferencial frente a Arch: Arch no tiene un mecanismo de
  rollback a nivel de sistema operativo — revertir un cambio requiere
  reinstalar paquetes uno por uno o restaurar un snapshot externo
  (Btrfs/Timeshift, si el usuario lo configuró aparte). `rpm-ostree`
  trae el rollback como mecanismo nativo del sistema.

## Punto de partida

Desde el módulo 31 la VM ya tiene dos despliegues reales:
- El despliegue activo actual, con `htop` en capas (`LayeredPackages`).
- El despliegue original, sin capas, conservado intacto.

## Concepto diferencial

`rpm-ostree rollback` no reinstala nada ni reconstruye un árbol —
simplemente reordena cuál de los despliegues **ya presentes en el
disco** arranca por defecto en el próximo reinicio. Es instantáneo
porque ambos árboles de commits ya existen; solo cambia un puntero,
muy distinto de:
- Deshacer una transacción de DNF (reinstala paquete por paquete).
- Restaurar un snapshot de VirtualBox (revierte el disco completo,
  incluidos archivos de usuario, no solo el sistema base).

## Práctica guiada

```bash
rpm-ostree status
rpm-ostree rollback
rpm-ostree status
systemctl reboot
```

Tras reiniciar, confirmar que `htop` ya no está disponible:

```bash
which htop
rpm-ostree status
```

Y cerrar el ciclo volviendo al despliegue con `htop` (segundo rollback):

```bash
rpm-ostree rollback
systemctl reboot
```

## Hallazgos reales

_Pendiente — se completa durante la práctica guiada._

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
