# Módulo 07 — Break & Fix de acceso (cierre de Fase 1 y Proyecto 1)

- Estado: En curso
- Fecha: 2026-09-25
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: primer Break & Fix formal del
  curso — provocar un incidente real de acceso (no simulado en el
  papel) y recuperarlo sin reinstalar, usando la consola local de
  VirtualBox como vía de recuperación fuera de banda, igual que un
  puerto serie/iDRAC en un servidor físico real.

## Entorno y punto de restauración

Snapshot `06-fin-fase1-antes-breakfix` tomado antes de este módulo
(fin de Fase 1: LVM extendido, VG `datos`/`compartido`, zonas de
firewalld separadas — `FedoraServer` en `enp0s8`, `public` en `enp0s3`).

## Cambio deliberado

Aprovechando lo aprendido en el módulo 06 sobre `connection.zone` de
NetworkManager, se mueve `enp0s8` (la interfaz host-only, hasta ahora en
zona `FedoraServer` con SSH y Cockpit permitidos) a la zona **`drop`**
— la zona más restrictiva de firewalld, que descarta todo el tráfico
entrante sin responder (ni siquiera rechaza, silencia).

```bash
# Ejecutado desde la consola local de VirtualBox (no por SSH)
sudo nmcli connection modify "Conexión cableada 2" connection.zone drop
sudo nmcli connection up "Conexión cableada 2"
```

Como `enp0s3` (NAT) está en zona `public` sin Cockpit habilitado y sin
reenvío de puertos configurado para SSH, este cambio deja la VM **sin
ningún camino de red** hacia SSH ni Cockpit desde el host.

## Síntoma

- `ssh fbeleno@192.168.59.101` desde el host: sin respuesta, timeout.
- `https://192.168.59.101:9090` desde el navegador: no carga.
- `https://localhost:9090` (ruta NAT): tampoco cargaba desde el módulo
  06 (ya esperado, sin relación con este incidente).

## Diagnóstico y recuperación

_(se completa con la práctica real: observación, registros, hipótesis,
prueba, causa raíz, solución, validación)_

## Impacto y riesgos

_(pendiente)_

## Cómo evitar recurrencia

_(pendiente)_

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
