# Módulo 19 — Break & Fix SELinux (cierre de Fase 3)

- Estado: En curso
- Fecha: 2026-09-26
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: tercer Break & Fix formal —
  recuperar un servicio con contexto SELinux incorrecto **sin
  desactivar `enforcing`** en ningún momento.

## Entorno y punto de restauración

Snapshot `18-fin-fase3-antes-breakfix` tomado antes de este módulo (fin
de Fase 3: SELinux enforcing, contextos, AVC/sealert, servicio web en
puerto 8585 con SELinux+firewalld, audit2allow analizado con criterio).

## Cambio deliberado

Mal-etiquetar el contenido real de `httpd` (módulo 17):

```bash
sudo chcon -t default_t /var/www/html/index.html
ls -Z /var/www/html/index.html
```

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
