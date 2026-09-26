# Módulo 15 — Contextos persistentes: `ls -Z`, `semanage`, `restorecon`

- Estado: En curso
- Fecha: 2026-09-26
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: entender por qué un cambio de
  contexto con `chcon` se pierde solo, y cómo persistirlo de verdad con
  `semanage fcontext`.

## Concepto diferencial

`chcon` cambia el contexto de un archivo **en el momento**, pero no
toca la política — es un atributo extendido temporal. Si después corre
`restorecon` (manual o durante un relabel completo del sistema), el
archivo vuelve al contexto que la política dice que "debería" tener
según su ruta, perdiendo el cambio de `chcon`.

`semanage fcontext` en cambio **escribe una regla en la base de datos
de política** (asociando un patrón de ruta a un tipo). `restorecon`
respeta esa regla y la vuelve a aplicar siempre — es la forma correcta
y persistente de cambiar contextos.

También existen contextos de **puertos** (`semanage port`), relevantes
para publicar servicios en puertos no estándar (módulo 17).

## Práctica guiada

```bash
sudo mkdir -p /srv/testweb
ls -Zd /srv/testweb
sudo chcon -t httpd_sys_content_t /srv/testweb
ls -Zd /srv/testweb
sudo restorecon -v /srv/testweb
ls -Zd /srv/testweb
```

Luego, la forma persistente:

```bash
sudo semanage fcontext -a -t httpd_sys_content_t "/srv/testweb(/.*)?"
sudo restorecon -Rv /srv/testweb
ls -Zd /srv/testweb
```

Y contextos de puertos:

```bash
sudo semanage port -l | grep -i http
sudo semanage port -l | grep -i cockpit
```

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
