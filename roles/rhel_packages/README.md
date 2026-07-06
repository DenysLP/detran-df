# Role: rhel_packages

Instala/remove pacotes e grupos em hosts **RHEL 7/8/9/10**.
Usa o módulo genérico `ansible.builtin.package`, que resolve para `yum`/`dnf`/`dnf5` conforme a versão.

## Variáveis principais (`defaults/main.yml`)

| Variável | Padrão | Descrição |
|---|---|---|
| `rhel_packages_install` | lista base | Pacotes a garantir instalados |
| `rhel_packages_remove` | `[]` | Pacotes a remover |
| `rhel_packages_groups` | `[]` | Grupos `@` a instalar (ex.: `@Development Tools`) |
| `rhel_packages_state` | `present` | `present` ou `latest` |

> Para habilitar repositórios extras (`enablerepo`) na instalação, configure o repo antes (via `ansible.builtin.yum_repository`) — o módulo `package` é agnóstico de repo.

## Exemplo

Veja [`playbooks/packages.yml`](../../playbooks/packages.yml).
