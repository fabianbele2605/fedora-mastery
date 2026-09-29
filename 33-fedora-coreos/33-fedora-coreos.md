# Módulo 33 — Fedora CoreOS

- Estado: En curso
- Fecha: 2026-09-28
- Versión de Fedora: Fedora CoreOS (canal estable más reciente)
- Objetivo diferencial frente a Arch y frente a Silverblue: CoreOS
  también es inmutable (basado en OSTree, como Silverblue), pero no se
  instala de forma interactiva con un asistente — se aprovisiona de
  forma **declarativa** con Ignition/Butane, pensado para hosts de
  contenedores sin intervención humana repetible.

## Concepto diferencial

- **Butane**: un YAML legible pensado para que lo escriba un humano
  (usuarios, claves SSH, archivos, unidades systemd).
- **Ignition**: el JSON de bajo nivel que Butane genera al compilarse
  — es lo que el firmware de arranque de CoreOS realmente interpreta
  la primera vez que arranca el disco.
- **No se descarga ningún JSON de Fedora** — el Ignition siempre se
  genera localmente, a partir de un Butane propio, con la herramienta
  `butane`.

## Entorno

VM temporal nueva, 2 GB de RAM (tabla de entorno del curso). A
diferencia de Silverblue/Server, el "instalador" es un binario
(`coreos-installer`) que corre dentro del propio Live ISO, no un
asistente gráfico tipo Anaconda.

## Práctica guiada

1. Descargar la ISO Live de Fedora CoreOS (canal estable).
2. Escribir `config.bu` (Butane) mínimo:
   ```yaml
   variant: fcos
   version: 1.5.0
   passwd:
     users:
       - name: core
         password_hash: <hash>
   ```
3. Compilar a Ignition:
   ```bash
   podman run --rm -i quay.io/coreos/butane:release \
     --pretty --strict < config.bu > config.ign
   ```
4. Crear la VM (2 GB RAM), arrancar desde la ISO Live.
5. Desde el Live, instalar aplicando el Ignition:
   ```bash
   sudo coreos-installer install /dev/sda --ignition-file config.ign
   ```
6. Quitar la ISO de la unidad óptica (mismo paso aprendido en el
   módulo 31) y reiniciar al sistema ya aprovisionado.
7. Iniciar sesión como `core` y validar `rpm-ostree status`.

## Hallazgos reales

_Pendiente — se completa durante la práctica guiada._

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
