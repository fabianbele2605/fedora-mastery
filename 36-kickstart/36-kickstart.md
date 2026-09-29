# Módulo 36 — Kickstart

- Estado: Completado
- Fecha: 2026-09-29
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial: arranca la Fase 7. Automatiza el mismo
  instalador Anaconda usado a mano en el módulo 01 (y en el 31 para
  Silverblue) — distinto de Ignition (módulo 33), que aplica
  configuración *después* de escribir una imagen ya compuesta.

## Concepto diferencial

Kickstart es un archivo de respuestas que Anaconda consume **durante**
la instalación interactiva: particionado, red, usuario, paquetes —
todo lo que se hizo a mano en el módulo 01, ahora declarado en texto
plano. A diferencia del método HTTP usado (con problemas) en el
módulo 33, acá se usa el mecanismo más robusto: un disco virtual
secundario con la etiqueta `OEMDRV`, que Anaconda detecta y carga
automáticamente sin red ni parámetros de arranque.

## Práctica guiada

1. Escribir `ks.cfg` con lo mínimo real: idioma, teclado, zona
   horaria, particionado automático, usuario, red, y una selección de
   paquetes.
2. Empaquetarlo en una imagen ISO con etiqueta de volumen `OEMDRV`:
   ```bash
   genisoimage -o oemdrv.iso -V OEMDRV -J -R ks.cfg
   ```
3. Crear una VM nueva (`Fedora_Kickstart`), con **dos** unidades
   ópticas: la ISO de Fedora Server (arranque) y `oemdrv.iso`
   (segunda unidad, el Kickstart).
4. Arrancar y observar: Anaconda debería avanzar solo, sin pantallas
   interactivas.
5. Validar el sistema resultante contra lo declarado en el `.ks`
   (hostname, usuario, zona horaria, paquetes).

## Hallazgos reales

1. **`genisoimage` ya estaba disponible en el host** (a diferencia de
   `podman`, que hubo que instalar en el módulo 33) — la ISO `OEMDRV`
   se generó sin fricción.
2. **VirtualBox detectó "Fedora" correctamente** en `OS Distribution`
   para esta ISO (contraste con "Red Hat" del módulo 31).
3. **Kickstart funcionó exactamente como se esperaba**: Anaconda saltó
   directo a "INSTALLATION PROGRESS" sin ninguna pantalla interactiva
   de idioma, red o particionado — contraste directo con la
   instalación 100% manual del módulo 01.
4. **Mismo incidente real de ISO no expulsada** que en los módulos 31
   y 33 (`reboot` en el Kickstart reinició automáticamente, pero
   ambas unidades ópticas — la de Server y `oemdrv.iso` — seguían
   adjuntas) — resuelto quitando ambos "attachments".
5. **Incidente real no planeado, el hallazgo más valioso del módulo**:
   la contraseña de `fbeleno` no funcionaba tras la instalación
   (`Login incorrect`). El primer intento de recuperación con
   `rd.break` reveló otro hallazgo real en cadena: *"Cannot open
   access to console, the root account is locked"* — consecuencia
   directa de `rootpw --lock` en el propio `ks.cfg` (buena práctica
   de seguridad que, sin querer, también bloqueó la vía estándar de
   recuperación vía `sulogin`).
6. **Solución real sin autenticación**: `init=/bin/bash` (reemplaza el
   proceso de arranque completo, sin systemd/PAM/sulogin de por
   medio) sí dio acceso de root directo. `passwd fbeleno` se ejecutó
   con éxito, pero **seguía sin funcionar el login** después de
   reiniciar.
7. **Diagnóstico de la causa real de fondo**: `ls -Z /etc/shadow`
   mostró `?` (sin contexto SELinux legible) tras escribirlo desde el
   entorno `init=/bin/bash`, que no tiene SELinux activo en absoluto
   — comparado con `/etc/passwd`, que sí conservaba su contexto
   correcto (`passwd_file_t`). `restorecon` no pudo reparar nada desde
   ese mismo entorno (sin política SELinux cargada para comparar).
8. **Reparación real y definitiva**: `touch /.autorelabel` + reinicio
   normal — SELinux relabeleó el sistema completo en el arranque
   siguiente (con la política ya cargada), y el login funcionó al
   primer intento.
9. **Validación final contra el `ks.cfg` declarado**: `hostnamectl`
   confirma `fedora-kickstart`, `sestatus` confirma `enforcing`
   (targeted), `timedatectl` confirma `America/Bogota`, y `rpm -q
   cockpit` confirma el paquete instalado — todo coincide con lo
   declarado en el archivo de texto, sin haber tocado un solo clic de
   Anaconda.

## Evidencias

### Preparación del Kickstart (host)

**01 — `mkdir kickstart-lab` + `openssl passwd -6`: hash generado**

![mkdir openssl passwd 6 hash generado](evidencias/01-mkdir-openssl-passwd-6-hash-generado.png)

**02 — `ks.cfg` escrito, confirmado con `cat`**

![ks cfg escrito cat confirmado](evidencias/02-ks-cfg-escrito-cat-confirmado.png)

**03 — `genisoimage`: `oemdrv.iso` creada con etiqueta `OEMDRV`**

![genisoimage oemdrv iso creada](evidencias/03-genisoimage-oemdrv-iso-creada.png)

### Creación de la VM

**04 — Asistente "Nueva máquina virtual" vacío**

![Nueva vm asistente inicial vacío](evidencias/04-nueva-vm-asistente-inicial-vacio.png)

**05 — Nombre `Fedora_Kickstart`, ISO de Server, `OS Distribution: Fedora`**

![Nombre Fedora Kickstart iso Server OS Distribution Fedora](evidencias/05-nombre-fedora-kickstart-iso-server-os-distribution-fedora.png)

**06 — Base Memory ajustada a 4096 MB**

![Hardware base memory 4096mb](evidencias/06-hardware-base-memory-4096mb.png)

**07 — Disco virtual de 45,86 GB, VDI dinámico**

![Disco virtual 45 86gb vdi](evidencias/07-disco-virtual-45-86gb-vdi.png)

**08 — Primer arranque, sin `oemdrv.iso` adjunta todavía (menú del instalador visible)**

![GRUB instalador primer arranque sin oemdrv](evidencias/08-grub-instalador-primer-arranque-sin-oemdrv.png)

### Adjuntando el Kickstart (OEMDRV)

**09 — Configuración → Almacenamiento: solo la ISO de Server adjunta**

![Configuración almacenamiento solo iso server](evidencias/09-configuracion-almacenamiento-solo-iso-server.png)

**10 — Selector de disco óptico: `oemdrv.iso` no aparece (nunca se agregó al catálogo)**

![Selector disco óptico oemdrv no listado](evidencias/10-selector-disco-optico-oemdrv-no-listado.png)

**11 — Navegando la carpeta "sistema operativo" (carpeta incorrecta)**

![Navegando carpeta sistema operativo](evidencias/11-navegando-carpeta-sistema-operativo.png)

**12 — Navegando la carpeta `kickstart-lab`: `oemdrv.iso` encontrado**

![Navegando carpeta kickstart lab oemdrv encontrado](evidencias/12-navegando-carpeta-kickstart-lab-oemdrv-encontrado.png)

**13 — `oemdrv.iso` agregado al catálogo, listo para seleccionar**

![oemdrv iso agregado catálogo seleccionar](evidencias/13-oemdrv-iso-agregado-catalogo-seleccionar.png)

**14 — Ambas unidades ópticas adjuntas: ISO de Server + `oemdrv.iso`**

![Ambas unidades ópticas adjuntas server oemdrv](evidencias/14-ambas-unidades-opticas-adjuntas-server-oemdrv.png)

### Instalación automatizada con Kickstart

**15 — Menú de GRUB con "Troubleshooting" resaltado por error, corregido antes de arrancar**

![GRUB menú troubleshooting resaltado error](evidencias/15-grub-menu-troubleshooting-resaltado-error.png)

**16 — Anaconda arrancando; avisos benignos de `vmwgfx` (driver gráfico virtual)**

![Anaconda arrancando vmwgfx warning benigno](evidencias/16-anaconda-arrancando-vmwgfx-warning-benigno.png)

**17 — Anaconda iniciando el instalador, sin ninguna pantalla interactiva**

![Anaconda instalador iniciando sin pantallas](evidencias/17-anaconda-instalador-iniciando-sin-pantallas.png)

**18 — "INSTALLATION PROGRESS": particionado automático (`Creating xfs on /dev/sda2`)**

![Installation progress creating xfs automático](evidencias/18-installation-progress-creating-xfs-automatico.png)

**19 — Progreso de instalación de paquetes: `kernel-modules.x86_64 (486/619)`**

![Installation progress kernel modules 486 619](evidencias/19-installation-progress-kernel-modules-486-619.png)

### Incidente real: ISO no expulsada (mismo patrón de módulos anteriores)

**20 — Tras el `reboot` del Kickstart, la VM volvió al menú del instalador**

![GRUB vuelve a troubleshooting iso no expulsada](evidencias/20-grub-vuelve-a-troubleshooting-iso-no-expulsada.png)

**21 — Configuración → Almacenamiento: ambas ISOs seguían adjuntas**

![Configuración almacenamiento ambas iso adjuntas](evidencias/21-configuracion-almacenamiento-ambas-iso-adjuntas.png)

**22 — Confirmando "Eliminar" la primera unidad óptica**

![Confirmar eliminar primera unidad óptica](evidencias/22-confirmar-eliminar-primera-unidad-optica.png)

**23 — Controlador IDE ya sin ningún medio adjunto**

![Controlador IDE vacío ambas iso quitadas](evidencias/23-controlador-ide-vacio-ambas-iso-quitadas.png)

**24 — Arranque real desde el disco instalado: dos entradas de GRUB (normal y rescate)**

![GRUB boot real disco instalado dos entradas](evidencias/24-grub-boot-real-disco-instalado-dos-entradas.png)

### Validación inicial y el problema real de la contraseña

**25 — Prompt de login confirma el hostname declarado: `fedora-kickstart login:`**

![Login hostname fedora kickstart confirmado](evidencias/25-login-hostname-fedora-kickstart-confirmado.png)

**26 — Primer intento de login: "Login incorrect"**

![Primer intento login incorrect](evidencias/26-primer-intento-login-incorrect.png)

**27 — Diálogo "Apagar la máquina" para reintentar con modo rescate**

![Diálogo apagar máquina](evidencias/27-dialogo-apagar-maquina.png)

**28 — VM apagada, detalles confirmados**

![VM apagada detalles](evidencias/28-vm-apagada-detalles.png)

**29 — El "Login incorrect" persiste en un segundo intento**

![Login incorrect persiste segundo intento](evidencias/29-login-incorrect-persiste-segundo-intento.png)

### Primer intento de recuperación: `rd.break` (bloqueado por `rootpw --lock`)

**30 — Entrada `(0-rescue-...)` seleccionada en el menú de GRUB**

![GRUB menú rescue seleccionado](evidencias/30-grub-menu-rescue-seleccionado.png)

**31-32 — Pantalla en negro mientras carga el entorno de rescate**

![Pantalla negra cargando rescue](evidencias/31-pantalla-negra-cargando-rescue.png)
![Pantalla negra integración ratón](evidencias/32-pantalla-negra-integracion-raton.png)

**33 — La entrada de rescate terminó en el login normal (con un carácter de escape suelto)**

![Rescue arrancó login normal carácter escape](evidencias/33-rescue-arranco-login-normal-caracter-escape.png)

**34 — Pantalla de edición de GRUB, línea original terminando en `rhgb quiet`**

![GRUB edición línea original rhgb quiet](evidencias/34-grub-edicion-linea-original-rhgb-quiet.png)

**35 — `Fin` no llegó al final real de la línea lógica (solo del renglón visual)**

![Fin no llegó al final real línea](evidencias/35-fin-no-llego-al-final-real-linea.png)

**36 — `Esc` para descartar y reintentar desde cero**

![Esc descartar cambios vuelta inicio](evidencias/36-esc-descartar-cambios-vuelta-inicio.png)

**37 — `rd.break` agregado correctamente al final de la línea**

![rd break agregado correctamente](evidencias/37-rd-break-agregado-correctamente.png)

**38 — Ctrl+X arrancando con el parámetro modificado**

![Ctrl X arrancando command list](evidencias/38-ctrl-x-arrancando-command-list.png)

**39 — Hallazgo real: "Cannot open access to console, the root account is locked" (`sulogin`)**

![Hallazgo root account locked sulogin](evidencias/39-hallazgo-root-account-locked-sulogin.png)

### Segundo intento: `init=/bin/bash` (acceso sin autenticación)

**40 — Segunda edición de GRUB, línea original limpia**

![GRUB edición segundo intento limpio](evidencias/40-grub-edicion-segundo-intento-limpio.png)

**41 — `init=/bin/bash` agregado correctamente**

![init bin bash agregado correctamente](evidencias/41-init-bin-bash-agregado-correctamente.png)

**42 — Shell de root directa (`bash-5.3#`), sin pedir autenticación**

![Shell root directa bash 5 3](evidencias/42-shell-root-directa-bash-5-3.png)

**43 — `mount -o remount,rw /` + `passwd fbeleno`: contraseña actualizada exitosamente**

![mount remount rw passwd fbeleno actualizada](evidencias/43-mount-remount-rw-passwd-fbeleno-actualizada.png)

**44 — `sync` confirmado**

![sync confirmado](evidencias/44-sync-confirmado.png)

**45 — Diálogo de confirmación para reiniciar la VM**

![Diálogo confirmar reiniciar VM](evidencias/45-dialogo-confirmar-reiniciar-vm.png)

**46 — El login **sigue** fallando tras el primer cambio de contraseña**

![Login sigue incorrecto tras primer passwd](evidencias/46-login-sigue-incorrecto-tras-primer-passwd.png)

### Diagnóstico de la causa real: contexto SELinux roto

**47-48 — Tercer intento de edición de GRUB, ajustando el cursor con cuidado**

![GRUB edición tercer intento línea original](evidencias/47-grub-edicion-tercer-intento-linea-original.png)
![Cursor no llegó al final segundo intento](evidencias/48-cursor-no-llego-al-final-segundo-intento.png)

**49 — `init=/bin/bash` agregado correctamente una tercera vez**

![init bin bash agregado tercera vez](evidencias/49-init-bin-bash-agregado-tercera-vez.png)

**50 — Shell de root directa, segunda vez**

![Shell root directa segunda vez](evidencias/50-shell-root-directa-segunda-vez.png)

**51 — Hallazgo clave: `ls -Z /etc/shadow` → `?` (sin contexto), `/etc/passwd` sí con contexto correcto**

![ls Z shadow sin contexto passwd con contexto](evidencias/51-ls-z-shadow-sin-contexto-passwd-con-contexto.png)

**52 — `passwd` de nuevo + `restorecon` sin efecto: `/etc/shadow` sigue sin contexto legible**

![passwd restorecon sin efecto shadow sigue roto](evidencias/52-passwd-restorecon-sin-efecto-shadow-sigue-roto.png)

**53 — `touch /.autorelabel` + `sync`: solución real definitiva**

![touch autorelabel sync](evidencias/53-touch-autorelabel-sync.png)

### Arranque final con relabeling y validación completa

**54-56 — Arranque normal, sin editar nada, dejando que SELinux relabele el sistema**

![GRUB menú entrada normal seleccionada](evidencias/54-grub-menu-entrada-normal-seleccionada.png)
![GRUB automatic boot 3s](evidencias/55-grub-automatic-boot-3s.png)
![Booting Fedora Server Edition](evidencias/56-booting-fedora-server-edition.png)

**57 — Login exitoso: `fbeleno@fedora-kickstart:~$`**

![Login exitoso fbeleno fedora kickstart](evidencias/57-login-exitoso-fbeleno-fedora-kickstart.png)

**58 — `hostnamectl` + `sestatus`: hostname y SELinux `enforcing` confirmados**

![hostnamectl sestatus confirmados](evidencias/58-hostnamectl-sestatus-confirmados.png)

**59 — `timedatectl` + `rpm -q cockpit`: zona horaria y paquete confirmados**

![timedatectl rpm q cockpit confirmados](evidencias/59-timedatectl-rpm-q-cockpit-confirmados.png)

## Pendientes
