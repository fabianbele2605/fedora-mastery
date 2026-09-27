# Módulo 23 — Actualizaciones entre versiones de Fedora Server

- Estado: Completado
- Fecha: 2026-09-27
- Versión de Fedora: Fedora Linux 44 → 45 (Server Edition)
- Objetivo diferencial frente a Arch: Fedora tiene versiones mayores
  discretas con ciclo de vida definido — Arch es rolling-release, no
  existe el concepto de "saltar de versión". El mecanismo
  (`dnf system-upgrade`) descarga todo primero y aplica los cambios en
  modo offline tras un reinicio, muy distinto a un `dnf upgrade` normal.

## Entorno y punto de restauración

Snapshot `22-antes-upgrade-mayor-f45` tomado antes de este módulo:
Fedora Server 44 completo, con Cockpit, `httpd` en 8585, contenedor
Podman rootless, SELinux enforcing, virtualización anidada habilitada.

## Concepto diferencial

`dnf system-upgrade` (plugin aparte de `dnf`) no actualiza en caliente:
descarga todos los paquetes de la nueva versión primero
(`system-upgrade download`), y recién al reiniciar entra en un modo
especial fuera de línea donde aplica la transacción completa
(`system-upgrade reboot`). Es una operación de alto riesgo real —
toca el sistema entero, no un solo servicio.

## Práctica guiada

```bash
cat /etc/fedora-release
sudo dnf install -y dnf-plugin-system-upgrade
sudo dnf system-upgrade download --releasever=45 -y
```

Luego, tras confirmar la descarga:

```bash
sudo dnf system-upgrade reboot
```

## Validación post-actualización (plan de reversión)

```bash
cat /etc/fedora-release
systemctl status cockpit.socket httpd podman
curl -s http://localhost:8585
sestatus
```

## Hallazgos reales

1. **1083 paquetes en la transacción** (48 instalando, 1033
   modernizando, 1041 reemplazando, 2 degradando; 1 GiB a descargar) —
   número más alto de lo típico porque el stack completo de `libvirt`
   instalado en el módulo 22 (con todos sus drivers de storage) infla
   bastante el conteo de un upgrade mayor.
2. **DNF5 usa `dnf5 offline reboot`**, no `dnf system-upgrade reboot`
   — el mensaje final de la descarga sugirió explícitamente el comando
   nuevo integrado, aunque el plugin `dnf-plugin-system-upgrade`
   sigue siendo necesario para iniciar la descarga.
3. **Clave GPG de Fedora 45 importada automáticamente**
   (`RPM-GPG-KEY-fedora-45-x86_64`) durante la preparación de la
   transacción offline — confirma en la práctica el mecanismo del
   módulo 10 (cada release trae su propia clave).
4. **GRUB conservó la entrada de Fedora 44 como respaldo** tras
   completar la actualización — el plan de reversión real: si algo
   falla, se puede arrancar el kernel viejo desde el mismo menú sin
   restaurar el snapshot.
5. **F45 aparece como "Prerelease"** — consistente con el ciclo real de
   Fedora (Rawhide → Branched/Beta → estable).
6. **Validación post-actualización, todo sobrevivió**:
   `cat /etc/fedora-release` → `Fedora release 45 (Forty Five)`;
   `cockpit.socket` y `httpd.service` activos sin cambios;
   `curl localhost:8585` sigue devolviendo el contenido real;
   `sestatus` confirma `enforcing` sin interrupciones durante toda la
   actualización.
7. **Hallazgo menor real**: `httpd` mostró un aviso nuevo post-upgrade
   (`Could not reliably determine the server's fully qualified domain
   name`) — benigno, típico por falta de `ServerName` explícito.
8. **El contenedor de Podman (módulo 21) no sobrevivió el reinicio
   como "corriendo"**: `podman ps` lo mostró vacío, pero
   `podman ps -a` confirmó que solo quedó `Exited (0)` (salida limpia,
   nada se perdió) — coherente con no haber configurado ninguna unidad
   `systemd --user` para que el contenedor rootless se relance solo al
   reiniciar el sistema.

## Evidencias

**01 — Fedora 44 confirmado, plugin instalado, inicio de la descarga**

![Fedora44 instalar plugin inicio descarga](evidencias/01-fedora44-instalar-plugin-inicio-descarga.png)

**02 — Resumen de la transacción: 1083 paquetes (modernización)**

![Resumen modernización 1083 paquetes](evidencias/02-resumen-modernizacion-1083-paquetes.png)

**03 — Descarga en progreso (paquetes de `libvirt`)**

![Descarga en progreso libvirt](evidencias/03-descarga-en-progreso-libvirt.png)

**04 — Descarga completa, clave GPG de F45 importada, sugerencia de `dnf5 offline reboot`**

![Descarga completa GPG importada sugerencia dnf5 offline](evidencias/04-descarga-completa-gpg-importada-sugerencia-dnf5-offline.png)

**05 — Confirmación del reinicio (`sudo dnf5 offline reboot`, prompt `y/N`)**

![dnf5 offline reboot confirmación](evidencias/05-dnf5-offline-reboot-confirmacion.png)

**06 — Reiniciando**

![Rebooting](evidencias/06-rebooting.png)

**07 — GRUB antes de aplicar la transacción offline (aún Fedora 44)**

![GRUB F44 antes de aplicar offline](evidencias/07-grub-f44-antes-de-aplicar-offline.png)

**08 — "Inicializando modernización del sistema..."**

![Inicializando modernización sistema](evidencias/08-inicializando-modernizacion-sistema.png)

**09 — Transacción offline aplicándose ([311/2124] `ca-certificates`)**

![Progreso transacción offline ca-certificates](evidencias/09-progreso-transaccion-offline-ca-certificates.png)

**10 — GRUB tras completar: Fedora 45 (Prerelease) como nueva entrada por defecto, Fedora 44 conservada como respaldo**

![GRUB F45 nuevo default F44 respaldo](evidencias/10-grub-f45-nuevo-default-f44-respaldo.png)

**11 — Pantalla de login en Fedora 45**

![Login Fedora 45](evidencias/11-login-fedora45.png)

**12 — Shell tras el primer login en F45**

![Shell tras login F45](evidencias/12-shell-tras-login-f45.png)

**13 — `systemctl status cockpit.socket httpd podman.socket`: todo activo, incluye el aviso benigno de FQDN de `httpd`**

![systemctl status cockpit httpd podman socket](evidencias/13-systemctl-status-cockpit-httpd-podman-socket.png)

**14 — `cat /etc/fedora-release` (45), `curl localhost:8585` (contenido real), `sestatus` (`enforcing` sin interrupciones)**

![fedora-release 45 curl sestatus](evidencias/14-fedora-release-45-curl-sestatus.png)

**15 — `podman ps`: vacío tras el reinicio**

![podman ps vacío](evidencias/15-podman-ps-vacio.png)

**16 — `podman ps -a`: el contenedor del módulo 21 sobrevivió con salida limpia (`Exited (0)`)**

![podman ps -a exited sobrevivió](evidencias/16-podman-ps-a-exited-sobrevivio.png)

## Pendientes
