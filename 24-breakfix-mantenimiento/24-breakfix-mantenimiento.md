# Módulo 24 — Break & Fix de mantenimiento

- Estado: En curso
- Fecha: 2026-09-27
- Versión de Fedora: Fedora Linux 45 (Server Edition)
- Objetivo diferencial frente a Arch: cierra la Fase 4 con un incidente
  de mantenimiento real ligado a un mecanismo específico de RPM (la
  protección de archivos de configuración modificados), a diferencia
  de los Break & Fix anteriores (acceso, paquetes, SELinux).

## Entorno y punto de restauración

Snapshot tomado antes de este módulo (post módulo 23, Fedora 45 ya
validado): Cockpit, `httpd` en 8585, contenedor Podman rootless,
SELinux enforcing.

## Concepto diferencial

RPM protege los archivos de configuración modificados por el admin:
si un paquete trae una versión nueva de un `.conf` que el usuario ya
personalizó, no lo sobreescribe directo — deja la versión nueva como
`.rpmnew` junto al archivo real. Pero esto **no protege contra un
error humano**: un script de mantenimiento, una restauración manual
mal dirigida, o un admin apurado puede terminar copiando esa versión
"de fábrica" encima de la configuración real, perdiendo cambios
válidos. Vamos a simular exactamente ese error, no una falla de RPM.

## Incidente simulado

Se sobreescribe deliberadamente `/etc/httpd/conf/httpd.conf` con una
copia de la configuración por defecto (`Listen 80` en vez de
`Listen 8585`, sin el `ServerName` ya configurado), simulando que un
proceso de mantenimiento restauró el archivo equivocado tras una
actualización rutinaria.

## Práctica guiada (diagnóstico real)

```bash
curl -s http://localhost:8585        # ya no debería responder
systemctl status httpd
ss -tlnp | grep httpd
grep -i listen /etc/httpd/conf/httpd.conf
journalctl -u httpd --no-pager | tail -20
```

## Reparación

Corregir manualmente el `Listen` (y `ServerName` si aplica) en
`httpd.conf`, validar sintaxis, reiniciar y confirmar:

```bash
sudo httpd -t
sudo systemctl restart httpd
curl -s http://localhost:8585
sestatus
```

## Hallazgos reales

_Pendiente — se completa durante la práctica guiada._

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
