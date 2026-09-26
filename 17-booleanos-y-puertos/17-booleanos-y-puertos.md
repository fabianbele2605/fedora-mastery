# Módulo 17 — Booleanos y puertos: servicio web en puerto no estándar

- Estado: En curso
- Fecha: 2026-09-26
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: publicar un servicio en un puerto
  no estándar sin desactivar `enforcing`, resolviendo la denegación con
  `semanage port` (la vía mínima y correcta) en vez de `audit2allow`
  (que abriría permisos genéricos de más). Cierra el **Proyecto 2** de
  la Fase 3.

## Concepto diferencial

Un puerto no estándar necesita permiso en **dos capas independientes**:
`firewalld` (si el puerto está permitido a nivel de red, módulo 06) y
`SELinux` (si el tipo de puerto está etiquetado para el dominio del
servicio, módulo 15). Abrir una sin la otra no alcanza — hay que
diagnosticar cuál de las dos está bloqueando en cada caso.

## Práctica guiada

```bash
sudo dnf install -y httpd
echo "<h1>fedora-mastery módulo 17</h1>" | sudo tee /var/www/html/index.html
sudo sed -i 's/^Listen 80/Listen 8585/' /etc/httpd/conf/httpd.conf
sudo systemctl start httpd
sudo systemctl status httpd
```

Luego diagnóstico (SELinux con `ausearch`/`sealert`, firewalld con
`firewall-cmd`), reparación con `semanage port -a -t http_port_t -p tcp
8585` y `firewall-cmd --add-port`, y validación con `curl` real.

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
