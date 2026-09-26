# Módulo 12 — Construcción de un RPM: SPEC, dependencias y validación

- Estado: Completado
- Fecha: 2026-09-26
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial frente a Arch: construir un RPM propio desde un
  archivo `.spec`, comparándolo con el `PKGBUILD` ya usado en
  `arch-linux-desktop` (módulo 26).

## Concepto diferencial

`rpmbuild` exige una estructura de directorios fija
(`~/rpmbuild/{SPECS,SOURCES,BUILD,RPMS,SRPMS}`), generada con
`rpmdev-setuptree` — a diferencia de `makepkg`, que solo necesita un
`PKGBUILD` en cualquier carpeta.

El archivo `.spec` tiene secciones obligatorias con nombres fijos
(`%prep`, `%build`, `%install`, `%files`, `%changelog`) — más rígido y
verboso que las funciones `build()`/`package()` de un PKGBUILD, pero más
explícito sobre qué pasa en cada etapa de la construcción.

## Práctica guiada

```bash
sudo dnf install rpm-build rpmdevtools
rpmdev-setuptree
ls ~/rpmbuild
```

Luego se escribe un `.spec` simple (paquete `fedora-mastery-diag`, un
script de diagnóstico propio sin fuente externa), se construye con
`rpmbuild -bb`, se valida el `.rpm` resultante y se instala.

## Hallazgos reales

1. **`rpm-build`/`rpmdevtools` instalaron 46 paquetes**, incluyendo
   `*-srpm-macros` de prácticamente todos los lenguajes que Fedora
   empaqueta (rust, go, ocaml, ghc, java, lua, zig, qt5/qt6...) — mucho
   más pesado que `base-devel` de Arch para lo mismo.
2. **Typo real en el `.spec`**: `%{_nindir}` en vez de `%{_bindir}` en
   la sección `%files` (verificado con `grep`, no era la fuente de la
   terminal) — corregido con `sed`.
3. **"mangling shebang... from `/bin/bash` to `#!/usr/bin/bash`"** —
   RPM normaliza automáticamente los shebangs a la ruta canónica
   (Fedora usa usrmerge: `/bin` → symlink a `/usr/bin`).
4. **"fecha fraudulenta en %changelog"**: puse "Fri Sep 26 2026" sin
   verificar el día real — `date -d "2026-09-26" +%A` confirmó que es
   **sábado**. `rpmbuild` valida que el nombre del día coincida con la
   fecha real del calendario, algo que un `PKGBUILD` no chequea.
5. **Heredoc anidado perdió una línea al pegarse** en la consola de
   VirtualBox: el `.spec` tenía la línea `echo "IP: ..."` en la versión
   escrita en el chat, pero el archivo real en la VM solo tenía 2 de
   los 3 `echo`. Corregido con `sed` en vez de repetir el pegado
   completo — lección real sobre los límites de pegar bloques grandes
   con heredocs anidados en una terminal de VM.
6. **Conexión directa con el módulo 10**: al instalar el `.rpm` local
   sin firmar, DNF avisó explícitamente *"comprobante OpenPGP omitido
   para 1 paquete desde repositorio: @commandline"* — confirma en la
   práctica el mecanismo de verificación GPG visto en la teoría.
7. **Validación final exitosa**: `fedora-mastery-diag` imprime
   hostname, las 3 IPs (NAT + host-only + IPv6) y el uso de disco real.

## Evidencias

**01-02 — Instalación de `rpm-build`/`rpmdevtools`: 46 paquetes, todos los `*-srpm-macros`**

![Resumen de la transacción, 46 paquetes](evidencias/01-instalar-rpm-build-rpmdevtools-46-paquetes.png)
![Progreso de instalación, srpm-macros de todos los lenguajes](evidencias/02-instalacion-progreso-srpm-macros.png)

**03 — `rpmdev-setuptree`: estructura de directorios creada**

![Estructura ~/rpmbuild](evidencias/03-rpmdev-setuptree-estructura.png)

**04-06 — El `.spec` con el typo real (`_nindir`), diagnóstico y corrección**

![spec creado, typo _nindir y archivo residual](evidencias/04-spec-creado-typo-nindir-archivo-residual.png)
![grep confirma el typo](evidencias/05-grep-confirma-typo-nindir.png)
![sed corrige a _bindir](evidencias/06-sed-corrige-a-bindir.png)

**07-08 — Primer build: advertencia de fecha fraudulenta, confirmada con `date`**

![rpmbuild primera vez, fecha fraudulenta](evidencias/07-rpmbuild-primera-vez-fecha-fraudulenta.png)
![date confirma que es sábado](evidencias/08-date-confirma-sabado.png)

**09 — Segundo build: limpio, sin advertencias**

![rpmbuild segunda vez, sin advertencias](evidencias/09-rpmbuild-segunda-vez-sin-advertencias.png)

**10-11 — Instalado y ejecutado, pero falta la línea IP — diagnóstico del heredoc perdido**

![rpm -qpi, -qpl, dnf install, ejecución sin línea IP](evidencias/10-rpm-qpi-qpl-dnf-install-falta-ip.png)
![hostname -I funciona solo, script instalado confirma el bug](evidencias/11-hostname-i-cat-script-instalado-bug-confirmado.png)

**12-13 — Corrección con `sed` y tercer build**

![sed agrega la línea IP faltante](evidencias/12-sed-agrega-linea-ip-faltante.png)
![rpmbuild tercera vez, build final](evidencias/13-rpmbuild-tercera-vez-final.png)

**14 — Validación final: las 3 líneas completas**

![dnf reinstall y ejecución final completa](evidencias/14-dnf-reinstall-ejecucion-final-completa.png)

## Pendientes
