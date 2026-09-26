# Módulo 13 — Break & Fix de paquetes (cierre de Fase 2)

- Estado: En curso
- Fecha: 2026-09-26
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: segundo Break & Fix formal del
  curso — romper un repositorio real (typo en la URL de metadatos) y
  diagnosticar el error de DNF hasta la causa raíz, reparando sin
  restaurar snapshot.

## Entorno y punto de restauración

Snapshot `12-fin-fase2-antes-breakfix` tomado antes de este módulo (fin
de Fase 2: RPM/pacman, DNF5 history/undo, GPG/repos, COPR evaluado sin
habilitar, primer RPM propio construido).

## Cambio deliberado

Se hace una copia de seguridad del archivo `.repo` real de `updates`, y
luego se introduce un typo real en su `metalink=` (URL de metadatos),
simulando un error de configuración manual.

## Síntoma

_(se completa con la práctica real)_

## Diagnóstico y recuperación

_(se completa con la práctica real: observación, registros, hipótesis,
prueba, causa raíz, solución, validación)_

## Impacto y riesgos

_(pendiente)_

## Cómo evitar recurrencia

_(pendiente)_

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
