# Módulo 29 — Break & Fix de compilación

- Estado: En curso
- Fecha: 2026-09-27
- Versión de Fedora: Fedora Linux 45 (Server Edition)
- Objetivo diferencial frente a Arch: cierra la Fase 5 y el Proyecto 4
  con un incidente de **metadatos RPM incorrectos** — un `%files`
  desalineado con lo que `%install` realmente coloca en el
  `BUILDROOT`, distinto de los incidentes de enlazado nativo (módulo
  26) y toolchain equivocado (módulo 28) ya cubiertos en esta fase.

## Entorno y punto de restauración

Snapshot tomada antes de este módulo: `fedser-diag.spec` funcional del
módulo 28 (compilando con el `cargo` de DNF), sin el paquete instalado
en el sistema.

## Concepto diferencial

Un `.spec` puede tener un `%build`/`%install` perfectamente correctos
y aun así fallar — porque `%files` es la lista de lo que el paquete
final debe contener, y `rpmbuild` la valida contra lo que realmente
existe en el `BUILDROOT` tras `%install`. Un typo ahí no rompe la
compilación de Rust: rompe el empaquetado, con un mensaje de error
propio de `rpmbuild`, no de `cargo`.

## Incidente simulado

Se introduce un typo real en la línea `%files` de
`~/rpmbuild/SPECS/fedser-diag.spec`: `%{_bindir}/fdser-diag` en vez de
`%{_bindir}/fedser-diag` (falta la primera "e").

## Práctica guiada (diagnóstico real)

```bash
rpmbuild -bb ~/rpmbuild/SPECS/fedser-diag.spec
```

Leer el mensaje de error de `rpmbuild` con atención — debería señalar
explícitamente qué archivo esperado no encontró.

## Reparación

Corregir el typo en `%files`, reconstruir, y confirmar que el `.rpm`
se genera con éxito e instala igual que en el módulo 28.

## Hallazgos reales

_Pendiente — se completa durante la práctica guiada._

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
