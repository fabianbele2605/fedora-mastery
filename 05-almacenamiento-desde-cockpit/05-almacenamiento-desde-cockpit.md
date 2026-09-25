# Módulo 05 — Almacenamiento desde Cockpit

- Estado: En curso
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

Luego, desde Cockpit → "Almacenamiento": verificar que aparece el
segundo disco (`sdb`, 10G) y crear un volumen/punto de montaje nuevo
desde la interfaz gráfica, comparando cada paso contra su equivalente
en `pvcreate`/`vgcreate`/`lvcreate`/`mkfs.xfs`/`mount`.

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
