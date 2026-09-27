# fedora-mastery

Tercer curso autodidacta de Linux, tutorado por IA (Claude), sobre **Fedora**
y su ecosistema específico. Continuación directa de:

- [`arch-linux-mastery`](https://github.com/fabianbele2605/arch-linux-mastery) — sysadmin, servidores, seguridad, kernel. **Completo (30/30 módulos).**
- [`arch-linux-desktop`](https://github.com/fabianbele2605/arch-linux-desktop) — Arch como sistema de escritorio completo. **En curso (29/54 módulos), avanzando en paralelo con este.**

📖 **[Ver la guía completa del curso](fedora-mastery-guia.md)** — temario detallado, laboratorio, plantillas, evaluación y cronograma.

## Alcance

Este curso **no repite** fundamentos genéricos de Linux ya cubiertos en los
cursos de Arch: systemd, kernel, filesystems, D-Bus, redes, pacman/AUR,
firewall/hardening genérico. Se enfoca únicamente en lo que es **específico
de Fedora**:

- RPM y DNF5 frente a pacman (resolución de dependencias, repos, COPR)
- SELinux en modo *enforcing* (frente al modelo DAC tradicional visto en Arch;
  ningún curso previo cubrió AppArmor)
- Sistemas inmutables: rpm-ostree, Silverblue, CoreOS, bootc
- Fedora como plataforma de referencia para Rust (toolchain, empaquetado `.rpm`)
- Fedora Server + Cockpit (administración web)
- Kickstart y Ansible aplicados a configuración específica de Fedora

## Metodología

Igual que en los cursos anteriores:

- Cada módulo combina **teoría + práctica real + evidencias** (capturas
  curadas con descripción).
- **Break & Fix**: incidentes reales provocados y resueltos, documentados
  sin ocultar errores (typos, builds rotos, limitaciones de VM, etc.).
- Todo se ejecuta sobre **VirtualBox**, nunca hardware físico.
- Commits con `Co-Authored-By: Claude`.
- **Regla de honestidad**: si un fallo no se resuelve, se documenta el
  resultado real y la restauración usada; nunca se inventa una captura ni
  se declara una práctica superada sin evidencia.

## Entorno

Anfitrión con 8 GB de RAM → **una sola VM encendida a la vez** (salvo el
proyecto final, con dos VMs ligeras corriendo juntas).

| VM | Fases | RAM | Disco | Notas |
|---|---|---|---|---|
| Fedora Server | 1, 4 | 4 GB, 2 vCPU | 45 GB | UEFI, NAT inicial, SSH + Cockpit por HTTPS |
| Fedora Silverblue | 6 (1ª mitad) | 4 GB | 40 GB | VM temporal, Guest Additions limitadas |
| Fedora CoreOS | 6 (2ª mitad) | 2 GB | 20 GB | VM temporal, aprovisionada con Ignition/Butane |

## Progreso

**Módulo actual: 26 / 40**

### Fase 1 — Fedora Server y Cockpit (00–07)
- [x] [00. Preparar VirtualBox en Ubuntu](../00-preparacion-virtualbox/00-preparacion-virtualbox.md)
- [x] [01. Instalar Fedora Server con Anaconda](../01-instalacion-fedora-server/01-instalacion-fedora-server.md)
- [x] [02. Anatomía de Fedora Server](../02-anatomia-fedora-server/02-anatomia-fedora-server.md)
- [x] [03. Primer acceso a Cockpit](../03-primer-acceso-cockpit/03-primer-acceso-cockpit.md)
- [x] [04. Administración desde Cockpit](../04-administracion-desde-cockpit/04-administracion-desde-cockpit.md)
- [x] [05. Almacenamiento desde Cockpit](../05-almacenamiento-desde-cockpit/05-almacenamiento-desde-cockpit.md)
- [x] [06. Administración remota segura](../06-administracion-remota-segura/06-administracion-remota-segura.md)
- [x] [07. Break & Fix de acceso](../07-breakfix-acceso/07-breakfix-acceso.md) 🔧

**Proyecto 1 completado**: Fedora Server accesible desde Ubuntu, con SSH,
Cockpit, configuración de red documentada (zonas de firewalld separadas
entre NAT y host-only), instantáneas y reporte de recuperación real.

### Fase 2 — RPM, DNF5 y COPR (08–13)
- [x] [08. RPM frente a pacman](../08-rpm-frente-a-pacman/08-rpm-frente-a-pacman.md)
- [x] [09. DNF5 y transacciones](../09-dnf5-transacciones/09-dnf5-transacciones.md)
- [x] [10. Repositorios y confianza (GPG)](../10-repositorios-y-confianza-gpg/10-repositorios-y-confianza-gpg.md)
- [x] [11. COPR](../11-copr/11-copr.md)
- [x] [12. Construir un RPM](../12-construir-un-rpm/12-construir-un-rpm.md) (SPEC + rpmbuild)
- [x] [13. Break & Fix de paquetes](../13-breakfix-paquetes/13-breakfix-paquetes.md) 🔧

**Fase 2 completa**: RPM frente a pacman, DNF5 con historial y undo real,
GPG en dos capas, COPR evaluado con criterio (sin habilitar nada sin
confianza), primer RPM propio construido, y un repositorio roto
diagnosticado y reparado de punta a punta.

### Fase 3 — SELinux enforcing (14–19)
- [x] [14. Política SELinux en Fedora](../14-selinux-politica-targeted/14-selinux-politica-targeted.md)
- [x] [15. Contextos persistentes](../15-contextos-persistentes/15-contextos-persistentes.md)
- [x] [16. Diagnóstico AVC](../16-diagnostico-avc/16-diagnostico-avc.md)
- [x] [17. Booleanos y puertos](../17-booleanos-y-puertos/17-booleanos-y-puertos.md)
- [x] [18. Políticas locales y audit2allow](../18-politicas-locales-audit2allow/18-politicas-locales-audit2allow.md)
- [x] [19. Break & Fix SELinux](../19-breakfix-selinux/19-breakfix-selinux.md) 🔧

**Fase 3 completa**: dominios y tipos, contextos persistentes,
diagnóstico AVC real (con el hallazgo del daemon `setroubleshootd`
caído), servicio web en puerto no estándar con las dos capas resueltas
(SELinux + firewalld), `audit2allow` evaluado con criterio (dos módulos
rechazados con justificación), y un Break & Fix real de contenido web
recuperado sin desactivar `enforcing` en ningún momento.

**Proyecto 2 completado**: servicio web (`httpd`) publicado en el puerto
no estándar 8585, con SELinux `enforcing` activo todo el tiempo —
denegación real diagnosticada y resuelta en dos capas independientes
(`semanage port` + `firewalld`), validado con acceso real desde el host.

### Fase 4 — Fedora Server avanzado y Cockpit (20–24)
- [x] [20. Servidor web (CLI + Cockpit)](../20-servidor-web-cockpit/20-servidor-web-cockpit.md)
- [x] [21. Cockpit y Podman](../21-cockpit-y-podman/21-cockpit-y-podman.md)
- [x] [22. Cockpit y virtualización](../22-cockpit-y-maquinas-virtuales/22-cockpit-y-maquinas-virtuales.md)
- [x] [23. Actualizaciones de versión](../23-actualizacion-version-mayor/23-actualizacion-version-mayor.md)
- [x] [24. Break & Fix de mantenimiento](../24-breakfix-mantenimiento/24-breakfix-mantenimiento.md) 🔧

**Fase 4 completa**: servicio web administrado desde CLI y Cockpit,
Podman rootless con contenedores e imágenes, virtualización anidada
habilitada y validada, un upgrade mayor real (F44→F45) con estrategia
de reversión por GRUB, y un Break & Fix de mantenimiento real —
restauración accidental de `httpd.conf` diagnosticada (incluyendo dos
errores propios de comandos: bandera `-I` en vez de `-i`, y `ss` sin
`sudo`) y reparada sin recurrir a la instantánea.

**Proyecto 3 completado**: aplicación en contenedor (Podman) con
supervisión desde Cockpit, servicio web con SELinux enforcing sin
interrupciones durante un upgrade mayor y un incidente de
mantenimiento, y procedimiento de recuperación real documentado de
punta a punta.

### Fase 5 — Rust en Fedora y empaquetado RPM (25–29)
- [x] [25. Toolchain Rust (rustup vs DNF)](../25-toolchain-rust/25-toolchain-rust.md)
- [ ] 26. Cargo y bibliotecas del sistema
- [ ] 27. Utilidad Rust para Fedora Server
- [ ] 28. Empaquetado RPM del binario
- [ ] 29. Break & Fix de compilación 🔧

### Fase 6 — Silverblue, rpm-ostree y CoreOS (30–35)
- [ ] 30. OSTree y rpm-ostree
- [ ] 31. Fedora Silverblue
- [ ] 32. Rollback de despliegues
- [ ] 33. Fedora CoreOS
- [ ] 34. Contenedores en CoreOS
- [ ] 35. Break & Fix de despliegue 🔧

### Fase 7 — Automatización específica de Fedora (36–38)
- [ ] 36. Kickstart
- [ ] 37. Ansible aplicado a Fedora
- [ ] 38. Modelos de despliegue (comparación)

### Fase 8 — Proyecto final (39)
- [ ] 39. Fedora Enterprise Lab

## Estructura del repo

Mismo patrón plano que `arch-linux-mastery`/`arch-linux-desktop`: una
carpeta numerada por módulo en la raíz del repo, cada una con su `.md`
(teoría + práctica + hallazgos + checklist) y su propia `evidencias/`.
Las capturas crudas caen en `img/` (gitignored) y se curan ahí cuando el
módulo se cierra.

```
fedora-mastery/
├── docs/
│   ├── README.md              ← este archivo (tracker de progreso)
│   ├── fedora-mastery-guia.md ← guía completa del curso
│   └── tutor.txt              ← metodología de tutoría (no negociable)
├── img/                        ← capturas crudas, gitignored
├── 00-preparacion-virtualbox/
│   ├── 00-preparacion-virtualbox.md
│   └── evidencias/
├── 01-instalacion-fedora-server/
│   ├── 01-instalacion-fedora-server.md
│   └── evidencias/
└── ...                          # una carpeta NN-nombre/ por módulo
```

Un `break-and-fix.md` dentro de la carpeta del módulo cuando incluye un
incidente provocado deliberadamente (marcados con 🔧 en el temario).

## Cursos siguientes

Tras este curso, la ruta continúa con **RHEL** (suscripciones, AppStream,
Leapp, Satellite/Insights, camino a RHCSA/RHCE) y **Kali** (metapaquetes,
metodología de pentesting, laboratorio aislado), cada uno enfocado solo en
lo que es específico de esa distro.
