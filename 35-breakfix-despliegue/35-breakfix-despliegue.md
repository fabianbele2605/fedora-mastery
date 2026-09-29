# Módulo 35 — Break & Fix de despliegue

- Estado: En curso
- Fecha: 2026-09-29
- Versión de Fedora: Fedora CoreOS 44.20260829.3.1
- Objetivo diferencial: cierra la Fase 6. A diferencia del Break &
  Fix del módulo 24 (Fedora Server, config de Apache editada a mano),
  acá el incidente ataca la propia definición declarativa del
  servicio contenerizado del módulo 34 — un cambio de "día 2" mal
  hecho sobre la unidad `systemd` que gestiona el contenedor.

## Entorno y punto de partida

VM de Fedora CoreOS del módulo 33/34, con `web-coreos.service`
gestionando un contenedor `httpd` en el puerto 8080, funcionando y
sobreviviendo reinicios.

## Incidente simulado

Se edita `/etc/systemd/system/web-coreos.service` introduciendo un
error real de "configuración de aprovisionamiento": una referencia de
imagen inexistente (simulando que alguien actualizó el Butane/unidad
apuntando a una imagen que no existe en el registro), representando
un error real de día 2 sobre infraestructura declarativa.

## Práctica guiada (diagnóstico real)

```bash
sudo sed -i 's|docker.io/library/httpd:alpine|docker.io/library/httpd:noexiste|' \
  /etc/systemd/system/web-coreos.service
sudo systemctl daemon-reload
sudo systemctl restart web-coreos.service
systemctl status web-coreos.service
journalctl -u web-coreos.service --no-pager | tail -20
curl -s http://localhost:8080
```

## Reparación

Corregir la referencia de imagen en la unidad, recargar, reiniciar el
servicio, y confirmar que vuelve a responder.

## Hallazgos reales

_Pendiente — se completa durante la práctica guiada._

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
