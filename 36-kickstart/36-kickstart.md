# Módulo 36 — Kickstart

- Estado: En curso
- Fecha: 2026-09-29
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial: arranca la Fase 7. Automatiza el mismo
  instalador Anaconda usado a mano en el módulo 01 (y en el 31 para
  Silverblue) — distinto de Ignition (módulo 33), que aplica
  configuración *después* de escribir una imagen ya compuesta.

## Concepto diferencial

Kickstart es un archivo de respuestas que Anaconda consume **durante**
la instalación interactiva: particionado, red, usuario, paquetes —
todo lo que se hizo a mano en el módulo 01, ahora declarado en texto
plano. A diferencia del método HTTP usado (con problemas) en el
módulo 33, acá se usa el mecanismo más robusto: un disco virtual
secundario con la etiqueta `OEMDRV`, que Anaconda detecta y carga
automáticamente sin red ni parámetros de arranque.

## Práctica guiada

1. Escribir `ks.cfg` con lo mínimo real: idioma, teclado, zona
   horaria, particionado automático, usuario, red, y una selección de
   paquetes.
2. Empaquetarlo en una imagen ISO con etiqueta de volumen `OEMDRV`:
   ```bash
   genisoimage -o oemdrv.iso -V OEMDRV -J -R ks.cfg
   ```
3. Crear una VM nueva (`Fedora_Kickstart`), con **dos** unidades
   ópticas: la ISO de Fedora Server (arranque) y `oemdrv.iso`
   (segunda unidad, el Kickstart).
4. Arrancar y observar: Anaconda debería avanzar solo, sin pantallas
   interactivas.
5. Validar el sistema resultante contra lo declarado en el `.ks`
   (hostname, usuario, zona horaria, paquetes).

## Hallazgos reales

_Pendiente — se completa durante la práctica guiada._

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
