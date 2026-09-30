# Módulo 38 — Modelos de despliegue (comparación)

- Estado: Completado
- Fecha: 2026-09-30
- Objetivo diferencial: cierra la Fase 7. No hay práctica nueva en VM
  — es una síntesis comparativa de los tres modelos de despliegue
  vividos en carne propia a lo largo del curso, con datos reales de
  cada módulo, no teoría genérica.

## Los tres modelos

### 1. Fedora tradicional (instalación manual + DNF)

- **Referencia real**: módulo 01 (instalación con Anaconda, cada
  pantalla recorrida a mano), módulos 08-13 (DNF paquete por paquete
  durante toda la vida del sistema), módulo 23 (upgrade mayor real
  F44→F45 con `dnf system-upgrade`, 1083 paquetes, ~1 GiB descargado).
- **Mecanismo**: el sistema de archivos es mutable desde el primer
  arranque. Cada `dnf install`/`dnf upgrade` modifica el estado in
  situ, paquete por paquete.
- **Rollback real**: no existe a nivel de sistema — el módulo 24
  (Break & Fix) lo demostró: reparar un incidente significa
  diagnosticar y corregir manualmente (`sed`, `systemctl restart`),
  no volver a un estado anterior con un comando.
- **Velocidad de instalación**: la más lenta de las tres — instala
  cientos de paquetes reales, uno por uno, resolviendo dependencias
  en cada transacción.

### 2. Kickstart (Anaconda automatizado)

- **Referencia real**: módulo 36 — el mismo instalador Anaconda del
  módulo 01, pero alimentado por un archivo de texto (`ks.cfg`) vía
  un disco `OEMDRV`, sin ninguna pantalla interactiva.
- **Mecanismo**: exactamente el mismo que Fedora tradicional una vez
  instalado — el sistema resultante es tan mutable como cualquier
  instalación manual, gestionado con DNF día a día.
- **Diferencial real**: automatiza el *proceso de instalación*, no
  el *modelo de gestión posterior*. El incidente real del módulo 36
  (contraseña rota por contexto SELinux corrupto, reparado con
  `touch /.autorelabel`) lo confirma: seguimos en terreno de
  diagnóstico manual tradicional, Kickstart no cambia eso.
- **Velocidad de instalación**: rápida de *lanzar* (sin intervención
  humana), pero el tiempo real de instalación de paquetes es el mismo
  que el modelo tradicional — sigue instalando cientos de paquetes.

### 3. Sistemas basados en imágenes (Silverblue/CoreOS)

- **Referencia real**: módulos 30-35. `rpm-ostree status` nunca
  muestra una lista de paquetes individuales — muestra **un
  despliegue** identificado por un commit hash y una firma GPG del
  árbol completo (módulo 31). El aprovisionamiento inicial es
  declarativo (Butane → Ignition, módulo 33), no interactivo ni un
  archivo de respuestas sobre un instalador tradicional.
- **Mecanismo**: el sistema base es de solo lectura. Layering
  (`rpm-ostree install`) prepara un despliegue **nuevo**, nunca
  modifica el actual en caliente — confirmado en el módulo 31 con
  `Staging deployment... done` y `deployment count change` en 0 al
  hacer rollback (módulo 32).
- **Rollback real**: `rpm-ostree rollback` es instantáneo — solo
  reordena qué despliegue (ya presente en el disco) arranca por
  defecto. Validado ida y vuelta en el módulo 32 sin reinstalar nada
  ni tocar ningún snapshot de VirtualBox.
- **Velocidad de instalación**: `coreos-installer install` (módulo
  33) completó en segundos — escribir una imagen ya comprimida al
  disco, no resolver ni instalar paquetes uno por uno.
- **Costo real**: para tocar herramientas de desarrollo o paquetes
  tradicionales hace falta un contenedor aparte (Toolbox, módulo 31)
  — el sistema base nunca se toca directamente para eso.

## Tabla comparativa (datos reales del curso)

| Criterio | Tradicional (01) | Kickstart (36) | Imagen (30-35) |
|---|---|---|---|
| Instalación | Manual, interactiva | Automatizada, mismo Anaconda | `coreos-installer`, segundos |
| Unidad de cambio | Paquete individual | Paquete individual | Despliegue completo (commit) |
| Rollback de sistema | No existe | No existe | Instantáneo (`rpm-ostree rollback`) |
| Diagnóstico de incidentes | Manual (mod. 07, 13, 19, 24) | Manual (mismo modelo, mod. 36) | Igual de manual para *contenedores* gestionados por systemd (mod. 35), pero el sistema base nunca se corrompe a medias |
| Reproducibilidad | Baja (cada instalación puede divergir) | Alta (mismo `.ks` = mismo resultado) | Muy alta (mismo commit = bit a bit idéntico) |
| Mejor para | Escritorio/servidor con cambios frecuentes de paquetes | Aprovisionamiento repetible de servidores tradicionales en escala | Hosts de contenedores, infraestructura inmutable, escritorio atómico |

## Criterio propio de cuándo usar cada uno

- **Fedora tradicional**: cuando el sistema necesita instalar y
  desinstalar paquetes con frecuencia, y el operador quiere control
  fino paquete por paquete (ej. una estación de desarrollo con
  necesidades cambiantes) — el costo es la falta de rollback real.
- **Kickstart**: cuando hay que provisionar **muchos** servidores
  Fedora tradicionales de forma reproducible (ej. un parque de
  servidores físicos o VMs con la misma configuración base), pero se
  acepta seguir gestionando cada uno con DNF después. No resuelve el
  problema de "estado que diverge con el tiempo" — solo el de "instalar
  rápido y consistente".
- **Imagen (Silverblue/CoreOS)**: cuando la prioridad es
  **reproducibilidad y reversibilidad reales** — hosts de
  contenedores, flotas donde un nodo con problemas se reemplaza
  (no se repara) por uno nuevo con la imagen correcta, o escritorios
  donde nunca se quiere un sistema a medio romper. El costo real es
  la fricción para tareas de desarrollo tradicionales (Toolbox de por
  medio) y el aprovisionamiento de "día 2" más rígido (confirmado en
  el módulo 35: reparar un contenedor declarativo requiere el mismo
  diagnóstico manual que cualquier otro servicio de systemd, la
  inmutabilidad protege el sistema base, no exime de administrarlo).

## Pendientes
