# Módulo 16 — Diagnóstico de denegaciones AVC: `ausearch`, `sealert`

- Estado: En curso
- Fecha: 2026-09-26
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: diagnosticar una denegación real
  de SELinux con las herramientas de auditoría, en vez de solo mirar
  permisos DAC como sería suficiente en Arch.

## Concepto diferencial

Cada vez que un dominio intenta acceder a un tipo sin permiso, el
kernel genera un evento **AVC** (Access Vector Cache) que queda
registrado en el log de auditoría. `ausearch` permite filtrar esos
eventos; `sealert` (si `setroubleshoot` está disponible) traduce el
mensaje técnico a una explicación humana con sugerencia de solución.

## Práctica guiada

Provocar una denegación real y controlada: copiar una clave de host
SSH, forzarle un contexto de otro servicio (`httpd_sys_content_t`),
agregarla como `HostKey` extra en `sshd_config`, y reiniciar `sshd`.

```bash
sudo cp /etc/ssh/ssh_host_ed25519_key /etc/ssh/ssh_host_ed25519_key.extra
sudo chcon -t httpd_sys_content_t /etc/ssh/ssh_host_ed25519_key.extra
ls -Z /etc/ssh/ssh_host_ed25519_key.extra
echo "HostKey /etc/ssh/ssh_host_ed25519_key.extra" | sudo tee -a /etc/ssh/sshd_config
sudo systemctl restart sshd
sudo systemctl status sshd
```

Luego, diagnóstico con `ausearch` y `sealert`, y reparación (quitar la
línea `HostKey` agregada, restaurar `sshd_config`, reiniciar `sshd`).

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
