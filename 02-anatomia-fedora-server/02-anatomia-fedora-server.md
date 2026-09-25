# Módulo 02 — Anatomía de Fedora Server

- Estado: En curso
- Fecha:
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: entender qué decisiones de empaquetado
  y repositorios son propias de la **edición** Fedora Server, antes de
  meterse con RPM/DNF a fondo (Fase 2).

## Concepto diferencial

Fedora no es una sola distro con "perfiles" opcionales como podría pensarse
viniendo de Arch (donde instalás lo que querés, punto). Fedora publica
**ediciones** completas y separadas (Workstation, Server, IoT, CoreOS,
Silverblue...), cada una con:

- Su propio ISO y flujo de instalación (Anaconda con un rol de instalación
  distinto, `server-product-environment` en este caso).
- Su propio conjunto de paquetes base y servicios habilitados por defecto
  (ya vimos que Cockpit viene activo en Server, y no lo estaría en una
  instalación mínima genérica).
- Repos separados por función: `fedora` (paquetes de la release, fijos),
  `updates` (parches posteriores al lanzamiento) y `updates-testing`
  (candidatos a `updates`, deshabilitado por defecto). Arch en cambio
  combina todo en `core`+`extra`, sin distinguir "base de la release" de
  "actualizaciones" como conceptos separados — cada paquete simplemente
  tiene la versión más nueva disponible, sin ISOs de rastreo por versión.

También existe el concepto de **grupo de instalación** (`dnf group list
--installed`), un conjunto de paquetes con nombre propio que la base de
datos de RPM/DNF recuerda como unidad (ej. `Server Product Core`). En
Arch, algo como `base` es solo un metapaquete con dependencias normales —
no hay un objeto "grupo" con estado propio en la base de datos de pacman.

## Práctica guiada

```bash
cat /etc/os-release
dnf repolist --all
dnf group list --installed
rpm -qa | wc -l
df -hT /
```

_(se completa con el resto de la práctica real a medida que se ejecuta)_

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes

