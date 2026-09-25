# Fedora Mastery

> **Curso 3 de la ruta Linux** · Continuación de `arch-linux-mastery` (sysadmin, servidores, seguridad y kernel) y `arch-linux-desktop` (escritorio completo).
>
> **Especialización principal:** Fedora Server y Cockpit. **Entorno:** VirtualBox en Ubuntu. **Duración estimada:** 20–26 semanas.

## 1. Propósito y alcance

Aprender **lo específico de Fedora y su ecosistema**, sin repetir fundamentos generales ya estudiados en Arch: systemd, kernel, sistemas de archivos, D-Bus, redes o seguridad conceptual. Cada tema debe contrastar las decisiones y herramientas de Fedora con la experiencia previa en Arch.

**Competencias de salida:**
- Administrar Fedora Server por SSH y mediante Cockpit.
- Comprender y solucionar problemas de RPM, DNF5, repositorios y COPR, comparándolos con pacman y AUR.
- Mantener SELinux en modo **enforcing**, diagnosticar AVC y aplicar la corrección menos permisiva.
- Empaquetar una utilidad Rust como `.rpm`.
- Administrar despliegues con rpm-ostree y experimentar con Silverblue y Fedora CoreOS.
- Documentar actualizaciones, incidentes y recuperación reproducible.

## 2. Laboratorio

| Recurso | Configuración propuesta |
|---|---|
| Anfitrión | Ubuntu |
| Hipervisor | VirtualBox |
| VM principal | Fedora Server, nueva y exclusiva |
| CPU | 2 vCPU |
| RAM de la VM | 4 GB inicialmente, ajustable |
| Disco | 45 GB, asignación dinámica |
| Firmware | UEFI |
| Red inicial | NAT; host-only o puente según el ejercicio |
| Acceso | SSH y Cockpit por HTTPS |
| Recuperación | Instantáneas antes de prácticas de riesgo |

El equipo dispone de **4–8 GB de RAM para VMs** y **40–80 GB de espacio**. No mantener varias VMs pesadas encendidas a la vez. Usar clones vinculados o VMs temporales para Silverblue/CoreOS y limpiar imágenes y capturas innecesarias. Si solo hay 40 GB realmente libres, reducir el disco virtual principal y planificar el almacenamiento antes de instalar: **45 GB de disco virtual no equivalen a 45 GB de espacio consumido inmediatamente**, pero sí pueden alcanzarse.

**Verificaciones antes de instalar:** versión de Ubuntu y VirtualBox; virtualización activada; espacio realmente libre; suma de RAM usada por anfitrión y VM; ISO oficial y suma de verificación; acceso a red; identificación inequívoca del disco virtual.

## 3. Método de estudio

Cada módulo sigue esta secuencia:

1. **Concepto diferencial:** ¿qué hace Fedora distinto de Arch y por qué?
2. **Teoría aplicada:** arquitectura, decisiones técnicas y límites.
3. **Práctica guiada:** comandos, configuración y resultados esperados.
4. **Evidencias:** capturas, salidas relevantes y archivos de configuración, sin secretos.
5. **Break & Fix:** provocar un fallo controlado en una VM recuperable.
6. **Diagnóstico:** síntoma → observación → logs → hipótesis → prueba → causa raíz.
7. **Solución y validación:** corregir y comprobar que no se debilitó la seguridad.
8. **Reto autónomo:** variante sin instrucciones paso a paso.
9. **Progreso:** actualizar README y registrar qué funcionó realmente.

> **Regla de honestidad:** si un fallo no se resuelve, documentar el resultado, los intentos y la restauración. Nunca inventar una captura ni declarar una práctica superada sin evidencia.

### Plantilla de evidencias por módulo

```markdown
# Módulo XX — Título
- Estado: Pendiente / En curso / Completado / Bloqueado
- Fecha:
- Versión de Fedora:
- Objetivo diferencial frente a Arch:
- Comandos y decisiones:
- Capturas: `evidence/XX/`
- Resultado observado:
- Incidente Break & Fix: `incident-reports/XX.md`
- Validación final:
- Reto autónomo:
- Pendientes:
```

### Plantilla Break & Fix

```markdown
# Incidente — Nombre
- Entorno y punto de restauración:
- Cambio deliberado:
- Síntoma:
- Observación:
- Registros consultados:
- Hipótesis:
- Prueba:
- Causa raíz confirmada (o no confirmada):
- Solución aplicada:
- Validación:
- Impacto y riesgos:
- Cómo evitar recurrencia:
- Capturas y comandos:
```

## 4. Temario completo: 40 módulos en 8 fases

### Fase 1 — Fedora Server y Cockpit (módulos 00–07)

**Prioridad principal.** Objetivo: dejar operativa la VM principal y administrar el servidor desde Ubuntu por terminal y navegador.

| Nº | Módulo | Laboratorio / evidencia |
|---|---|---|
| 00 | Preparar VirtualBox en Ubuntu | UEFI, NAT, disco dinámico y primera instantánea; captura de configuración. |
| 01 | Instalar Fedora Server con Anaconda | Decisiones frente a instalación manual de Arch; registro de particionado y versión. |
| 02 | Anatomía de Fedora Server | Ediciones, perfiles, repositorios y paquetes de la instalación. |
| 03 | Primer acceso a Cockpit | HTTPS, usuarios, autenticación y relación con SSH. |
| 04 | Administración desde Cockpit | Servicios, cuentas, registros, recursos y actualizaciones; comparación con CLI. |
| 05 | Almacenamiento desde Cockpit | Segundo disco virtual, LVM y montajes; documentar cambios. |
| 06 | Administración remota segura | NAT/host-only/puente, firewalld y acceso restringido a Cockpit. |
| 07 | Break & Fix de acceso | Simular fallo de Cockpit, SSH o red; recuperar sin reinstalar. |

**Proyecto 1:** Fedora Server accesible desde Ubuntu, con SSH, Cockpit, configuración de red documentada, instantánea y reporte de recuperación.

### Fase 2 — RPM, DNF5 y COPR (módulos 08–13)

| Nº | Módulo | Laboratorio / evidencia |
|---|---|---|
| 08 | RPM frente a pacman | Consultar base de datos, propietario de archivos, metadatos y dependencias. |
| 09 | DNF5 y transacciones | Resolver dependencias, revisar historial y comprender límites de undo/rollback. |
| 10 | Repositorios y confianza | Configuración, firmas GPG, actualizaciones y procedencia. |
| 11 | COPR | Evaluar mantenedor, código, paquete y riesgos antes de habilitar repositorios. |
| 12 | Construir un RPM | Archivo SPEC, rpmbuild, dependencias y validación de paquete. |
| 13 | Break & Fix de paquetes | Repositorio roto, conflicto o dependencia insatisfecha. |

**Comparación importante:** COPR no es un AUR idéntico ni una garantía de curaduría: es infraestructura de construcción y distribución de paquetes de terceros; evaluar cada proyecto.

### Fase 3 — SELinux enforcing (módulos 14–19)

| Nº | Módulo | Laboratorio / evidencia |
|---|---|---|
| 14 | Política SELinux en Fedora | Política targeted, dominios y tipos; contraste práctico con el modelo DAC tradicional (usuarios/grupos/permisos) visto en Arch. |
| 15 | Contextos persistentes | `ls -Z`, `ps -Z`, `semanage`, `restorecon`; archivos, procesos y puertos. |
| 16 | Diagnóstico AVC | `ausearch`, registros de auditoría y `sealert` si está disponible. |
| 17 | Booleanos y puertos | Publicar servicio en puerto no estándar con etiquetas y permisos mínimos. |
| 18 | Políticas locales y audit2allow | Interpretar propuestas, identificar reglas excesivas y justificar excepciones. |
| 19 | Break & Fix SELinux | Recuperar un servicio con contexto incorrecto **sin desactivar enforcing**. |

**Proyecto 2:** servicio web con puerto no estándar, SELinux enforcing, evidencia de denegación, corrección mínima y validación.

> `audit2allow` es una herramienta de análisis y último recurso, **no** una orden de aceptar automáticamente todas las reglas sugeridas. Primero comprobar etiquetas, booleanos, puertos y configuración de la aplicación.

### Fase 4 — Fedora Server avanzado y Cockpit (módulos 20–24)

| Nº | Módulo | Laboratorio / evidencia |
|---|---|---|
| 20 | Servidor web | Desplegar y administrar servicio desde CLI y Cockpit. |
| 21 | Cockpit y Podman | Imágenes, contenedores, volúmenes y supervisión. |
| 22 | Cockpit y virtualización | Estudiar capacidades y limitaciones de virtualización anidada en VirtualBox; práctica opcional según hardware. |
| 23 | Actualizaciones de versión | Respaldos, procedimiento de upgrade y estrategia de recuperación. |
| 24 | Break & Fix de mantenimiento | Recuperar servicio o configuración tras una actualización problemática simulada. |

**Proyecto 3:** aplicación en contenedor, supervisión web, SELinux enforcing y procedimiento de mantenimiento.

### Fase 5 — Rust en Fedora y empaquetado RPM (módulos 25–29)

| Nº | Módulo | Laboratorio / evidencia |
|---|---|---|
| 25 | Toolchain Rust | Comparar rustup, paquetes Fedora y versiones disponibles. |
| 26 | Cargo y bibliotecas del sistema | Dependencias nativas, paquetes `-devel`, pkg-config y enlazado. |
| 27 | Utilidad Rust para Fedora Server | Crear una CLI pequeña que consulte información de diagnóstico del sistema. |
| 28 | Empaquetado RPM | SPEC para binario Rust, build, instalación, desinstalación y validación. |
| 29 | Break & Fix de compilación | Resolver librería nativa ausente, error de enlace o metadatos RPM incorrectos. |

**Proyecto 4:** utilidad de diagnóstico escrita en Rust y distribuida como `.rpm` propio.

### Fase 6 — Silverblue, rpm-ostree y CoreOS (módulos 30–35)

| Nº | Módulo | Laboratorio / evidencia |
|---|---|---|
| 30 | OSTree y rpm-ostree | Contrastar despliegues basados en imágenes con DNF tradicional. |
| 31 | Fedora Silverblue | VM temporal, layering, Toolbox y separación entre sistema y aplicaciones. |
| 32 | Rollback de despliegues | Actualizar, inspeccionar despliegues y regresar al anterior. |
| 33 | Fedora CoreOS | VM temporal, Butane e Ignition; aprovisionamiento declarativo. |
| 34 | Contenedores en CoreOS | Servicio contenerizado y comparación operativa con Fedora Server. |
| 35 | Break & Fix de despliegue | Corregir configuración de aprovisionamiento o recuperar despliegue previo. |

**Nota:** Silverblue y CoreOS no son intercambiables: Silverblue está orientado a escritorio atómico; CoreOS, a hosts para contenedores y aprovisionamiento automatizado. No asumir que todas las operaciones rpm-ostree se realizan igual en ambos.

### Fase 7 — Automatización específica de Fedora (módulos 36–38)

| Nº | Módulo | Laboratorio / evidencia |
|---|---|---|
| 36 | Kickstart | Automatizar una instalación Fedora y conservar configuración reproducible. |
| 37 | Ansible aplicado a Fedora | Gestionar DNF, SELinux, firewalld y Cockpit; no repetir fundamentos de Ansible. |
| 38 | Modelos de despliegue | Comparar Fedora tradicional, Kickstart y sistemas basados en imágenes. |

### Fase 8 — Proyecto final (módulo 39)

**Fedora Enterprise Lab.** Entorno reproducible con:
- Fedora Server administrado por SSH y Cockpit.
- Repositorios RPM controlados y un paquete propio en Rust.
- Servicio en Podman protegido con SELinux enforcing.
- Procedimiento de actualización, respaldo y recuperación probado.
- Automatización de configuración Fedora mediante Ansible.
- Laboratorio temporal que contraste Fedora Server con Silverblue o CoreOS.
- Capturas, registro de cambios e informes Break & Fix verificables.

**Criterio de aprobación:** otra persona debe poder reproducir el entorno siguiendo el repositorio, identificar las versiones usadas y comprobar los resultados sin depender de explicaciones verbales.

## 5. Calendario sugerido

| Bloque | Semanas orientativas |
|---|---:|
| Fedora Server y Cockpit | 5 |
| RPM, DNF5 y COPR | 3 |
| SELinux enforcing | 4 |
| Fedora Server avanzado | 3 |
| Rust y RPM | 3 |
| Silverblue y CoreOS | 4 |
| Automatización y proyecto final | 4 |
| **Total orientativo** | **26** |

Con 5–7 horas semanales, ajustar el ritmo según los laboratorios y las limitaciones de hardware. No avanzar si una práctica crítica de recuperación quedó sin validar.

## 6. Evaluación

| Nivel | Evidencia exigida |
|---|---|
| Básico | Instalar Fedora Server y administrarlo mediante Cockpit y DNF. |
| Intermedio | Diagnosticar conflictos RPM y denegaciones SELinux. |
| Avanzado | Desplegar servicios protegidos, empaquetar Rust como RPM y ejecutar mantenimiento recuperable. |
| Experto | Reproducir el laboratorio, administrar despliegues basados en imágenes y resolver incidentes documentados. |

## 7. Estructura del repositorio

Estructura plana (mismo patrón que `arch-linux-mastery`/`arch-linux-desktop`):
una carpeta numerada por módulo en la raíz, con su `.md` y su `evidencias/`.
Ver la metodología de tutoría fija en `docs/tutor.txt`.

```text
fedora-mastery/
├── docs/
│   ├── README.md
│   ├── fedora-mastery-guia.md
│   └── tutor.txt
├── img/                          # capturas crudas, gitignored
├── 00-preparacion-virtualbox/
│   ├── 00-preparacion-virtualbox.md
│   └── evidencias/
├── 01-instalacion-fedora-server/
│   ├── 01-instalacion-fedora-server.md
│   └── evidencias/
└── ...                            # una carpeta NN-nombre/ por módulo, hasta el 39
```

### Plantilla de `PROGRESS.md`

```markdown
# Progreso Fedora Mastery

| Módulo | Estado | Evidencia | Break & Fix | Fecha |
|---|---|---|---|---|
| 00 | Pendiente | — | — | — |
| 01 | Pendiente | — | — | — |

## Próximo objetivo
Preparar VM Fedora Server y registrar configuración inicial.

## Bloqueos
Ninguno.

## Lecciones aprendidas
- ...
```

## 8. Referencias oficiales

Consultar documentación correspondiente **a la versión instalada**, ya que Fedora, DNF y sus ediciones evolucionan.

- [Fedora Server](https://docs.fedoraproject.org/en-US/fedora-server/)
- [Documentación de Fedora](https://docs.fedoraproject.org/)
- [Cockpit](https://cockpit-project.org/)
- [DNF5](https://dnf5.readthedocs.io/)
- [Fedora COPR](https://copr.fedorainfracloud.org/)
- [Fedora Packaging Guidelines](https://docs.fedoraproject.org/en-US/packaging-guidelines/)
- [SELinux en Fedora](https://docs.fedoraproject.org/en-US/quick-docs/selinux-getting-started/)
- [Fedora Silverblue](https://docs.fedoraproject.org/en-US/fedora-silverblue/)
- [Fedora CoreOS](https://docs.fedoraproject.org/en-US/fedora-coreos/)

## 9. Primer hito

Completar el módulo 07 con una VM Fedora Server funcional, Cockpit accesible desde Ubuntu, una instantánea de VirtualBox verificada y un informe real de recuperación ante una pérdida simulada de acceso. Esa VM será la base del resto del curso.
