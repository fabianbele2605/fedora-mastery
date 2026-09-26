# Módulo 21 — Cockpit y Podman: contenedores, imágenes y volúmenes

- Estado: En curso
- Fecha: 2026-09-26
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: Podman es el motor de contenedores
  por defecto en Fedora/RHEL — sin demonio central como Docker, rootless
  por defecto. Arch suele usar Docker salvo instalación manual de
  Podman aparte.

## Concepto diferencial

Podman corre contenedores **sin un demonio central** (a diferencia de
`dockerd`) y, por defecto, **rootless** — como tu usuario normal, no
como root. Una consecuencia real de esto: un contenedor rootless **no
puede bindear puertos por debajo de 1024** sin una capability/ajuste de
sysctl extra, algo que sí puede hacer un contenedor Docker corriendo
con el demonio como root.

Cockpit tiene un plugin nativo (`cockpit-podman`) que muestra los
mismos contenedores, imágenes y volúmenes que ve `podman` por CLI —
mismo patrón de correlación 1:1 ya visto con `systemd`/servicios.

## Práctica guiada

```bash
sudo dnf install -y podman cockpit-podman
podman run -d --name web-modulo21 -p 8080:80 docker.io/library/httpd:alpine
podman ps
curl -s http://localhost:8080 | head -5
```

Luego, comparar contra Cockpit → "Podman" (contenedores, imágenes).

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
