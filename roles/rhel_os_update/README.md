# Role: rhel_os_update

Atualiza o sistema operacional em hosts **RHEL 7/8/9/10**, com reboot controlado.
Detecta o gerenciador de pacotes automaticamente: **yum** (RHEL 7), **dnf** (RHEL 8/9), **dnf5** (RHEL 10).

## Variáveis principais (`defaults/main.yml`)

| Variável | Padrão | Descrição |
|---|---|---|
| `rhel_os_update_supported_majors` | `[7,8,9,10]` | Versões maiores aceitas |
| `rhel_os_update_scope` | `all` | `all` (tudo) ou `security` (só erratas) |
| `rhel_os_update_exclude` | `[]` | Pacotes a excluir (ex.: `kernel*`) |
| `rhel_os_update_update_cache` | `true` | Atualiza cache antes |
| `rhel_os_update_autoremove` | `false` | Remove pacotes órfãos |
| `rhel_os_update_reboot` | `auto` | `auto` (`needs-restarting -r`), `true` ou `false` |
| `rhel_os_update_reboot_timeout` | `900` | Timeout do reboot (s) |

> `needs-restarting` vem do `yum-utils` (RHEL 7) / `dnf-utils` (RHEL 8+). Se ausente, o modo `auto` não força reboot.

## Uso no AAP

Exponha `rhel_os_update_scope` e `rhel_os_update_reboot` como **survey**. Rode com `serial` para atualizar em lotes.

## Exemplo

Veja [`playbooks/os-update.yml`](../../playbooks/os-update.yml).
