# Módulo 11 — COPR: búsqueda, revisión y evaluación de riesgos

- Estado: En curso
- Fecha: 2026-09-25
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: COPR no es un AUR equivalente —
  distinto modelo de confianza y de construcción de paquetes.

## Concepto diferencial

En AUR, el usuario compila el paquete localmente desde un `PKGBUILD`:
confía en la receta, pero el binario lo genera su propia máquina. En
**COPR**, el binario ya viene precompilado por la infraestructura de
build de Fedora (Copr Build System, usando `mock`) — el modelo de
confianza se parece más a agregar un PPA de Ubuntu que al AUR: hay que
confiar en el mantenedor del proyecto **y** en que Fedora construyó ese
binario correctamente, sin compilarlo uno mismo.

Antes de habilitar un repo COPR conviene revisar: quién es el
mantenedor, cuántos proyectos tiene, si el `.spec` es público, y si el
paquete hace lo que dice.

## Práctica guiada

```bash
sudo dnf copr search eza
# revisar el mantenedor/proyecto antes de habilitar
sudo dnf copr enable -y <maintainer>/<project>
cat /etc/yum.repos.d/_copr:copr.fedorainfracloud.org:*.repo
sudo dnf install eza
rpm -qi eza
```

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
