# Módulo 14 — Política SELinux en Fedora

- Estado: En curso
- Fecha: 2026-09-26
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: SELinux agrega una capa de **MAC**
  (Mandatory Access Control) sobre el **DAC** tradicional de Unix
  (dueño/grupo/permisos) que es lo único que existe en Arch. Arranca la
  Fase 3 del curso.

## Concepto diferencial

En Arch, el control de acceso es puramente **DAC**: si tenés permisos de
lectura/escritura sobre un archivo (por dueño, grupo, u "otros"), podés
acceder. Fin de la historia.

Fedora agrega **SELinux**, una capa de **MAC** que se evalúa además del
DAC, no en su lugar. Cada **proceso** corre en un **dominio** (visible
con `ps -Z`), cada **archivo/objeto** tiene un **tipo** (visible con
`ls -Z`), y la política decide qué dominios pueden acceder a qué tipos.
Si el dominio de un proceso no tiene permiso explícito sobre el tipo de
un archivo, SELinux bloquea el acceso **aunque los permisos Unix lo
permitan** — el DAC decir que sí no alcanza, hace falta que el MAC
también diga que sí.

Fedora usa la política **`targeted`** por defecto (no `mls` ni
`minimum`): solo confina procesos específicos considerados de riesgo
(servicios de red, daemons expuestos), dejando al usuario normal
("unconfined") sin restricciones adicionales de SELinux.

## Práctica guiada

```bash
getenforce
sestatus
ps -eZ | head -20
ls -Z /etc/ssh
id -Z
```

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
