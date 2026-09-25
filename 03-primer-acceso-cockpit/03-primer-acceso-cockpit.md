# Módulo 03 — Primer acceso a Cockpit

- Estado: En curso
- Fecha: 2026-09-25
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: formalizar el mecanismo detrás de
  Cockpit (activación, autenticación, certificado, firewall) — ya lo
  usamos "a lo bruto" en el módulo 01 sin entender el porqué de cada
  paso.

## Concepto diferencial

Cockpit no es un panel de administración con su propia base de usuarios
o su propio demonio siempre corriendo. Tres cosas concretas:

1. **Activación por socket**: lo que systemd habilita por defecto es
   `cockpit.socket`, no `cockpit.service`. El proceso real (`cockpit-ws`)
   recién arranca cuando llega la primera conexión a 9090 — mismo patrón
   de activación diferida que otros sockets de systemd vistos en
   `arch-linux-mastery` (módulo 09), aplicado acá a una interfaz web.
2. **Autenticación vía PAM**: Cockpit no tiene su propia tabla de
   usuarios/contraseñas. Usa el mismo stack PAM que `sshd`/`login` — la
   contraseña que cambiás con `passwd` es la misma para SSH, consola y
   Cockpit, sin sincronización manual porque en realidad es un solo
   origen de verdad.
3. **Certificado autofirmado autogenerado**: en el primer arranque,
   Cockpit genera su propio certificado TLS si no encuentra ninguno en
   `/etc/cockpit/ws-certs.d/`. La advertencia de "conexión no segura" que
   aceptamos en el módulo 01 viene de acá — no es un error, es el
   comportamiento esperado sin una PKI propia configurada.

## Práctica guiada

```bash
systemctl status cockpit.socket
sudo firewall-cmd --list-services
sudo ls -la /etc/cockpit/ws-certs.d/
ps aux | grep cockpit
```

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
