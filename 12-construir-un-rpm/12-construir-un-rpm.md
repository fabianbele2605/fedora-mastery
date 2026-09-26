# Módulo 12 — Construcción de un RPM: SPEC, dependencias y validación

- Estado: En curso
- Fecha: 2026-09-26
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: construir un RPM propio desde un
  archivo `.spec`, comparándolo con el `PKGBUILD` ya usado en
  `arch-linux-desktop` (módulo 26).

## Concepto diferencial

`rpmbuild` exige una estructura de directorios fija
(`~/rpmbuild/{SPECS,SOURCES,BUILD,RPMS,SRPMS}`), generada con
`rpmdev-setuptree` — a diferencia de `makepkg`, que solo necesita un
`PKGBUILD` en cualquier carpeta.

El archivo `.spec` tiene secciones obligatorias con nombres fijos
(`%prep`, `%build`, `%install`, `%files`, `%changelog`) — más rígido y
verboso que las funciones `build()`/`package()` de un PKGBUILD, pero más
explícito sobre qué pasa en cada etapa de la construcción.

## Práctica guiada

```bash
sudo dnf install rpm-build rpmdevtools
rpmdev-setuptree
ls ~/rpmbuild
```

Luego se escribe un `.spec` simple (paquete `fedora-mastery-diag`, un
script de diagnóstico propio sin fuente externa), se construye con
`rpmbuild -bb`, se valida el `.rpm` resultante y se instala.

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
