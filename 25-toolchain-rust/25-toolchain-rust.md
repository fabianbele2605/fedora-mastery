# Módulo 25 — Toolchain Rust: rustup frente a DNF

- Estado: En curso
- Fecha: 2026-09-27
- Versión de Fedora: Fedora Linux 45 (Server Edition)
- Objetivo diferencial frente a Arch: arranca la Fase 5. Comparar dos
  caminos reales para tener Rust en Fedora — el paquete de sistema
  (ligado al ciclo de release de Fedora) frente a `rustup` (herramienta
  oficial del proyecto Rust, versionado independiente del SO).

## Concepto diferencial

Fedora empaqueta Rust como cualquier otro lenguaje del sistema:
`dnf install rust cargo` instala una versión fija, la misma que trae
el repositorio de esa versión de Fedora, actualizada solo cuando DNF
actualiza el sistema. `rustup`, en cambio, es el instalador oficial del
proyecto Rust: vive en `$HOME/.cargo` y `$HOME/.rustup`, permite tener
varias toolchains instaladas a la vez (stable/beta/nightly), cambiar
de una a otra por proyecto, y actualizar el compilador sin depender de
un `dnf update` del sistema completo.

## Práctica guiada

```bash
# Camino 1: paquete de Fedora
sudo dnf install -y rust cargo
rustc --version
cargo --version
which rustc cargo
```

```bash
# Camino 2: rustup (instalador oficial)
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
# tras instalar, cargar el entorno de la sesión actual
source "$HOME/.cargo/env"
rustc --version
cargo --version
which rustc cargo
rustup show
```

## Decisión de alcance

Comparar ambas versiones (`rustc --version`) y decidir con criterio
cuál usar para el resto de la Fase 5 (módulos 26-29), considerando que
el módulo 28 va a empaquetar un binario como `.rpm` — un factor real a
favor de compilar con el toolchain que después firme mejor con las
convenciones de Fedora Packaging para Rust.

## Hallazgos reales

_Pendiente — se completa durante la práctica guiada._

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
