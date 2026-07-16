# Role: linux_packages

Instala/remove pacotes e grupos em hosts **RHEL 7/8/9/10**, **Debian** e **Ubuntu**.
Usa o módulo genérico `ansible.builtin.package`, que resolve para `dnf`/`yum` (RedHat)
ou `apt` (Debian/Ubuntu). A **baseline** de pacotes usa os nomes corretos de cada
família, carregados de `vars/RedHat.yml` / `vars/Debian.yml` (ex.: `vim-enhanced` no
RHEL, `vim` no Debian).

## Variáveis principais (`defaults/main.yml`)

| Variável | Padrão | Descrição |
|---|---|---|
| `linux_packages_supported_families` | `[RedHat, Debian]` | Famílias aceitas |
| `linux_packages_manage_baseline` | `true` | Instala a baseline da família |
| `linux_packages_install` | `[]` | Pacotes adicionais (nomes válidos na distro alvo) |
| `linux_packages_remove` | `[]` | Pacotes a remover |
| `linux_packages_groups` | `[]` | Grupos `@` — **só RHEL** (ex.: `@Development Tools`) |
| `linux_packages_state` | `present` | `present` ou `latest` |

> A baseline padrão (`vim/wget/curl/tar/unzip/bash-completion`) está em `vars/`.
> Em Debian/Ubuntu a role atualiza o cache do apt antes de instalar. Grupos de
> pacotes (`@...`) são um conceito de RHEL e são ignorados fora da família RedHat.

## Exemplo

Veja [`playbooks/packages.yml`](../../playbooks/packages.yml).
