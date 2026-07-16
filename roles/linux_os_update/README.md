# Role: linux_os_update

Atualiza o sistema operacional em hosts **RHEL 7/8/9/10**, **Debian** e **Ubuntu**, com
reboot controlado. Detecta a família e o gerenciador de pacotes automaticamente:

| Família | Gerenciador | `all` | `security` |
|---|---|---|---|
| RedHat | `yum` (7) / `dnf` (8/9) / `dnf5` (10) | `state: latest` | `security: yes` |
| Debian / Ubuntu | `apt` | `upgrade: dist` | `unattended-upgrade` |

## Variáveis principais (`defaults/main.yml`)

| Variável | Padrão | Descrição |
|---|---|---|
| `linux_os_update_supported_families` | `[RedHat, Debian]` | Famílias aceitas |
| `linux_os_update_scope` | `all` | `all` (tudo) ou `security` (só segurança) |
| `linux_os_update_exclude` | `[]` | Pacotes a excluir — **só RHEL** (no Debian use `apt-mark hold`) |
| `linux_os_update_update_cache` | `true` | Atualiza cache antes |
| `linux_os_update_autoremove` | `false` | Remove pacotes órfãos |
| `linux_os_update_reboot` | `auto` | `auto`, `true` ou `false` |
| `linux_os_update_reboot_timeout` | `900` | Timeout do reboot (s) |

## Detecção de reboot (modo `auto`)

- **RHEL**: `needs-restarting -r` (do `yum-utils`/`dnf-utils`; se ausente, não reinicia).
- **Debian/Ubuntu**: presença do arquivo `/var/run/reboot-required`.

> No escopo `security` em Debian/Ubuntu a role instala e executa `unattended-upgrades`,
> que por padrão aplica apenas as atualizações dos repositórios de segurança.

## Uso no AAP

Exponha `linux_os_update_scope` e `linux_os_update_reboot` como **survey**. Rode com
`serial` para atualizar em lotes.

## Exemplo

Veja [`playbooks/os-update.yml`](../../playbooks/os-update.yml).
