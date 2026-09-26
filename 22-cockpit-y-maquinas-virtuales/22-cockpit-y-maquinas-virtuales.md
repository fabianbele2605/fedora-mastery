# Módulo 22 — Cockpit y máquinas virtuales dentro de VirtualBox

- Estado: En curso
- Fecha: 2026-09-26
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: probar un límite real de la
  virtualización anidada (nested virtualization) — capacidad
  específica que hay que habilitar en el hipervisor externo, no algo
  que dependa del sistema operativo huésped.

## Concepto diferencial

Para que la VM de Fedora Server pueda crear **sus propias VMs** (vía
`libvirt`/KVM y `cockpit-machines`), la CPU física necesita
virtualización anidada habilitada — y eso depende de un ajuste
específico de VirtualBox (`nested-hw-virt`), desactivado por defecto.

## Verificación previa (antes de instalar nada)

```bash
# Desde el host
VBoxManage showvminfo "Fedora_Server" --machinereadable | grep -i "nested\|paravirt"
cat /sys/module/kvm_amd/parameters/nested
egrep -c 'svm' /proc/cpuinfo
```

**Resultado real**: `nested-hw-virt="off"` en la VM, pero el host sí
soporta SVM anidado (`nested=1`, 12 cores con flag `svm`). Se apagó la
VM, se habilitó `nested-hw-virt on`, y se reinició.

## Práctica guiada

```bash
sudo dnf install -y virt-install libvirt cockpit-machines
egrep -c 'svm|vmx' /proc/cpuinfo
systemctl status libvirtd
```

Luego, desde Cockpit → "Máquinas virtuales", intentar crear una VM
mínima y documentar el resultado real (éxito o límite encontrado).

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
