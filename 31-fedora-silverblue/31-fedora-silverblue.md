# Módulo 31 — Fedora Silverblue

- Estado: Completado
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

1. **Instalación real bloqueada por ISO no expulsada**: tras terminar
   Anaconda, la VM reinició y volvió a arrancar desde la ISO (aún
   "insertada" en la unidad óptica virtual) en vez del disco ya
   instalado — se resolvió quitando el archivo adjunto de la unidad
   IDE en VirtualBox antes de reiniciar.
2. **La bienvenida de la terminal ya adelanta el concepto del módulo,
   textualmente**: *"This terminal is running on the host system. You
   may want to try out the Toolbox for a directly mutable
   environment..."* — Silverblue mismo avisa que el host no es para
   `dnf install`.
3. **`rpm-ostree status` no lista paquetes individuales**: muestra un
   único despliegue (`fedora:fedora/44/x86_64/silverblue`) identificado
   por un **commit hash** (análogo a Git) y una **firma GPG de todo el
   árbol**, muy distinto de la firma paquete-por-paquete del módulo 10.
4. **`rpm-ostree` serializa transacciones**: el primer intento de
   layering (`rpm-ostree install htop`) falló con
   `error: Transaction in progress: refresh-md` — un `refresh-md`
   automático de `gnome-software` seguía corriendo. Se resolvió con
   `rpm-ostree cancel`, no fue un problema de red.
5. **Layering real confirmado**: `rpm-ostree install htop` terminó con
   *"Staging deployment... done"* y *"Changes queued for next boot"* —
   el paquete no quedó disponible hasta `systemctl reboot`.
   `rpm-ostree status` mostró **dos despliegues coexistiendo** (mismo
   `BaseCommit`, uno con `LayeredPackages: htop` y otro sin capas),
   confirmando que nada se sobrescribe.
6. **Contraste directo con el mismo paquete por dos vías**: dentro de
   `toolbox enter` (contenedor `fedora-toolbox-44`, creado vía Podman —
   conecta con el módulo 21), `sudo dnf install htop` se instaló **al
   instante**, sin transacción única ni reinicio — mismo paquete,
   mecanismo opuesto al layering del host.
7. **`neofetch` ya no existe en los repos de Fedora** (descontinuado,
   reemplazado por `fastfetch`) — corregido usando `htop` en su lugar
   para el contraste.
8. **Matiz real sobre Flatpak**: `flatpak list` mostró casi todas las
   apps de GNOME preinstaladas como Flatpak, pero **Firefox no
   apareció ahí** — `which firefox` resolvió a `/usr/bin/firefox`, y
   `rpm -q firefox` confirmó que viene incluido directamente en la
   imagen base OSTree, no como Flatpak ni como layered package. La
   separación "apps = Flatpak" tiene esta excepción histórica real.

## Evidencias

### Creación de la VM

**01 — Árbol de instantáneas antes de crear la VM (VirtualBox)**

![Snapshots árbol antes crear VM](evidencias/01-virtualbox-snapshots-arbol-antes-crear-vm.png)

**02 — Asistente "Nueva máquina virtual" recién abierto**

![Asistente inicial vacío](evidencias/02-nueva-vm-asistente-inicial-vacio.png)

**03 — Nombre `Fedora_Silverblue` e ISO ya seleccionada**

![Nombre y ISO seleccionada](evidencias/03-nombre-fedora-silverblue-iso-seleccionada.png)

**04 — Desplegable de ISOs recientes (confirmando la ruta correcta)**

![Desplegable ISO recientes](evidencias/04-desplegable-iso-image-recientes.png)

**05 — Hallazgo real: "OS Distribution" quedó bloqueado en "Red Hat" con la ISO seleccionada**

![OS Distribution Red Hat bloqueado](evidencias/05-os-distribution-red-hat-bloqueado.png)

**06 — Base Memory con el valor por defecto (5132 MB)**

![Base Memory default 5132MB](evidencias/06-hardware-base-memory-default-5132mb.png)

**07 — Base Memory ajustada a 4108 MB (~4 GB, según la tabla del curso)**

![Base Memory ajustada 4108MB](evidencias/07-hardware-base-memory-ajustada-4108mb.png)

**08 — Disco virtual de 40 GB, formato VDI dinámico**

![Disco virtual 40GB VDI](evidencias/08-disco-virtual-40gb-vdi-dinamico.png)

**09 — Advertencia de "Unattended Installation" por falta de contraseña**

![Unattended installation advertencia](evidencias/09-unattended-installation-advertencia-password.png)

**10 — Instalación desatendida desmarcada, listo para "Terminar"**

![Unattended installation desmarcada](evidencias/10-unattended-installation-desmarcada-terminar.png)

### Instalación con Anaconda

**11 — Menú de GRUB del instalador, primer arranque**

![GRUB instalador primer arranque](evidencias/11-grub-instalador-primer-arranque.png)

**12 — Arranque de Anaconda: servicios de systemd inicializando**

![Anaconda arranque systemd](evidencias/12-anaconda-arranque-systemd-servicios.png)

**13 — Bienvenida de Anaconda, idioma Español**

![Anaconda bienvenida idioma español](evidencias/13-anaconda-bienvenida-idioma-espanol.png)

**14 — Resumen de instalación: sin sección "Instalación de software" (imagen fija, no hay selección de paquetes)**

![Anaconda resumen sin selección de software](evidencias/14-anaconda-resumen-sin-seleccion-software.png)

**15 — Destino de la instalación: disco de 40 GiB detectado**

![Anaconda destino instalación disco 40GiB](evidencias/15-anaconda-destino-instalacion-disco-40gib.png)

**16 — Resumen final, listo para "Comenzar instalación"**

![Anaconda resumen listo comenzar](evidencias/16-anaconda-resumen-listo-comenzar.png)

### Incidente real: ISO no expulsada tras el primer reinicio

**17 — Tras terminar la instalación, la VM reinició y volvió al menú de GRUB del instalador**

![Problema reinicio vuelve a GRUB instalador](evidencias/17-problema-reinicio-vuelve-a-grub-instalador.png)

**18 — Configuración → Almacenamiento: la ISO seguía adjunta al controlador IDE**

![Configuración almacenamiento ISO adjunta](evidencias/18-configuracion-almacenamiento-ide-iso-adjunta.png)

**19 — Primer intento (equivocado): diálogo para eliminar la unidad óptica completa, cancelado**

![Diálogo eliminar unidad óptica cancelado](evidencias/19-dialogo-eliminar-unidad-optica-cancelado.png)

**20 — Panel de atributos del "Optical Drive" bajo el controlador IDE**

![Optical Drive atributos dispositivo IDE](evidencias/20-optical-drive-atributos-dispositivo-ide.png)

**21 — Opción correcta encontrada: "Remove attachment" en el desplegable de la unidad óptica**

![Menú Remove attachment](evidencias/21-menu-remove-attachment-optical-drive.png)

**22 — Confirmación de "Remove attachment" (mismo diálogo de eliminar unidad óptica)**

![Confirmar eliminar unidad óptica](evidencias/22-confirmar-eliminar-unidad-optica.png)

**23 — Controlador IDE ya sin ningún medio adjunto**

![Controlador IDE vacío tras quitar ISO](evidencias/23-controlador-ide-vacio-tras-quitar-iso.png)

**24 — Detalles de la VM: almacenamiento reducido a solo el VDI (SATA)**

![Detalles VM storage solo VDI SATA](evidencias/24-detalles-vm-storage-solo-vdi-sata.png)

### Primer arranque real y configuración de GNOME

**25 — GRUB arrancando de verdad: `Booting 'Fedora Linux 44.1.7 (Silverblue) (ostree:0)'`**

![GRUB boot real ostree:0](evidencias/25-grub-boot-real-ostree-0.png)

**26 — Splash de arranque (Plymouth)**

![Splash arranque Plymouth](evidencias/26-splash-arranque-plymouth.png)

**27 — Splash con el logo de Fedora**

![Splash logo Fedora](evidencias/27-splash-logo-fedora.png)

**28 — Fondo de escritorio cargando, antes del asistente inicial**

![Fondo escritorio cargando](evidencias/28-fondo-escritorio-cargando.png)

**29 — `gnome-initial-setup`: bienvenida y selección de idioma**

![GNOME initial setup bienvenida idioma](evidencias/29-gnome-initial-setup-bienvenida-idioma.png)

**30 — Pantalla de privacidad (servicios de ubicación)**

![GNOME initial setup privacidad ubicación](evidencias/30-gnome-initial-setup-privacidad-ubicacion.png)

**31 — Zona horaria detectada: Bogotá, Colombia (UTC-05)**

![GNOME initial setup zona horaria Bogotá](evidencias/31-gnome-initial-setup-zona-horaria-bogota.png)

**32 — "Acerca de usted": campos vacíos, a completar**

![GNOME initial setup acerca de usted vacío](evidencias/32-gnome-initial-setup-acerca-de-usted-vacio.png)

**33 — Nombre completo "Fabian Beleño", usuario `fbeleno` autocompletado**

![GNOME initial setup Fabian Beleño fbeleno](evidencias/33-gnome-initial-setup-fabian-beleno-fbeleno.png)

**34 — Establecimiento de la contraseña del usuario**

![GNOME initial setup contraseña](evidencias/34-gnome-initial-setup-contrasena.png)

**35 — "Configuración completada", listo para usar el sistema**

![GNOME initial setup configuración completada](evidencias/35-gnome-initial-setup-configuracion-completada.png)

**36 — Bienvenida de Fedora Linux 44.1.7 (Silverblue), tour omitido**

![Bienvenida Fedora 44 Silverblue omitir tour](evidencias/36-bienvenida-fedora-44-silverblue-omitir-tour.png)

**37 — Escritorio GNOME limpio, sin ventanas**

![Escritorio GNOME vacío](evidencias/37-escritorio-gnome-vacio.png)

**38 — Vista de Actividades con todas las apps preinstaladas**

![Vista actividades todas las apps](evidencias/38-vista-actividades-todas-las-apps.png)

**39 — Búsqueda de "terminal" en Actividades**

![Búsqueda terminal resultados](evidencias/39-busqueda-terminal-resultados.png)

**40 — Primera terminal abierta: el mensaje de bienvenida ya menciona Toolbox explícitamente**

![Terminal bienvenida Toolbox mensaje textual](evidencias/40-terminal-bienvenida-toolbox-mensaje-textual.png)

### `rpm-ostree`: status, layering y reinicio

**41 — `rpm-ostree status`: un único despliegue, commit hash y firma GPG del árbol completo (con un `refresh-md` de `gnome-software` en curso)**

![rpm-ostree status busy refresh-md commit GPG](evidencias/41-rpm-ostree-status-busy-refresh-md-commit-gpg.png)

**42 — `rpm-ostree install htop` falla: `error: Transaction in progress: refresh-md`**

![rpm-ostree install htop error transaction in progress](evidencias/42-rpm-ostree-install-htop-error-transaction-in-progress.png)

**43 — `rpm-ostree cancel` + reintento: descarga de `htop`/`hwloc-libs` en curso**

![rpm-ostree cancel install htop descarga inicio](evidencias/43-rpm-ostree-cancel-install-htop-descarga-inicio.png)

**44 — Layering completado: "Staging deployment... done", cambios en cola para el próximo reinicio**

![rpm-ostree install htop staging deployment reboot](evidencias/44-rpm-ostree-install-htop-staging-deployment-reboot.png)

**45 — `rpm-ostree status` con dos despliegues coexistiendo (uno con `LayeredPackages: htop`, sin activar aún)**

![rpm-ostree status dos despliegues LayeredPackages htop](evidencias/45-rpm-ostree-status-dos-despliegues-layeredpackages-htop.png)

**46 — Pantalla de login (GDM) tras el reinicio**

![GDM login post reboot](evidencias/46-gdm-login-post-reboot.png)

**47 — Escritorio tras iniciar sesión, ya sobre el despliegue con `htop`**

![Escritorio tras login post layering](evidencias/47-escritorio-tras-login-post-layering.png)

**48 — `rpm-ostree status` confirma el despliegue con `htop` activo (`•`), y `htop --version` funciona**

![rpm-ostree status htop activo htop version confirmado](evidencias/48-rpm-ostree-status-htop-activo-htop-version-confirmado.png)

### Toolbox: contraste con DNF tradicional

**49 — `toolbox create`: descarga de la imagen `registry.fedoraproject.org/fedora-toolbox:44`**

![toolbox create descarga imagen registry](evidencias/49-toolbox-create-descarga-imagen-registry.png)

**50 — Contenedor `fedora-toolbox-44` creado**

![toolbox create contenedor creado fedora-toolbox-44](evidencias/50-toolbox-create-contenedor-creado-fedora-toolbox-44.png)

**51 — `toolbox enter` + `cat /etc/os-release`: confirma "Toolbox Container Image"**

![toolbox enter os-release fedora toolbox container](evidencias/51-toolbox-enter-os-release-fedora-toolbox-container.png)

**52 — `/etc/os-release` completo dentro del Toolbox**

![os-release toolbox completo](evidencias/52-os-release-toolbox-completo.png)

**53 — Hallazgo real: `dnf install neofetch` falla, paquete descontinuado en los repos de Fedora**

![dnf install neofetch error no coincide argumento](evidencias/53-dnf-install-neofetch-error-no-coincide-argumento.png)

**54 — `dnf install htop` dentro del Toolbox: resumen de la transacción**

![dnf install htop toolbox resumen transacción](evidencias/54-dnf-install-htop-toolbox-resumen-transaccion.png)

**55 — Progreso de la descarga de `htop`/`hwloc-libs`**

![dnf install htop toolbox progreso descarga](evidencias/55-dnf-install-htop-toolbox-progreso-descarga.png)

**56 — Instalación instantánea completada, `htop --version` confirma la misma versión que en el host**

![dnf install htop toolbox completado versión instantánea](evidencias/56-dnf-install-htop-toolbox-completado-version-instantanea.png)

### Flatpak y el matiz real de Firefox

**57 — `flatpak list`: casi todas las apps de GNOME preinstaladas como Flatpak**

![flatpak list apps GNOME preinstaladas](evidencias/57-flatpak-list-apps-gnome-preinstaladas.png)

**58 — `which firefox` → `/usr/bin/firefox` (ruta tradicional, no aparece en `flatpak list`)**

![which firefox usr bin no flatpak](evidencias/58-which-firefox-usr-bin-no-flatpak.png)

**59 — `rpm -q firefox` confirma: viene incluido en la imagen base OSTree, no como Flatpak**

![rpm -q firefox confirma imagen base](evidencias/59-rpm-q-firefox-confirma-imagen-base.png)

## Pendientes
