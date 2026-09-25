# Módulo 05 — Almacenamiento desde Cockpit

- Estado: Completado
- Fecha: 2026-09-25
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: comparar el flujo gráfico de
  Cockpit (que en el fondo usa `udisks2`) contra los comandos LVM/XFS
  equivalentes, y retomar el pendiente del módulo 02 (27.41G libres en la
  VG `fedora` sin asignar).

## Concepto diferencial

- **XFS solo crece, nunca se encoge** (`xfs_growfs`, sin `xfs_shrinkfs`)
  — limitación real del filesystem por defecto de Fedora Server, muy
  distinta a Btrfs (ya visto en `arch-linux-desktop`, módulos 21-24),
  que permite crecer y encoger subvolúmenes libremente.
- El panel "Almacenamiento" de Cockpit es una capa visual sobre
  **`udisks2`** — cada acción de la UI dispara `lvextend`, `resize2fs`/
  `xfs_growfs`, `mkfs`, etc. por detrás.
- Segundo disco virtual de 10G agregado a la VM desde el host (en
  caliente, vía `VBoxManage storageattach` sobre el controlador SATA)
  para practicar crear un PV/VG nuevo desde cero.

## Práctica guiada

```bash
lsblk
sudo vgs
sudo lvs
df -hT /

# Extender la LV existente con el espacio libre de la VG
sudo lvextend -l +100%FREE /dev/fedora/root
sudo xfs_growfs /
df -hT /
```

Luego, desde Cockpit → "Almacenamiento": crear un VG nuevo (`datos`) sobre
`sdb`, una LV (`compartido`) usando todo el espacio, formatearla en XFS
con punto de montaje `/data`, y verificar por CLI:

```bash
lsblk
df -hT /data
cat /etc/fstab | tail -3
```

## Hallazgos reales

1. **Hot-plug de SATA funcionó sin configurarlo explícitamente**: el
   segundo disco (`sdb`, 10G) agregado en caliente desde el host
   (`VBoxManage storageattach`) apareció en la VM sin reiniciar ni marcar
   ningún flag de hotplug.
2. **Extensión de la LV existente, sin desmontar nada**:
   `lvextend -l +100%FREE /dev/fedora/root` (15 GiB → 42.41 GiB) +
   `xfs_growfs /` en caliente. `df -hT /` pasó de 15G/19% a 43G/9%.
   Confirma que **XFS solo puede crecer, nunca encogerse** — no existe
   `xfs_shrinkfs`.
3. **El diálogo "Formatear" de un disco individual en Cockpit NO ofrece
   LVM2 como tipo** (solo XFS/EXT4/VFAT/NTFS/Swap) — crear un grupo de
   volúmenes es un flujo aparte, desde el menú ☰ de la vista principal de
   Almacenamiento ("Crear un grupo de volúmenes LVM2"). También apareció
   ahí la opción de **Stratis** (otro gestor de volúmenes nativo de
   Fedora, alternativa a LVM+XFS) — mencionado, no probado en este
   módulo.
4. **El volumen lógico sí permite "Encogimiento" y "Crecer"** desde
   Cockpit (a diferencia del filesystem XFS que solo crece) — la
   limitación de "solo crecer" es del filesystem, no de LVM.
5. **`x-parent=<UUID>` en `/etc/fstab`**: Cockpit agregó automáticamente
   esta opción no estándar (específica de `udisks2`) para garantizar que
   el VG esté activo antes de montar `/data` en el arranque — algo que
   una entrada de `fstab` escrita a mano normalmente no incluiría.
6. **`sdb` quedó nombrado `datos-compartido`** en device-mapper
   (`VG-LV` unidos con guion) — confirma la convención de nombres real de
   LVM, igual que `fedora-root` para la raíz.

## Evidencias

**01 — Estado inicial: `sdb` (10G) ya detectado en caliente**
`lsblk`, `vgs`, `lvs`, `df -hT /` antes de tocar nada — confirma que el disco agregado en caliente desde el host ya aparece sin reiniciar.

![Estado inicial con sdb detectado](evidencias/01-lsblk-vgs-lvs-df-estado-inicial-con-sdb.png)

**02 — Extensión de `fedora/root`: 15 GiB → 42.41 GiB, `xfs_growfs` en caliente**
`lvextend -l +100%FREE` + `xfs_growfs /`, sin desmontar. `df -hT /` pasa de 15G/19% a 43G/9%.

![lvextend y xfs_growfs, resultado](evidencias/02-lvextend-xfs-growfs-resultado.png)

**03 — Cockpit "Almacenamiento": `sdb` aparece como "Datos sin formato" (10.7 GB)**
También muestra el error real de `udisksd` al no poder cargar el módulo `btrfs`.

![sdb datos sin formato en Cockpit](evidencias/03-cockpit-almacenamiento-sdb-datos-sin-formato.png)

**04 — Causa raíz confirmada: `udisks2-btrfs` no está instalado**
Benigno — no hay ningún archivo roto, el paquete completo no existe en el sistema.

![rpm -q udisks2-btrfs no instalado](evidencias/04-rpm-q-udisks2-btrfs-no-instalado.png)

**05 — Detalle de `sdb`: SMART "El disco está OK", menú "Crear una tabla de particiones"**
Este menú es del disco completo, no el que necesitábamos para LVM.

![Detalle sdb, SMART y menú de tabla de particiones](evidencias/05-sdb-detalle-smart-menu-tabla-particiones.png)

**06 — Menú correcto: "Datos sin formato" → "Formatear"**
El menú específico de la sección de datos sin formato, con la opción correcta resaltada.

![Menú de Datos sin formato con Formatear](evidencias/06-menu-sdb-datos-sin-formato-formatear.png)

**07-08 — Diálogo "Formatear /dev/sdb": sin opción de LVM2**
Solo XFS/EXT4/VFAT/NTFS/Swap en el desplegable "Tipo" — crear un VG es un flujo aparte.

![Formatear sdb, diálogo vacío](evidencias/07-formatear-sdb-dialogo-vacio.png)
![Tipo dropdown sin LVM2](evidencias/08-formatear-sdb-tipo-dropdown-sin-lvm2.png)

**09 — Menú ☰ de Almacenamiento: "Crear un grupo de volúmenes LVM2" y "Stratis"**
Confirma que LVM2 se crea desde acá, no desde el formato de un disco individual. También aparece Stratis, otro gestor de volúmenes nativo de Fedora.

![Menú de almacenamiento con LVM2 y Stratis](evidencias/09-menu-almacenamiento-crear-vg-lvm2-stratis.png)

**10-11 — "Crear un grupo de volúmenes": de `vgroup0` a `datos` con `sdb` marcado**

![Diálogo crear VG, nombre default](evidencias/10-crear-grupo-volumenes-dialogo-default.png)
![Diálogo crear VG, nombre datos y sdb marcado](evidencias/11-crear-grupo-volumenes-nombre-datos-sdb-marcado.png)

**12 — VG `datos` creado, `sdb` como volumen físico (0/11 GB)**

![VG datos creado con sdb como PV](evidencias/12-vg-datos-creado-sdb-como-pv.png)

**13-14 — "Crear un volumen lógico": de `lvol0` a `compartido`**

![Diálogo crear LV, nombre default](evidencias/13-crear-volumen-logico-dialogo-default.png)
![Diálogo crear LV, nombre compartido](evidencias/14-crear-volumen-logico-nombre-compartido.png)

**15-16 — LV `compartido` creado, todavía sin formato — y con opción de "Encogimiento"**
A diferencia de XFS, el volumen lógico LVM sí permite encogerse.

![LV compartido creado, datos sin formato](evidencias/15-lv-compartido-creado-datos-sin-formato.png)
![LV compartido con Encogimiento/Crecer/Desactivar](evidencias/16-lv-compartido-encogimiento-crecer-desactivar.png)

**17-19 — Formatear `compartido`: XFS, nombre `data`, punto de montaje `/data`**

![Menú Formatear resaltado](evidencias/17-menu-datos-sin-formato-formatear-resaltado.png)
![Diálogo Formatear compartido vacío](evidencias/18-formatear-compartido-dialogo-vacio.png)
![Diálogo con nombre data y /data](evidencias/19-formatear-compartido-nombre-data-punto-montaje.png)

**20 — `/data` formateado y montado (XFS, 0.24/11 GB)**

![data formateado y montado](evidencias/20-data-formateado-montado-xfs.png)

**21 — Validación final por CLI: `lsblk`, `df -hT /data`, `/etc/fstab`**
Confirma `datos-compartido` en device-mapper, montaje real, y la opción `x-parent=<UUID>` agregada automáticamente por Cockpit.

![Validación final lsblk, df y fstab](evidencias/21-lsblk-df-fstab-validacion-final.png)

## Pendientes
