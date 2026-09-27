# Módulo 31 — Fedora Silverblue

- Estado: En curso
- Fecha: 2026-09-27
- Versión de Fedora: Fedora Silverblue (última estable)
- Objetivo diferencial frente a Arch: primera VM del curso con un
  sistema base realmente inmutable — Arch nunca tuvo equivalente.
  Confirma en la práctica el contraste conceptual del módulo 30
  (DNF transaccional vs. despliegues por imagen) sobre un sistema
  Silverblue real, no simulado con el CLI aislado.

## Entorno

VM temporal nueva (no la Fedora Server usada desde el módulo 00):
4 GB de RAM según la tabla de entorno del curso, Guest Additions
limitadas (es esperado en un sistema inmutable — no se puede instalar
software arbitrario sobre `/usr`).

## Concepto diferencial

Silverblue está orientado a **escritorio atómico**: el sistema base
(`/usr`) es de solo lectura, gestionado por `rpm-ostree` como una
serie de despliegues versionados. Para instalar aplicaciones de
escritorio se usa **Flatpak** (sandboxed, fuera del árbol OSTree). Para
herramientas de desarrollo o CLI que sí necesitan tocar paquetes
tradicionales, se usa **Toolbox**: un contenedor con un sistema Fedora
tradicional completo (RPM/DNF normal) dentro, aislado del host.
**Layering** (`rpm-ostree install`) es la excepción — agrega un
paquete al árbol base, pero requiere reiniciar para aplicarse (no es
una instalación en caliente como con DNF).

## Práctica guiada

1. Descargar la ISO de Fedora Silverblue.
2. Crear la VM (4 GB RAM, similar al proceso del módulo 00/01).
3. Instalar Silverblue.
4. Explorar el despliegue real:
   ```bash
   rpm-ostree status
   ```
5. Layering de un paquete adicional:
   ```bash
   sudo rpm-ostree install <paquete>
   ```
   (requiere reiniciar para activarse — a diferencia de DNF)
6. Confirmar Toolbox:
   ```bash
   toolbox create
   toolbox enter
   ```
7. Confirmar Flatpak como vía de aplicaciones de escritorio.

## Hallazgos reales

_Pendiente — se completa durante la práctica guiada._

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
