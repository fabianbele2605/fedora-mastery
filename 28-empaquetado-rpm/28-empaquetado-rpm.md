# Módulo 28 — Empaquetado RPM de `fedser-diag`

- Estado: En curso
- Fecha: 2026-09-27
- Versión de Fedora: Fedora Linux 45 (Server Edition)
- Objetivo diferencial frente a Arch: cerrar el ciclo de la Fase 5
  empaquetando el binario Rust del módulo 27 como `.rpm` real, con el
  mismo flujo `rpmbuild`/SPEC del módulo 12 pero aplicado a un binario
  compilado en vez de un script — y resolviendo en la práctica la
  decisión de toolchain pendiente desde el módulo 25.

## Concepto diferencial

Un SPEC que compile con `rustup` funcionaría en esta VM, pero jamás en
un entorno real de build de Fedora (`mock`/Koji): esos entornos están
aislados y sin red, solo tienen lo que se declare en `BuildRequires` e
instale `dnf`. Por eso el `%build` de este módulo usa explícitamente el
`cargo`/`rustc` de DNF (módulo 25), no el de `rustup`.

## Práctica guiada

Preparar el árbol de fuentes (mismo patrón del módulo 12):

```bash
cd ~
cp -r ~/fedser-diag fedser-diag-0.1.0
tar czf ~/rpmbuild/SOURCES/fedser-diag-0.1.0.tar.gz fedser-diag-0.1.0
```

Escribir `~/rpmbuild/SPECS/fedser-diag.spec` con:
- `BuildRequires: cargo rust`
- `%build`: `cargo build --release` (con el `cargo` de DNF, confirmar
  con `which cargo` antes de construir)
- `%install`: instalar el binario en `%{buildroot}%{_bindir}`
- `%files`: `%{_bindir}/fedser-diag`

Construir, instalar y validar:

```bash
rpmbuild -bb ~/rpmbuild/SPECS/fedser-diag.spec
sudo dnf install -y ~/rpmbuild/RPMS/x86_64/fedser-diag-0.1.0-1*.rpm
which fedser-diag
fedser-diag
```

Desinstalar limpiamente:

```bash
sudo dnf remove -y fedser-diag
which fedser-diag
```

## Hallazgos reales

_Pendiente — se completa durante la práctica guiada._

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
