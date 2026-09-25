# Módulo 01 — Instalación de Fedora Server con Anaconda

- Estado: Completado
- Fecha: 2026-09-25
- Versión de Fedora: Fedora Linux 44 (Server Edition), kernel `6.19.10-300.fc44.x86_64`
- Objetivo diferencial frente a Arch: en Arch la instalación es manual
  (particionar, `pacstrap`, `chroot`, configurar todo a mano). Fedora usa
  **Anaconda**, un instalador guiado que toma decisiones de particionado,
  red y usuarios en una sola sesión, con validación en tiempo real de qué
  falta completar antes de permitir instalar.

## Concepto diferencial

Anaconda no deja avanzar a "Comenzar la instalación" mientras haya
elementos marcados en naranja en el resumen. Compara esto con Arch, donde
un `pacstrap` mal hecho recién falla (o deja un sistema roto) al reiniciar.
Anaconda mueve esa validación al momento de configurar, no al final.

También es relevante: **Fedora Server deshabilita la cuenta root por
defecto** en cuanto detecta que se creó un usuario con privilegios de
administrador (membresía al grupo `wheel`). No hay que hacer nada extra
para lograr esto — es el comportamiento esperado y la práctica recomendada
en Fedora/RHEL modernos (sudo desde un usuario normal, nunca login directo
como root).

## Práctica guiada

1. Arranque desde GRUB de la ISO → **"Test this media & install Fedora 44"**
   (valida la integridad de la ISO antes de instalar, ya habíamos verificado
   el SHA256 manualmente también).
2. Idioma: Español (Colombia).
3. Destino de instalación: disco de 45 GB (`ATA VBOX HARDDISK`),
   particionado automático, sin cifrado.
4. Red y nombre de equipo: NAT conectado por `enp0s3` (autodetectado).
5. Creación de usuario:
   - Usuario: `fbeleno`
   - "Añadir privilegios administrativos a esta cuenta de usuario
     (membresía al grupo wheel)" — marcado.
   - Contraseña con fortaleza "Buena".
6. Cuenta de root: sin tocar — quedó inhabilitada automáticamente al crear
   el usuario administrador (ver Concepto diferencial).
7. "Comenzar la instalación" → progreso hasta "¡Completado!".
8. Antes de reiniciar: se quitó la ISO de la unidad óptica virtual
   (Dispositivos → Unidad óptica → Eliminar disco) para no volver a
   arrancar el instalador.
9. Primer arranque real: login en modo texto (`tty1`), sin entorno gráfico.

```bash
# Verificaciones post-instalación, ya dentro de la VM:
sudo -v
cat /etc/fedora-release
sudo hostnamectl set-hostname fedora-server.lab
hostnamectl
```

## Hallazgos reales

1. **El hostname no quedó aplicado por Anaconda.** Aunque el plan original
   era configurarlo en la pantalla "Red y nombre de equipo" durante la
   instalación, ese campo no es obligatorio (no aparece marcado en
   naranja) y se saltó sin querer. El sistema arrancó como `localhost`.
   Se corrigió post-instalación con `hostnamectl set-hostname` — confirmado
   con `hostnamectl` mostrando `Static hostname: fedora-server.lab`.
2. **Cockpit está activo desde el primer arranque**, sin configurarlo: el
   propio prompt de login en tty1 ya muestra
   `Web console: https://localhost:9090 or https://10.0.2.15:9090/`.
   Esto no pasa en una instalación mínima de Arch — es una decisión de
   diseño explícita de Fedora Server.
3. **El acceso a Cockpit desde el navegador del host falló al principio**
   (`https://10.0.2.15:9090` se quedaba cargando indefinidamente). Causa
   raíz: en modo NAT, el host no puede iniciar conexiones hacia la IP
   interna de la VM (`10.0.2.x`) sin una regla de reenvío de puertos
   explícita — el NAT de VirtualBox solo permite tráfico saliente desde la
   VM. Se resolvió con:
   ```bash
   VBoxManage controlvm "Fedora_Server" natpf1 "cockpit,tcp,,9090,,9090"
   ```
   y accediendo luego a `https://localhost:9090` desde el host. Este
   hallazgo se documenta acá pero es también la base teórica del módulo 06
   (administración remota) — se retomará ahí con más profundidad (SSH,
   firewalld del lado del guest).

## Validación

- Login exitoso por consola: `fbeleno@localhost:~$` → luego
  `fbeleno@fedora-server.lab` tras corregir el hostname.
- `sudo -v` funciona sin error (confirma membresía en `wheel`).
- `cat /etc/fedora-release` → `Fedora release 44 (Forty Four)`.
- Cockpit accesible desde el navegador del host vía
  `https://localhost:9090` (con reenvío de puertos), certificado
  autofirmado aceptado manualmente, panel "Visión global" muestra
  `fedora-server.lab ejecutando Fedora Linux 44 (Server Edition)`,
  CPU/memoria en tiempo real, e inicio de sesión registrado desde
  `::ffff:10.0.2.2` (la IP del host dentro del NAT).
- Snapshot de VirtualBox tomado como punto de recuperación:
  ```bash
  VBoxManage snapshot "Fedora_Server" take "00-instalacion-base" \
    --description "Fedora Server 44 recién instalado, usuario fbeleno con sudo, hostname fedora-server.lab"
  ```
  UUID: `56ae7042-baa1-46f7-ac89-a12004373940`.

## Incidente resuelto: `mcelog.service` falló al iniciar

Cockpit reportó **"1 servicio ha fallado"** en el panel "Salud" nada más
loguearse por primera vez. Diagnóstico real, no forzado:

- **Síntoma:** panel "Salud" de Cockpit → "1 servicio ha fallado".
- **Observación:** `systemctl status mcelog` → `Active: failed (Result:
  exit-code)`.
- **Logs:**
  ```
  mcelog: ERROR: AMD Processor family 25: mcelog does not support this processor.
  Please use the edac_mce_amd module instead.
  CPU is unsupported
  ```
- **Hipótesis:** `mcelog` (Machine Check Exception Logging Daemon) es una
  herramienta de espacio de usuario pensada originalmente para CPUs Intel;
  el mensaje indica explícitamente que no soporta AMD familia 25 (Zen 3/4
  — la CPU física del host, expuesta tal cual al guest).
- **Prueba:** `lsmod | grep edac_mce_amd` → sin salida. El módulo del
  kernel que reemplaza a `mcelog` en AMD tampoco está cargado, pero eso es
  esperable: ese módulo lee el controlador de memoria físico para detectar
  errores de hardware (RAM/caché defectuosos), algo que VirtualBox no
  expone al guest de forma útil. No hay nada que monitorear en una VM.
- **Causa raíz confirmada:** incompatibilidad conocida de `mcelog` con
  CPUs AMD modernas — no es un fallo de la instalación ni de la VM.
- **Solución aplicada:**
  ```bash
  sudo systemctl disable --now mcelog
  sudo systemctl mask mcelog
  sudo systemctl reset-failed mcelog
  ```
- **Validación:** `systemctl status mcelog` → `Loaded: masked`,
  `Active: inactive (dead)`. Panel "Salud" de Cockpit recargado → ya no
  aparece ningún servicio fallado, solo el aviso normal de actualizaciones
  de seguridad disponibles.
- **Impacto y riesgo:** ninguno — `mcelog` no cumplía ninguna función en
  este entorno virtualizado; enmascararlo no reduce la seguridad ni la
  observabilidad real de la VM.
- **Cómo evitar recurrencia:** en futuras VMs de Fedora sobre hardware
  AMD (host o físico), verificar `systemctl status mcelog` temprano y
  enmascararlo de una vez si el host es AMD familia ≥ 17h (Zen o
  posterior), en vez de esperar a que Cockpit lo reporte.

También se habilitó el **"Acceso administrativo"** dentro de Cockpit
(estaba en modo de solo lectura al primer login) — necesario para ver y
gestionar servicios desde la interfaz web.

## Reto autónomo

Pendiente: repetir la instalación completa (Anaconda) pero con
particionado **personalizado** (LVM manual) en vez de automático, y
comparar el resultado de `lsblk`/`df -h` contra esta instalación.

## Evidencias

**01 — GRUB de la ISO: "Test this media & install Fedora 44"**
Arranque desde la ISO verificada; opción elegida por defecto, que valida integridad antes de instalar.

![GRUB, test media & install](evidencias/01-grub-test-media-install-fedora-44.png)

**02 — Anaconda: bienvenida y selección de idioma**
Pantalla inicial de Anaconda con "Español (Colombia)" preseleccionado.

![Bienvenida de Anaconda, selección de idioma](evidencias/02-anaconda-bienvenida-seleccion-idioma.png)

**03 — Resumen de instalación con pendientes en naranja**
Destino de instalación, red/hostname, cuenta root y creación de usuario marcados como incompletos antes de poder instalar.

![Resumen de instalación con elementos pendientes](evidencias/03-resumen-instalacion-pendientes-en-naranja.png)

**04 — Destino de instalación sin disco seleccionado**
Anaconda bloquea el avance: "No se seleccionó un disco".

![Destino de instalación sin disco](evidencias/04-destino-instalacion-sin-disco-seleccionado.png)

**05 — Disco de 45 GiB seleccionado, particionado automático**
Disco `ATA VBOX HARDDISK` marcado, configuración de almacenamiento automática, sin cifrado.

![Disco de 45 GiB seleccionado](evidencias/05-destino-instalacion-disco-45gib-seleccionado.png)

**06 — Resumen tras seleccionar el disco**
"Destino de la instalación" resuelto; solo faltan los ajustes de usuario.

![Resumen tras seleccionar disco](evidencias/06-resumen-tras-seleccionar-disco.png)

**07 — Creación de usuario `fbeleno` con privilegios de administrador**
Usuario con membresía al grupo `wheel` marcada, contraseña con fortaleza "Buena".

![Creación de usuario fbeleno](evidencias/07-creacion-usuario-fbeleno-con-wheel.png)

**08 — Resumen listo: cuenta root inhabilitada sin advertencia**
Al crear el usuario administrador, Anaconda acepta la cuenta root inhabilitada como válida (ya no aparece en naranja).

![Resumen listo para instalar](evidencias/08-resumen-listo-cuenta-root-inhabilitada.png)

**09 — Progreso de instalación: "¡Completado!"**
Barra de progreso al 100%, listo para reiniciar.

![Instalación completada](evidencias/09-progreso-instalacion-completado.png)

**10 — Primer arranque real: kernel `6.19.10-300.fc44.x86_64`**
Arranca desde el disco duro (no vuelve al instalador), confirmando que la instalación quedó persistida.

![Arranque del kernel instalado](evidencias/10-primer-arranque-kernel-fc44.png)

**11 — Prompt de login con Cockpit ya anunciado**
`Web console: https://localhost:9090 or https://10.0.2.15:9090/` aparece sin haber configurado nada — Cockpit viene activo por defecto en Fedora Server.

![Prompt de login con URL de Cockpit](evidencias/11-prompt-login-cockpit-anunciado.png)

**12 — Primer login exitoso por consola**
`fbeleno@localhost:~$` — shell obtenida con la contraseña configurada en Anaconda.

![Primer login por consola](evidencias/12-primer-login-shell-localhost.png)

**13 — Verificación de sudo, versión de Fedora y hostname**
`sudo -v` sin error, `Fedora release 44 (Forty Four)`, y `hostnamectl` confirmando `Static hostname: fedora-server.lab` tras corregirlo manualmente.

![Verificaciones post-instalación](evidencias/13-verificacion-sudo-fedora-release-hostname.png)

**14 — Snapshot de VirtualBox tomado desde el host**
Punto de recuperación `00-instalacion-base` creado por línea de comandos, con descripción del estado del sistema.

![Snapshot tomado desde el host](evidencias/14-snapshot-virtualbox-tomado-desde-host.png)

**15 — Intento fallido de acceder a Cockpit (sin `https://`)**
El navegador interpretó `10.0.2.15:9090` como búsqueda en vez de URL — pantalla de Google en vez de Cockpit.

![Intento fallido sin protocolo](evidencias/15-intento-fallido-cockpit-sin-protocolo-https.png)

**16 — Reenvío de puertos aplicado (`natpf1`)**
Comando `VBoxManage controlvm "Fedora_Server" natpf1 "cockpit,tcp,,9090,,9090"` ejecutado desde el host, tras confirmar que el NAT bloqueaba conexiones entrantes hacia `10.0.2.15`.

![Reenvío de puertos natpf1](evidencias/16-reenvio-puertos-natpf1-cockpit.png)

**17 — Advertencia de certificado autofirmado**
`NET::ERR_CERT_AUTHORITY_INVALID` al entrar a `https://localhost:9090` — esperado, Cockpit usa un certificado autofirmado por defecto.

![Advertencia de certificado](evidencias/17-advertencia-certificado-autofirmado.png)

**18 — Pantalla de login de Cockpit**
"Servidor: fedora-server.lab" confirma que el hostname corregido se propagó correctamente a Cockpit.

![Login de Cockpit](evidencias/18-cockpit-pantalla-login-fedora-server-lab.png)

**19 — Cockpit, Visión global: "1 servicio ha fallado"**
Primer login en Cockpit, en modo "Acceso limitado", con la alerta de salud que dispara el diagnóstico de `mcelog`.

![Cockpit con servicio fallado](evidencias/19-cockpit-vision-global-servicio-fallado.png)

**20 — Cockpit pide contraseña para elevar a acceso administrativo**
Diálogo "Cambiar a acceso administrativo" solicitando la contraseña de `fbeleno` — el equivalente en la interfaz web a escribir la contraseña de `sudo`. Esta captura no llegó por chat; apareció al curar `img/` contra los archivos reales con timestamp y se agregó recién en la revisión de evidencias.

![Cockpit solicita contraseña de acceso administrativo](evidencias/20-cockpit-solicitud-contrasena-acceso-administrativo.png)

**21 — Acceso administrativo habilitado en Cockpit**
Ya no aparece "Acceso limitado" — Cockpit en modo completo, con CPU/memoria en tiempo real.

![Acceso administrativo habilitado](evidencias/21-cockpit-acceso-administrativo-habilitado.png)

**22 — `mcelog` marcado como "Falló al iniciar" en la lista de Servicios**
Vista de Cockpit confirmando cuál es el servicio problemático.

![mcelog falló al iniciar](evidencias/22-servicios-mcelog-fallo-al-iniciar.png)

**23 — Diagnóstico por terminal: CPU AMD familia 25 no soportada**
`systemctl status mcelog` y `journalctl -u mcelog` muestran el mensaje exacto: `mcelog does not support this processor`.

![Diagnóstico mcelog CPU AMD](evidencias/23-diagnostico-mcelog-cpu-amd-no-soportada.png)

**24 — `mcelog` deshabilitado y enmascarado**
`systemctl disable --now mcelog` + `systemctl mask mcelog` aplicados; el estado aún mostraba "failed" residual de antes del enmascarado.

![mcelog deshabilitado y enmascarado](evidencias/24-mcelog-deshabilitado-y-enmascarado.png)

**25 — Estado limpio tras `reset-failed`**
`Loaded: masked`, `Active: inactive (dead)` — sin rastro del fallo anterior.

![mcelog inactivo tras reset-failed](evidencias/25-mcelog-reset-failed-estado-inactivo.png)

**26 — Lista de Servicios en Cockpit sin alertas**
El ícono de "Servicios" en el menú lateral ya no tiene el punto rojo de alerta.

![Servicios sin alertas](evidencias/26-cockpit-servicios-sin-alertas.png)

**27 — Validación final: panel de Salud limpio**
Solo queda el aviso normal de "Actualizaciones de seguridad disponibles" — ningún servicio fallado. Cierra el incidente y el módulo.

![Validación final, salud limpia](evidencias/27-cockpit-salud-limpia-validacion-final.png)

## Nota sobre curación de evidencias

Al revisar `img/` contra los archivos reales con timestamp (`Captura desde
2026-09-25 HH-MM-SS.png`), aparecieron 2 capturas que nunca llegaron por
chat:

- `10-18-55` — descartada: contenido idéntico al de la captura 10
  (`10-18-59`, 4 segundos después), mismo mensaje de arranque de GRUB, sin
  cambios más allá del parpadeo del cursor.
- `10-45-51` — **no era duplicado**, es la captura 20 de esta lista (el
  diálogo de contraseña de Cockpit) — un paso real que se había saltado
  al curar solo por el orden del chat.
