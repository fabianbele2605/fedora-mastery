# Módulo 37 — Ansible aplicado a Fedora

- Estado: En curso
- Fecha: 2026-09-29
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial: arranca la Fase 7. No repite fundamentos de
  Ansible (playbooks, inventarios básicos) — se enfoca en los módulos
  específicos de Fedora: `ansible.builtin.dnf`, `ansible.posix.seport`
  (contexto de puerto SELinux), `ansible.posix.firewalld`.

## Concepto diferencial

Reproduce declarativamente el mismo patrón de dos capas de seguridad
del módulo 17 (SELinux + firewalld para un servicio web en puerto no
estándar), pero con la propiedad central de Ansible: **idempotencia**.
Correr el mismo playbook dos veces debe dar el mismo resultado sin
duplicar cambios — la segunda corrida debe reportar `changed=0`.

## Práctica guiada

1. Instalar Ansible en el host (control node).
2. Confirmar conectividad SSH hacia `Fedora_Server` (módulo 06).
3. Escribir un inventario mínimo y un playbook que:
   - Instale `httpd` (`ansible.builtin.dnf`).
   - Agregue el puerto 8586/tcp al contexto `http_port_t`
     (`ansible.posix.seport`).
   - Abra 8586/tcp en la zona activa de firewalld
     (`ansible.posix.firewalld`).
   - Habilite y arranque `httpd` (`ansible.builtin.service`).
4. Correr el playbook una primera vez — observar los `changed`.
5. Correr el playbook una segunda vez — confirmar `changed=0`.
6. Validar con `curl` real desde el host.

## Hallazgos reales

_Pendiente — se completa durante la práctica guiada._

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
