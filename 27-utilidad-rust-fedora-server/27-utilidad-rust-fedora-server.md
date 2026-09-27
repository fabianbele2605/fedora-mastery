# Módulo 27 — Utilidad Rust para Fedora Server

- Estado: En curso
- Fecha: 2026-09-27
- Versión de Fedora: Fedora Linux 45 (Server Edition)
- Objetivo diferencial frente a Arch: escribir una CLI real en Rust
  que consolide señales de diagnóstico que venimos revisando a mano
  durante todo el curso (versión de Fedora, SELinux, disco, memoria) —
  la pieza que se empaqueta como `.rpm` propio en el módulo 28.

## Concepto diferencial

Una utilidad de diagnóstico de sistema en Rust casi nunca reimplementa
en Rust puro lo que ya hacen herramientas maduras del sistema
(`df`, `free`, `getenforce`) — las envuelve vía `std::process::Command`
y consolida su salida. A diferencia del módulo 26 (enlazado contra una
librería C real, `libcrypto`), aquí no hay dependencia nativa que
declarar: menos superficie para el `.spec` del módulo 28.

**Decisión de toolchain** (heredada del módulo 25): desarrollo
interactivo con `rustup`; el build final que se empaquete en el
módulo 28 se compilará con el `cargo` de DNF.

## Práctica guiada

```bash
cargo new fedser-diag
cd fedser-diag
```

Reemplazar `src/main.rs` con una versión inicial que ejecute
`cat /etc/fedora-release`, `getenforce`, `df -h /`, `free -h` vía
`std::process::Command`, capture su salida y la imprima en un
reporte consolidado.

```bash
cargo build
./target/debug/fedser-diag
```

## Hallazgos reales

_Pendiente — se completa durante la práctica guiada._

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
