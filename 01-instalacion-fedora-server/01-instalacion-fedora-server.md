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

## Pendientes abiertos (se retoman en módulos siguientes)

- Cockpit reportó **"1 servicio ha fallado"** en el panel "Salud" — sin
  diagnosticar todavía. Se investiga al reanudar la sesión, antes de
  cerrar este módulo o como parte del módulo 07 (Break & Fix de acceso).
- Habilitar el "acceso administrativo" dentro de Cockpit (estaba en modo
  de solo lectura al primer login).

## Reto autónomo

Pendiente: repetir la instalación completa (Anaconda) pero con
particionado **personalizado** (LVM manual) en vez de automático, y
comparar el resultado de `lsblk`/`df -h` contra esta instalación.

## Evidencias

_Pendiente — se completa cuando el usuario confirme "verifica img"._
