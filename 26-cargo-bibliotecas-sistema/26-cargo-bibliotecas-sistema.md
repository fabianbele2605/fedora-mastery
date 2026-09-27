# Módulo 26 — Cargo y bibliotecas del sistema

- Estado: En curso
- Fecha: 2026-09-27
- Versión de Fedora: Fedora Linux 45 (Server Edition)
- Objetivo diferencial frente a Arch: en Arch, una librería nativa
  necesaria para compilar un crate Rust suele resolverse con un único
  paquete de pacman. Fedora separa explícitamente el paquete de
  *runtime* (`openssl-libs`) del paquete `-devel` (`openssl-devel`,
  con los headers `.h` y los archivos `.pc` de `pkg-config`) — sin el
  `-devel`, Cargo descarga y compila el crate sin problema, pero el
  enlazador falla al no encontrar la librería C real.

## Concepto diferencial

`pkg-config` es el mecanismo estándar que usan los crates Rust con
componente nativo (bindings a C) para localizar headers y flags de
compilación de una librería del sistema. Fedora empaqueta ese archivo
`.pc` únicamente dentro del paquete `-devel`, nunca en el paquete de
runtime — una separación deliberada para no forzar el peso de
compilación en instalaciones que solo necesitan ejecutar el software,
no compilarlo.

## Práctica guiada

Crear un proyecto Cargo que dependa de un crate con componente nativo
(`openssl`, el ejemplo más común y pedagógico) y provocar el fallo de
enlazado real antes de instalar el `-devel`:

```bash
cargo new sonda-openssl
cd sonda-openssl
cargo add openssl
cargo build
```

Tras confirmar el fallo real, instalar la dependencia correcta y
recompilar:

```bash
sudo dnf install -y openssl-devel
cargo build
pkg-config --libs --cflags openssl
```

## Hallazgos reales

_Pendiente — se completa durante la práctica guiada._

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
