# Módulo 39 — Fedora Enterprise Lab (Proyecto final)

- Estado: En curso
- Fecha: 2026-09-30
- Versión de Fedora: Fedora Linux 45 (Server Edition)
- Objetivo: cierra el curso completo (Fase 8). Integra en un solo
  entorno coherente las piezas construidas por separado en módulos
  anteriores, en vez de crear algo nuevo desde cero.

## Piezas a integrar (todas ya construidas en el curso)

- Fedora Server administrado por SSH (módulo 06) y Cockpit (módulos
  03-04).
- `fedser-diag`, utilidad propia en Rust empaquetada como `.rpm`
  (módulos 25-29).
- Servicio en Podman gestionado declarativamente por `systemd`, con
  SELinux enforcing (patrón del módulo 34, aplicado aquí a Server).
- Automatización con el playbook de Ansible del módulo 37 (con las
  soluciones reales ya encontradas para DNF5/`seport`).

## Plan de la práctica

1. Snapshot de respaldo `39-antes-proyecto-final` en VirtualBox.
2. Extender `web-declarativo.yml` (módulo 37) para agregar:
   - Copiar e instalar el `.rpm` de `fedser-diag`.
   - Desplegar un contenedor Podman gestionado por una unidad
     `systemd` (mismo patrón del módulo 34), con su puerto abierto en
     firewalld.
   - Verificar `SELinux enforcing`.
3. Correr el playbook integrado, validar con `curl`/`fedser-diag` real.
4. Probar un ciclo real de actualización controlada + respaldo:
   cambiar algo a propósito, correr el playbook, y usar la snapshot
   como red de seguridad si hace falta (documentar honestamente el
   resultado real, sea cual sea).

## Hallazgos reales

_Pendiente — se completa durante la práctica guiada._

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
