# Módulo 30 — OSTree y rpm-ostree

- Estado: En curso
- Fecha: 2026-09-27
- Versión de Fedora: Fedora Linux 45 (Server Edition)
- Objetivo diferencial frente a Arch: arranca la Fase 6. Arch no tiene
  equivalente a los despliegues basados en imágenes — es rolling
  release sobre un sistema de archivos mutable, igual que el DNF
  tradicional ya visto. Este módulo contrasta ese modelo transaccional
  (vivido en los módulos 08-13, 23-24) contra el modelo de imágenes de
  `rpm-ostree`, antes de crear la VM de Silverblue en el módulo 31.

## Concepto diferencial

- **DNF** (todo el curso hasta ahora): cada transacción modifica el
  sistema de archivos in situ, paquete por paquete. Un rollback real
  requiere reinstalar versiones anteriores una por una (o restaurar un
  snapshot externo, como en los Break & Fix de este curso).
- **rpm-ostree**: el sistema raíz completo es un árbol de commits
  (análogo a Git). Cada actualización crea un **despliegue** nuevo e
  inmutable; el rollback es instantáneo — se reinicia sobre el
  despliegue anterior, no se revierte paquete por paquete.

## Práctica guiada

Instalar el CLI de `rpm-ostree` en esta misma Fedora Server (paquete
DNF normal, disponible fuera de Silverblue/CoreOS) y confirmar su
límite real: sin una raíz basada en OSTree, no hay despliegues que
gestionar.

```bash
sudo dnf install -y rpm-ostree
rpm-ostree --version
rpm-ostree status
```

## Hallazgos reales

_Pendiente — se completa durante la práctica guiada._

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
