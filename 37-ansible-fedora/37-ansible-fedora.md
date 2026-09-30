# Módulo 37 — Ansible aplicado a Fedora

- Estado: Completado
- Fecha: 2026-09-29
- Versión de Fedora: Fedora Linux 44 (Server Edition)
- Objetivo diferencial: arranca la Fase 7. No repite fundamentos de
  Ansible (playbooks, inventarios básicos) — se enfoca en los módulos
  específicos de Fedora: `ansible.builtin.dnf`, `ansible.posix.seport`
  (contexto de puerto SELinux), `ansible.posix.firewalld`.

## Concepto diferencial

Reproduce declarativamente el mismo patrón de dos capas de seguridad
del módulo 17 (SELinux + firewalld para un servicio web en puerto no
estándar), pero con la propiedad central de Ansible: **idempotencia**.
Correr el mismo playbook dos veces debe dar el mismo resultado sin
duplicar cambios — la segunda corrida debe reportar `changed=0`.

## Práctica guiada

1. Instalar Ansible en el host (control node).
2. Confirmar conectividad SSH hacia `Fedora_Server` (módulo 06).
3. Escribir un inventario mínimo y un playbook que:
   - Instale `httpd` (`ansible.builtin.dnf`).
   - Agregue el puerto 8586/tcp al contexto `http_port_t`
     (`ansible.posix.seport`).
   - Abra 8586/tcp en la zona activa de firewalld
     (`ansible.posix.firewalld`).
   - Habilite y arranque `httpd` (`ansible.builtin.service`).
4. Correr el playbook una primera vez — observar los `changed`.
5. Correr el playbook una segunda vez — confirmar `changed=0`.
6. Validar con `curl` real desde el host.

## Hallazgos reales

1. **El host (Ubuntu) no trae Ansible preinstalado** — instalado con
   `sudo apt install -y ansible-core`, que además arrastró el
   metapaquete `ansible` completo con las collections community.
2. **La colección `ansible.posix` empaquetada por Ubuntu (1.5.4) no
   incluye el módulo `seport`** — ni siquiera al actualizarla vía
   `ansible-galaxy collection install ansible.posix --upgrade`
   (versión 2.2.2 de Galaxy). Pivoteado a `ansible.builtin.command`
   con `semanage port -a`, logrando idempotencia manual con
   `register`/`when` en vez del módulo dedicado.
3. **`ansible.builtin.dnf` falló con `Could not import the libdnf5
   python module`** — la VM `Fedora_Server` ya está en Fedora 45
   (módulo 23), que usa DNF5 por defecto; el binding de Ansible
   necesita el paquete `python3-libdnf5`, ausente en el nodo remoto.
4. **Instalar esa única dependencia arrastró una actualización
   completa del sistema** (205 paquetes, repositorio
   `updates-testing`): la larga lista de "Eliminando" en la salida de
   `dnf` en realidad eran pares de "Modernizar" (upgrade in situ), no
   pérdida de software — confirmado con `dnf history info` y
   validando que `libvirtd` (módulo 22) seguía intacto.
5. **Tras esa actualización, el propio módulo `dnf5.py` de Ansible se
   rompió** con `AttributeError: 'Base' object has no attribute
   'load_config_from_file'` — incompatibilidad real entre la versión
   de `libdnf5` recién actualizada (Fedora 45 Prerelease, repos de
   testing) y la API que el binding de Ansible esperaba. Pivoteado
   también a `command`/`rpm -q` con idempotencia manual.
6. **`ssh usuario@host "comando"` sin `-t` no puede pedir la
   contraseña de `sudo`** ("a terminal is required to read the
   password") — necesario el flag `-t` para asignar un pseudo-TTY en
   comandos remotos no interactivos.
7. **El servicio no respondía en el puerto 8586 pese a SELinux y
   firewalld ya configurados**: faltaba la propia directiva `Listen
   8586` en `httpd.conf` — las dos capas de seguridad no bastan si el
   servicio nunca escucha ahí. Agregada con `ansible.builtin.lineinfile`
   + un *handler* (`notify`) que reinicia `httpd` solo cuando el
   archivo cambia.
8. **Idempotencia real demostrada en tres corridas**: 1ª corrida
   `changed=2` (instala/configura), 2ª corrida tras agregar `Listen`
   `changed=2` (el ajuste de `httpd.conf` + su handler), 3ª corrida
   `changed=0` total — ni el propio *handler* se disparó, confirmando
   que el playbook completo ya reflejaba el estado declarado sin
   necesidad de repetir ningún cambio.

## Evidencias

### Instalación y verificación del control node

**01 — `sudo apt install ansible-core` + `ansible --version`**

![apt install ansible core ansible version](evidencias/01-apt-install-ansible-core-ansible-version.png)

**02 — `ansible-galaxy collection list`: `ansible.posix 1.5.4` (paquete APT)**

![ansible galaxy collection list posix 1 5 4](evidencias/02-ansible-galaxy-collection-list-posix-1-5-4.png)

### Encontrando la VM y la IP correctas

**03 — `VBoxManage showvminfo`: adaptador host-only `vboxnet0`**

![vboxmanage showvminfo hostonly vboxnet0](evidencias/03-vboxmanage-showvminfo-hostonly-vboxnet0.png)

**04 — Primer intento: VM equivocada (`Fedora_Kickstart`, no `Fedora_Server`)**

![ip addr vm equivocada fedora kickstart](evidencias/04-ip-addr-vm-equivocada-fedora-kickstart.png)

**05 — GRUB de `Fedora_Server`, la VM correcta**

![grub fedora server vm correcta](evidencias/05-grub-fedora-server-vm-correcta.png)

**06 — Sesión iniciada en `Fedora_Server`**

![fedora server sesion iniciada](evidencias/06-fedora-server-sesion-iniciada.png)

**07 — `ip addr`: interfaz `enp0s8` con IP `192.168.59.101`**

![ip addr fedora server enp0s8 192 168 59 101](evidencias/07-ip-addr-fedora-server-enp0s8-192-168-59-101.png)

**08 — SSH exitoso desde el host hacia `Fedora_Server`**

![ssh exitoso desde host a fedora server](evidencias/08-ssh-exitoso-desde-host-a-fedora-server.png)

**09 — `exit`, sesión SSH cerrada**

![exit ssh sesion cerrada](evidencias/09-exit-ssh-sesion-cerrada.png)

### Inventario y primer playbook (con `seport`)

**10 — `inventory.ini` escrito y confirmado**

![inventory ini escrito confirmado](evidencias/10-inventory-ini-escrito-confirmado.png)

**11 — Primer `web-declarativo.yml`, con `ansible.posix.seport`**

![playbook v1 con seport escrito](evidencias/11-playbook-v1-con-seport-escrito.png)

**12 — Typo real detectado: `8586/tpc` en vez de `8586/tcp`**

![typo 8586 tpc detectado](evidencias/12-typo-8586-tpc-detectado.png)

### Primeros intentos fallidos de ejecución

**13 — Error real: `sudo` local de más, flags con un solo guión**

![error sudo local flags un solo guion](evidencias/13-error-sudo-local-flags-un-solo-guion.png)

**14 — Reintento correcto: prompt "SSH password"**

![reintento ssh password prompt](evidencias/14-reintento-ssh-password-prompt.png)

**15 — Prompt "BECOME password"**

![become password prompt](evidencias/15-become-password-prompt.png)

**16 — Error real: `ansible.posix.seport` no se pudo resolver**

![error seport no resuelto](evidencias/16-error-seport-no-resuelto.png)

**17 — `ansible-doc -l ansible.posix`: la colección 1.5.4 no incluye `seport`**

![ansible doc l posix sin seport](evidencias/17-ansible-doc-l-posix-sin-seport.png)

### Diagnóstico de la colección faltante

**18 — `ansible-galaxy collection install ansible.posix --upgrade`: versión 2.2.2 instalada**

![ansible galaxy collection install upgrade 2 2 2](evidencias/18-ansible-galaxy-collection-install-upgrade-2-2-2.png)

**19 — `collection list`: ambas versiones (1.5.4 y 2.2.2) coexisten**

![collection list ambas versiones coexisten](evidencias/19-collection-list-ambas-versiones-coexisten.png)

**20 — `find ~/.ansible/collections -iname "*seport*"`: sin resultados, ni siquiera en la 2.2.2**

![find seport sin resultados](evidencias/20-find-seport-sin-resultados.png)

### Pivote a `command` + idempotencia manual (versión 2 del playbook)

**21 — `web-declarativo.yml` reescrito con `command`/`semanage port` y `register`/`when`**

![playbook v2 command semanage idempotencia manual](evidencias/21-playbook-v2-command-semanage-idempotencia-manual.png)

**22 — Playbook v2 completo, confirmado**

![playbook v2 confirmado completo](evidencias/22-playbook-v2-confirmado-completo.png)

**23 — Error real: `Could not import the libdnf5 python module`**

![error libdnf5 python module](evidencias/23-error-libdnf5-python-module.png)

### Instalando `python3-libdnf5` en el nodo remoto

**24 — Error real: comando corrido en la consola de la VM en vez del host**

![ls vm consola directa no host](evidencias/24-ls-vm-consola-directa-no-host.png)

**25 — `ssh` sin `-t`: "sudo: a terminal is required"**

![ssh sin t sudo terminal required](evidencias/25-ssh-sin-t-sudo-terminal-required.png)

**26 — `ssh -t`: instalando `python3-libdnf5`, 205 paquetes en la transacción**

![ssh t instalando python3 libdnf5 205 paquetes](evidencias/26-ssh-t-instalando-python3-libdnf5-205-paquetes.png)

**27 — `dnf history list`: transacción 19, 205 paquetes alterados**

![dnf history list transaccion 19 205 paquetes](evidencias/27-dnf-history-list-transaccion-19-205-paquetes.png)

**28 — `dnf history info 19`: todo "Modernizar" (upgrade), no eliminación real**

![dnf history info 19 modernizar no eliminar](evidencias/28-dnf-history-info-19-modernizar-no-eliminar.png)

**29 — `systemctl status libvirtd` cortado por `head -5`**

![systemctl status libvirtd head 5 cortado](evidencias/29-systemctl-status-libvirtd-head-5-cortado.png)

**30 — `systemctl status libvirtd` completo: `inactive (dead)`, activado por sockets (normal)**

![systemctl status libvirtd completo inactive dead](evidencias/30-systemctl-status-libvirtd-completo-inactive-dead.png)

### Segundo error de compatibilidad y pivote de `dnf` a `command`

**31 — Segundo intento del playbook: `AttributeError: load_config_from_file`**

![segundo intento playbook error dnf5 load config](evidencias/31-segundo-intento-playbook-error-dnf5-load-config.png)

**32 — Playbook v3 (con `command`/`rpm -q` para httpd): exitoso, `ok=6 changed=2`**

![playbook v3 httpd command exitoso ok6 changed2](evidencias/32-playbook-v3-httpd-command-exitoso-ok6-changed2.png)

### Idempotencia real y el hallazgo del `Listen` faltante

**33 — `curl` a 8586 vacío, inicio de la segunda corrida**

![curl 8586 vacio segunda corrida inicio](evidencias/33-curl-8586-vacio-segunda-corrida-inicio.png)

**34 — Segunda corrida: idempotencia parcial confirmada (`changed=0`, `skipped=2`)**

![segunda corrida idempotencia changed0 skipped2](evidencias/34-segunda-corrida-idempotencia-changed0-skipped2.png)

**35 — `ss -tlnp` sin `-t`: mismo error de terminal para `sudo`**

![ss tlnp httpd sin t sudo error](evidencias/35-ss-tlnp-httpd-sin-t-sudo-error.png)

**36 — `ss -tlnp` confirma: `httpd` solo escucha en `*:8585`, nunca en 8586**

![ss tlnp httpd solo 8585 confirmado](evidencias/36-ss-tlnp-httpd-solo-8585-confirmado.png)

**37 — Playbook v4: tarea `lineinfile` agrega `Listen 8586`, `changed`**

![playbook v4 listen 8586 lineinfile changed](evidencias/37-playbook-v4-listen-8586-lineinfile-changed.png)

**38 — `RUNNING HANDLER [reiniciar httpd]`: disparado por el `notify`**

![running handler reiniciar httpd changed](evidencias/38-running-handler-reiniciar-httpd-changed.png)

**39 — Inicio del playbook v4: `Listen 8586` agregado**

![playbook v4 inicio listen 8586 agregado](evidencias/39-playbook-v4-inicio-listen-8586-agregado.png)

**40 — Handler disparado, `changed`**

![playbook v4 handler disparado changed](evidencias/40-playbook-v4-handler-disparado-changed.png)

**41 — Recap del playbook v4: `ok=7 changed=2 failed=0`**

![playbook v4 recap ok7 changed2](evidencias/41-playbook-v4-recap-ok7-changed2.png)

**42 — `curl` a 8586: HTML real del módulo 17**

![curl 8586 html real modulo 17](evidencias/42-curl-8586-html-real-modulo-17.png)

### Confirmación final de idempotencia total

**43 — Intento con contraseña mal tipeada: `UNREACHABLE`**

![tercer intento password incorrecta unreachable](evidencias/43-tercer-intento-password-incorrecta-unreachable.png)

**44-45 — Cuarta corrida en progreso**

![cuarta corrida reintento inicio](evidencias/44-cuarta-corrida-reintento-inicio.png)
![cuarta corrida continuacion](evidencias/45-cuarta-corrida-continuacion.png)

**46 — Idempotencia total confirmada: `ok=6 changed=0 failed=0`, ni el handler se disparó**

![idempotencia total confirmada ok6 changed0](evidencias/46-idempotencia-total-confirmada-ok6-changed0.png)

## Pendientes
