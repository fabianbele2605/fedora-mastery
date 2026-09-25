# Módulo 06 — Administración remota segura

- Estado: En curso
- Fecha: 2026-09-25
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: entender el modelo de **zonas** de
  `firewalld` (cada interfaz de red pertenece a una zona, cada zona tiene
  su propia lista de servicios permitidos) y usarlo para separar el
  acceso de administración del acceso público — no reglas planas como en
  muchas configuraciones típicas de `nftables`/`ufw`.

## Concepto diferencial

Hasta el módulo 01 accedimos a la VM únicamente por NAT + reenvío de
puertos manual (`VBoxManage ... natpf1`). Es funcional, pero no
representa cómo se administra un servidor real en una red — un
administrador real tiene una interfaz de gestión separada del tráfico
público. Hoy se agregó una segunda interfaz de red **Host-Only**
(`vboxnet0`, 192.168.59.1/24) a la VM desde el host, que permite que
Ubuntu hable directo con la VM sin reenvío de puertos.

`firewalld` organiza el filtrado en **zonas**: cada interfaz de red se
asigna a una zona (`public`, `FedoraServer`, `internal`, etc.), y cada
zona define qué servicios/puertos están permitidos. Esto permite, por
ejemplo, permitir Cockpit y SSH desde la interfaz Host-Only (zona de
confianza) pero restringirlos en la interfaz NAT (zona pública).

## Práctica guiada

```bash
nmcli device status
ip addr show
sudo firewall-cmd --get-active-zones
sudo firewall-cmd --list-all-zones | less
```

Luego, probar acceso directo desde el host (sin reenvío de puertos) a la
IP de la interfaz Host-Only:

```bash
# Desde el host (Ubuntu), una vez identificada la IP de la interfaz host-only
ssh fbeleno@<ip-hostonly>
```

Y evaluar mover Cockpit/SSH a una zona más restrictiva en la interfaz
NAT, dejando la interfaz Host-Only como la única "de confianza".

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
