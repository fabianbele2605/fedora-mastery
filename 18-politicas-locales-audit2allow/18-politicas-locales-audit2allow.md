# Módulo 18 — Políticas locales y `audit2allow`

- Estado: En curso
- Fecha: 2026-09-26
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: interpretar críticamente lo que
  `audit2allow` propone, en vez de aceptarlo como solución automática.

## Concepto diferencial

`audit2allow` genera un módulo de política que permite exactamente lo
que fue denegado — no juzga si eso es una buena decisión de seguridad.
La regla de oro (ya citada por `sealert` en el módulo 16): primero
revisar si el problema se resuelve con `semanage fcontext`, booleanos, o
`semanage port` — todos mecanismos ya cubiertos por la política
oficial. `audit2allow` es el último recurso, para casos genuinamente sin
cobertura, y cada regla que genera debe justificarse antes de instalarla
con `semodule -i`.

## Práctica guiada

Generar (sin instalar) el módulo para la denegación real del módulo 16:

```bash
sudo ausearch -c sshd --raw | audit2allow -M mymodule-sshd
cat mymodule-sshd.te
```

Analizar críticamente la regla generada, y decidir si instalarla o
rechazarla con justificación. Repetir el análisis con la denegación de
`loadkeys`/`dac_override` vista desde el módulo 16.

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
