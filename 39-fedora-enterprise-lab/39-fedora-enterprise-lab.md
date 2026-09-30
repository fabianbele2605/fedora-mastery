# Módulo 39 — Fedora Enterprise Lab (Proyecto final)

- Estado: Completado
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

1. **Integración exitosa en un solo intento**: extendiendo el playbook
   ya probado del módulo 37 (con las soluciones a los problemas reales
   de DNF5/`seport` ya incorporadas), el playbook integrado completo
   corrió sin ningún error nuevo — `ok=13 changed=5 failed=0` en la
   primera corrida.
2. **El `.rpm` de `fedser-diag` no necesitó copiarse**: como Ansible
   controla la misma VM donde se construyó en el módulo 28, el
   archivo ya vivía en `~/rpmbuild/RPMS/x86_64/` del nodo remoto —
   solo hubo que instalarlo con `dnf install -y <ruta local>` vía
   `command` (mismo patrón que evita el módulo `dnf` roto).
3. **Cuatro piezas del curso funcionando simultáneamente en el mismo
   servidor, validadas con evidencia real**: Cockpit (`HTTP 200`),
   `httpd` del módulo 17 (`<h1>fedora-mastery módulo 17</h1>`), el
   nuevo contenedor Podman gestionado por `systemd`
   (`It works! Apache httpd`), y `fedser-diag` reportando el estado
   real completo (Fedora 45, SELinux `Enforcing`, disco al 17%,
   memoria real).
4. **Idempotencia total confirmada en el entorno integrado completo**:
   segunda corrida del playbook completo → `ok=11 changed=0 failed=0`
   — ninguna de las piezas (httpd, `fedser-diag`, contenedor,
   firewalld, SELinux) necesitó ningún cambio.
5. **Ciclo real de actualización probado y exitoso**: cambiar la
   imagen del contenedor (`httpd:alpine` → `httpd:latest`) y volver a
   correr el playbook aplicó el cambio correctamente
   (`changed=2`, el *handler* recreó el contenedor) — confirmado con
   `podman inspect --format '{{.ImageName}}'` mostrando la imagen
   nueva real, y `curl` respondiendo sin interrupción.
6. **La recuperación de emergencia no hizo falta**, pero la snapshot
   `39-antes-proyecto-final` estuvo disponible como red de seguridad
   real durante todo el ejercicio — se documenta honestamente que el
   camino feliz funcionó, sin forzar un incidente artificial que no
   ocurrió.
7. **Cierre del curso completo**: las 8 fases (Fedora Server/Cockpit,
   RPM/DNF5/COPR, SELinux enforcing, Fedora Server avanzado,
   Rust/empaquetado RPM, Silverblue/CoreOS/rpm-ostree, automatización
   con Kickstart/Ansible) convergen en un solo entorno reproducible,
   con evidencia real de cada pieza integrándose sin fricciones
   nuevas — el trabajo de documentar honestamente cada error real a
   lo largo del curso (typos, incompatibilidades de versión,
   colecciones incompletas) fue justamente lo que permitió que esta
   integración final corriera limpia al primer intento.

## Evidencias

### Respaldo y arranque

**01 — Snapshot `39-antes-proyecto-final` creada, árbol completo de instantáneas del curso**

![Snapshot 39 antes proyecto final árbol completo](evidencias/01-snapshot-39-antes-proyecto-final-arbol-completo.png)

**02 — `Fedora_Server` arrancando (GRUB, Fedora 45)**

![Fedora Server arrancando GRUB 45](evidencias/02-fedora-server-arrancando-grub-45.png)

### Primera corrida del playbook integrado

**03 — Inicio: `Gathering Facts`, verificación de `httpd`**

![Playbook integrado primera corrida inicio](evidencias/03-playbook-integrado-primera-corrida-inicio.png)

**04 — `Listen 8586`, `seport`, firewalld (heredado del módulo 37)**

![Playbook integrado Listen 8586 seport firewalld](evidencias/04-playbook-integrado-listen-8586-seport-firewalld.png)

**05 — `fedser-diag` instalado (`changed`), unidad `systemd` de `web-enterprise` escrita (`changed`)**

![fedser-diag instalado unidad systemd escrita changed](evidencias/05-fedser-diag-instalado-unidad-systemd-escrita-changed.png)

**06 — Firewalld 8080, `web-enterprise` habilitado, SELinux enforcing confirmado, handler disparado**

![Firewalld 8080 web-enterprise SELinux enforcing handler](evidencias/06-firewalld-8080-web-enterprise-selinux-enforcing-handler.png)

**07 — Recap primera corrida: `ok=13 changed=5 failed=0`**

![Recap primera corrida ok13 changed5](evidencias/07-recap-primera-corrida-ok13-changed5.png)

### Validación de las cuatro piezas integradas

**08 — `curl` a Cockpit (200), httpd módulo 17 (8586), contenedor (8080), y `fedser-diag`**

![Validación Cockpit 200 httpd 8586 contenedor 8080 fedser-diag](evidencias/08-validacion-cockpit-200-httpd-8586-contenedor-8080-fedser-diag.png)

**09 — Reporte completo de `fedser-diag`: Fedora 45, SELinux Enforcing, disco 17%, memoria real**

![fedser-diag reporte completo memoria real](evidencias/09-fedser-diag-reporte-completo-memoria-real.png)

### Segunda corrida: idempotencia total del entorno integrado

**10 — Inicio de la segunda corrida**

![Segunda corrida idempotencia inicio](evidencias/10-segunda-corrida-idempotencia-inicio.png)

**11 — `Listen`/`seport`/firewalld: todo `ok` o `skipping`**

![Segunda corrida Listen seport firewalld continuación](evidencias/11-segunda-corrida-listen-seport-firewalld-continuacion.png)

**12 — `fedser-diag` y la unidad `systemd`: `skipping`, sin cambios**

![Segunda corrida fedser-diag systemd skipping](evidencias/12-segunda-corrida-fedser-diag-systemd-skipping.png)

**13 — Idempotencia total confirmada: `ok=11 changed=0 failed=0`**

![Idempotencia total recap ok11 changed0](evidencias/13-idempotencia-total-recap-ok11-changed0.png)

### Ciclo real de actualización controlada

**14 — `sed` cambia la imagen a `httpd:latest`, `grep` confirma el cambio en el playbook**

![sed actualiza imagen httpd latest grep confirmado](evidencias/14-sed-actualiza-imagen-httpd-latest-grep-confirmado.png)

**15-17 — Tercera corrida aplicando la actualización**

![Tercera corrida actualización inicio](evidencias/15-tercera-corrida-actualizacion-inicio.png)
![Tercera corrida Listen seport continuación](evidencias/16-tercera-corrida-listen-seport-continuacion.png)
![Tercera corrida fedser-diag changed continuación](evidencias/17-tercera-corrida-fedser-diag-changed-continuacion.png)

**18 — Recap de la actualización: `ok=12 changed=2 failed=0`**

![Actualización recap ok12 changed2](evidencias/18-actualizacion-recap-ok12-changed2.png)

**19 — Validación final: `curl` real + `podman inspect` confirma `httpd:latest` corriendo**

![Validación final curl podman inspect httpd latest](evidencias/19-validacion-final-curl-podman-inspect-httpd-latest.png)

## Pendientes
