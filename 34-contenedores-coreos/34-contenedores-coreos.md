# Módulo 34 — Contenedores en CoreOS

- Estado: En curso
- Fecha: 2026-09-29
- Versión de Fedora: Fedora CoreOS 44.20260829.3.1
- Objetivo diferencial frente a Fedora Server (módulo 21): en Server,
  Podman se instala y se lanza a mano. En CoreOS, el patrón esperado
  es declarar el contenedor **como parte del aprovisionamiento**
  (Ignition genera una unidad `systemd` que arranca el contenedor),
  coherente con su propósito de host de contenedores desatendido.

## Concepto diferencial

Ignition solo corre en el **primer arranque** del disco. No hay forma
de "reaplicar" un Butane nuevo sobre un nodo ya aprovisionado sin
reinstalarlo — el patrón real de CoreOS en producción es **infra
inmutable**: para cambiar la configuración declarativa se aprovisiona
un nodo nuevo, no se edita el existente a mano.

## Práctica guiada

1. Confirmar Podman disponible (ya viene en la imagen base) y correr
   un contenedor ad-hoc, igual que en el módulo 21:
   ```bash
   podman run -d --name web-coreos -p 8080:80 docker.io/library/httpd:alpine
   podman ps
   curl -s http://localhost:8080 | head -5
   ```
2. Crear manualmente la unidad `systemd` que Ignition habría generado
   (para exponer el mecanismo real, con la limitación anotada arriba):
   ```ini
   # /etc/systemd/system/web-coreos.service
   [Unit]
   Description=Contenedor web declarativo
   After=network-online.target

   [Service]
   ExecStartPre=-/usr/bin/podman rm -f web-coreos
   ExecStart=/usr/bin/podman run --name web-coreos -p 8080:80 docker.io/library/httpd:alpine
   ExecStop=/usr/bin/podman stop -t 10 web-coreos

   [Install]
   WantedBy=multi-user.target
   ```
3. Habilitar y arrancar la unidad, confirmar que sobrevive un
   reinicio:
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable --now web-coreos.service
   systemctl reboot
   ```
4. Tras el reinicio, validar que el contenedor arrancó solo, sin
   intervención manual:
   ```bash
   systemctl status web-coreos.service
   podman ps
   curl -s http://localhost:8080 | head -5
   ```

## Hallazgos reales

_Pendiente — se completa durante la práctica guiada._

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
