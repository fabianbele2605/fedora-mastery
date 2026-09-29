# Módulo 33 — Fedora CoreOS

- Estado: Completado
- Fecha: 2026-09-28
- Versión de Fedora: Fedora CoreOS (canal estable más reciente)
- Objetivo diferencial frente a Arch y frente a Silverblue: CoreOS
  también es inmutable (basado en OSTree, como Silverblue), pero no se
  instala de forma interactiva con un asistente — se aprovisiona de
  forma **declarativa** con Ignition/Butane, pensado para hosts de
  contenedores sin intervención humana repetible.

## Concepto diferencial

- **Butane**: un YAML legible pensado para que lo escriba un humano
  (usuarios, claves SSH, archivos, unidades systemd).
- **Ignition**: el JSON de bajo nivel que Butane genera al compilarse
  — es lo que el firmware de arranque de CoreOS realmente interpreta
  la primera vez que arranca el disco.
- **No se descarga ningún JSON de Fedora** — el Ignition siempre se
  genera localmente, a partir de un Butane propio, con la herramienta
  `butane`.

## Entorno

VM temporal nueva, 2 GB de RAM (tabla de entorno del curso). A
diferencia de Silverblue/Server, el "instalador" es un binario
(`coreos-installer`) que corre dentro del propio Live ISO, no un
asistente gráfico tipo Anaconda.

## Práctica guiada

1. Descargar la ISO Live de Fedora CoreOS (canal estable).
2. Escribir `config.bu` (Butane) mínimo:
   ```yaml
   variant: fcos
   version: 1.5.0
   passwd:
     users:
       - name: core
         password_hash: <hash>
   ```
3. Compilar a Ignition:
   ```bash
   podman run --rm -i quay.io/coreos/butane:release \
     --pretty --strict < config.bu > config.ign
   ```
4. Crear la VM (2 GB RAM), arrancar desde la ISO Live.
5. Desde el Live, instalar aplicando el Ignition:
   ```bash
   sudo coreos-installer install /dev/sda --ignition-file config.ign
   ```
6. Quitar la ISO de la unidad óptica (mismo paso aprendido en el
   módulo 31) y reiniciar al sistema ya aprovisionado.
7. Iniciar sesión como `core` y validar `rpm-ostree status`.

## Hallazgos reales

1. **No se descarga ningún JSON de Fedora**: el `config.ign` se generó
   100% localmente, compilando un `config.bu` propio con `butane` (vía
   Podman) — confirmado comparando el YAML de entrada con el JSON de
   salida (`passwordHash` en formato camelCase, `"ignition": {
   "version": "3.4.0" }`).
2. **El host (Ubuntu) no trae Podman preinstalado**, a diferencia de
   Fedora Server (módulo 21) — instalado con `sudo apt install
   podman`.
3. **Problema real de red no resuelto, sí sorteado con criterio**:
   intentar servir `config.ign` por HTTP (`python3 -m http.server`)
   desde el host y descargarlo desde la VM vía `10.0.2.2` (gateway NAT
   de VirtualBox) falló persistentemente con `curl: (7) ... Network is
   unreachable`, en 0-1 ms, pese a que `ping` sí funcionaba, `ss`
   confirmaba el servidor escuchando en `0.0.0.0:8000`, y se agregó
   una regla de `ufw` permitiendo esa subred — la causa exacta quedó
   sin diagnosticar del todo. Se resolvió cambiando de estrategia:
   pegar el contenido del `.ign` directamente en la terminal de la VM
   con un heredoc (mismo patrón usado en los módulos 25/26/28), sin
   depender de la red.
4. **`nc` no existe en el entorno Live de CoreOS** — es
   deliberadamente minimalista, ni siquiera trae herramientas básicas
   de diagnóstico de red más allá de `ping`/`curl`.
5. **`coreos-installer install` es muchísimo más rápido que Anaconda**:
   "Read disk 2.9 GiB/2.9 GiB (100%)" y "Writing Ignition config" en
   segundos — es escribir una imagen ya compuesta al disco, no una
   transacción de paquetes.
6. **Confirmación textual del aprovisionamiento real**: el primer
   arranque post-instalación mostró explícitamente `Ignition:
   user-provided config was applied` (a diferencia del Live, que
   decía `no config provided by user`) — y el login con la contraseña
   generada por `openssl passwd -6` funcionó de punta a punta.
7. **Hallazgo central, contraste directo con Silverblue**:
   `rpm-ostree status` muestra el despliegue como
   `ostree-image-signed:docker://quay.io/fedora/fedora-coreos:stable`
   — CoreOS distribuye su imagen base como una **imagen de contenedor
   OCI real** desde Quay.io, no como un commit OSTree "puro" servido
   desde un repo tradicional (Silverblue mostraba
   `fedora:fedora/44/x86_64/silverblue`). Es el modelo de "contenedor
   nativo" (`bootc`) que ya se había insinuado en el módulo 30
   (feature `container` de `rpm-ostree --version`).
8. **`AutomaticUpdatesDriver: Zincati`, activo y sondeando
   periódicamente** — CoreOS trae actualizaciones automáticas
   nativas por defecto, algo que Silverblue no hace (ahí las
   actualizaciones son manuales, con `rpm-ostree upgrade`) — coherente
   con el propósito de CoreOS como host desatendido para contenedores.

## Evidencias

### Butane → Ignition (100% local, sin descargar nada de Fedora)

**01 — `config.bu` escrito vía heredoc, con hash de contraseña de placeholder**

![config bu escrito heredoc placeholder](evidencias/01-config-bu-escrito-heredoc-placeholder.png)

**02 — `openssl passwd -6`: hash real generado**

![openssl passwd 6 hash generado](evidencias/02-openssl-passwd-6-hash-generado.png)

**03 — `sed` reemplaza el placeholder por el hash real**

![sed reemplaza placeholder hash real](evidencias/03-sed-reemplaza-placeholder-hash-real.png)

**04 — Hallazgo real: `podman` no está instalado en el host Ubuntu**

![podman no instalado host ubuntu](evidencias/04-podman-no-instalado-host-ubuntu.png)

**05 — `podman run quay.io/coreos/butane` compila `config.bu` a `config.ign` (JSON real)**

![podman butane compila config ign json](evidencias/05-podman-butane-compila-config-ign-json.png)

### Creación de la VM

**06 — Asistente "Nueva máquina virtual" vacío**

![Nueva vm asistente vacío](evidencias/06-nueva-vm-asistente-vacio.png)

**07 — Nombre `Fedora_CoreOS`, ISO seleccionada, `OS Distribution` detectado como "Fedora" (contraste con el módulo 31)**

![Nombre Fedora CoreOS iso OS Distribution Fedora](evidencias/07-nombre-fedora-coreos-iso-os-distribution-fedora.png)

**08 — Base Memory ajustada a 2048 MB**

![Hardware base memory 2048mb](evidencias/08-hardware-base-memory-2048mb.png)

**09 — Disco virtual de 20,54 GB, VDI dinámico**

![Disco virtual 20 54gb vdi](evidencias/09-disco-virtual-20-54gb-vdi.png)

**10 — VM creada, detalles: ISO de 1 GB adjunta**

![VM creada detalles iso 1gb adjunta](evidencias/10-vm-creada-detalles-iso-1gb-adjunta.png)

### Arranque del Live y provisión de red

**11 — Menú de arranque minimalista, "Automatic boot in 3 seconds"**

![Menú arranque minimalista live 3 segundos](evidencias/11-menu-arranque-minimalista-live-3-segundos.png)

**12 — Arranque de systemd: `docker.socket`, generación de host keys SSH**

![Arranque systemd docker socket ssh keygen](evidencias/12-arranque-systemd-docker-socket-ssh-keygen.png)

**13 — Auto-login como `core`, MOTD con el comando de ejemplo de `coreos-installer`**

![Auto login core motd comando coreos installer](evidencias/13-auto-login-core-motd-comando-coreos-installer.png)

**14 — Host: `python3 -m http.server 8000` corriendo**

![Host python3 http server 8000](evidencias/14-host-python3-http-server-8000.png)

### Incidente real de red no resuelto del todo (sorteado con heredoc)

**15 — `curl -s` sin salida visible**

![curl s config ign sin salida](evidencias/15-curl-s-config-ign-sin-salida.png)

**16 — `curl -v`: "Network is unreachable" (primer intento)**

![curl v network is unreachable primer intento](evidencias/16-curl-v-network-is-unreachable-primer-intento.png)

**17 — `ip addr`/`ip route` en la VM: configuración de red correcta**

![ip addr ip route configuración red correcta](evidencias/17-ip-addr-ip-route-configuracion-red-correcta.png)

**18 — Typo real: punto extra en la URL → "Could not resolve host"**

![typo punto extra could not resolve host](evidencias/18-typo-punto-extra-could-not-resolve-host.png)

**19 — URL corregida, sigue "Network is unreachable"**

![curl url corregida sigue network unreachable](evidencias/19-curl-url-corregida-sigue-network-unreachable.png)

**20 — `ping -c 3 10.0.2.2`: exitoso, 0% de pérdida**

![ping 10 0 2 2 exitoso 0 pérdida](evidencias/20-ping-10-0-2-2-exitoso-0-perdida.png)

**21 — `firewalld` no existe en el Live, `nft list ruleset` vacío**

![firewalld no encontrado nft ruleset vacío](evidencias/21-firewalld-no-encontrado-nft-ruleset-vacio.png)

**22 — Host: `ufw allow from 10.0.2.0/24 to any port 8000`**

![ufw allow 10 0 2 0 24 puerto 8000 host](evidencias/22-ufw-allow-10-0-2-0-24-puerto-8000-host.png)

**23 — Tras la regla de `ufw`, sigue "Network is unreachable"**

![curl tras regla ufw sigue unreachable](evidencias/23-curl-tras-regla-ufw-sigue-unreachable.png)

**24 — Mismo error probando con datos móviles del host**

![curl mismo error red tras mis datos](evidencias/24-curl-mismo-error-red-tras-mis-datos.png)

**25 — `curl -4` (IPv4 forzado): mismo error**

![curl 4 ipv4 forzado mismo error](evidencias/25-curl-4-ipv4-forzado-mismo-error.png)

**26 — Host: `ss -tlnp` confirma el servidor escuchando en `0.0.0.0:8000`**

![host ss tlnp confirma servidor 0 0 0 0 8000](evidencias/26-host-ss-tlnp-confirma-servidor-0-0-0-0-8000.png)

**27 — `nc: command not found` — el Live de CoreOS ni siquiera trae netcat**

![nc command not found vm minimalista](evidencias/27-nc-command-not-found-vm-minimalista.png)

**28 — Cambio de estrategia: `cat config.ign` en el host para copiar el contenido**

![host cat config ign contenido para copiar](evidencias/28-host-cat-config-ign-contenido-para-copiar.png)

**29 — `/tmp/config.ign` pegado vía heredoc en la VM, contenido confirmado idéntico**

![vm heredoc tmp config ign pegado confirmado](evidencias/29-vm-heredoc-tmp-config-ign-pegado-confirmado.png)

### Instalación real y primer arranque aprovisionado

**30 — `coreos-installer install /dev/sda --ignition-file /tmp/config.ign`: completado en segundos**

![coreos installer install dev sda completado](evidencias/30-coreos-installer-install-dev-sda-completado.png)

**31 — Configuración → Almacenamiento: la ISO seguía adjunta tras la instalación**

![configuración almacenamiento iso adjunta post install](evidencias/31-configuracion-almacenamiento-iso-adjunta-post-install.png)

**32 — Confirmar "Remove attachment" (eliminar la ISO de la unidad óptica)**

![confirmar eliminar unidad óptica](evidencias/32-confirmar-eliminar-unidad-optica.png)

**33 — Controlador IDE vacío tras quitar la ISO**

![controlador ide vacío tras quitar iso](evidencias/33-controlador-ide-vacio-tras-quitar-iso.png)

**34 — Primer arranque real: "Ignition: user-provided config was applied"**

![primer arranque real ignition user provided config applied](evidencias/34-primer-arranque-real-ignition-user-provided-config-applied.png)

**35 — Login como `core` con la contraseña real generada, exitoso**

![login core password real exitoso](evidencias/35-login-core-password-real-exitoso.png)

**36 — `rpm-ostree status` (imagen de `quay.io`, `AutomaticUpdatesDriver: Zincati`) + `/etc/os-release`**

![rpm-ostree status imagen quay io zincati os release](evidencias/36-rpm-ostree-status-imagen-quay-io-zincati-os-release.png)

## Pendientes
