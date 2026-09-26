# Módulo 10 — Repositorios, actualizaciones y confianza (GPG)

- Estado: En curso
- Fecha: 2026-09-25
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: entender cómo Fedora verifica la
  procedencia de cada paquete (GPG por defecto, no opcional) y cómo
  resuelve tener el mismo paquete disponible en dos repos distintos
  (`fedora` + `updates`), algo que Arch no necesita porque todo vive en
  `core`/`extra` sin esta separación.

## Concepto diferencial

- Cada repo (`/etc/yum.repos.d/*.repo`) trae `gpgcheck=1` y una ruta a
  la clave pública (`/etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-*`) — la firma
  se verifica en cada instalación por defecto.
- **Las claves GPG importadas se guardan como "paquetes" en la base de
  datos de RPM** (`gpg-pubkey-xxxxxxxx`), visibles con `rpm -q
  gpg-pubkey`. pacman en cambio usa un keyring completamente separado
  gestionado por `pacman-key`, nunca mezclado con la base de datos de
  paquetes instalados.
- Fedora separa **`fedora`** (paquetes fijos de la release) de
  **`updates`** (parches posteriores). Cuando el mismo paquete existe en
  ambos con versiones distintas, DNF resuelve a la versión más nueva
  disponible entre todos los repos habilitados — no hace falta un
  sistema de prioridades explícito para el caso común.

## Práctica guiada

```bash
cat /etc/yum.repos.d/fedora.repo
rpm -q gpg-pubkey --qf '%{name}-%{version}-%{release} --> %{summary}\n'
sudo dnf5 repoquery --repo=fedora bash
sudo dnf5 repoquery --repo=updates bash
```

## Evidencias

_Pendiente — se completa al cerrar el módulo con "verifica img"._

## Pendientes
