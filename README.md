# aap-automacoes — Roles Ansible (AAP)

Roles Ansible para automação de hosts **Linux** — **RHEL 7/8/9/10**, **Debian** e
**Ubuntu** — pensadas para uso no **Ansible Automation Platform (AAP)**. Cada role é um
building block de Job Templates. **Todas detectam a família do SO** (`os_family`) e
carregam de `vars/<família>.yml` o pacote, serviço e caminhos corretos antes de agir.

## Estrutura

```
.
├── collections/requirements.yml   # coleções (ansible.posix, community.general)
├── inventories/exemplo/hosts.ini  # inventário de exemplo (grupo [linux])
├── playbooks/                     # 1 playbook de exemplo por role
│   ├── checkup.yml                # check-up READ-ONLY Linux
│   ├── windows-checkup.yml        # check-up READ-ONLY Windows
│   ├── os-update.yml
│   ├── packages.yml
│   ├── ntp.yml
│   └── zabbix-agent.yml
└── roles/
    ├── linux_checkup/             # inventário/check-up READ-ONLY Linux (rodar antes das demais)
    ├── windows_checkup/           # inventário/check-up READ-ONLY Windows (via WinRM)
    ├── linux_os_update/           # update do SO (yum/dnf/dnf5/apt) + reboot controlado
    ├── linux_packages/            # instalação/remoção de pacotes e grupos
    ├── linux_ntp/                 # NTP via chrony (padrão NTP.br)
    └── linux_zabbix_agent/        # zabbix-agent2 (repo oficial + TLS/PSK)
```

## Roles

| Role | Função | Plataformas | Playbook |
|---|---|---|---|
| `linux_checkup` | Inventário/check-up **read-only** (updates, proxy, agentes, tempo) | RHEL / Debian / Ubuntu | `playbooks/checkup.yml` |
| `windows_checkup` | Inventário/check-up **read-only** via WinRM (acesso, rede, disco, updates, proxy, agentes, firewall, pacotes, domínio) | Windows | `playbooks/windows-checkup.yml` |
| `linux_os_update` | Atualiza o SO, reboot `auto/true/false` | RHEL / Debian / Ubuntu | `playbooks/os-update.yml` |
| `linux_packages` | Instala/remove pacotes e grupos | RHEL / Debian / Ubuntu | `playbooks/packages.yml` |
| `linux_ntp` | Instala e configura chrony (NTP.br) | RHEL / Debian / Ubuntu | `playbooks/ntp.yml` |
| `linux_zabbix_agent` | Instala e configura zabbix-agent2 | RHEL / Debian / Ubuntu | `playbooks/zabbix-agent.yml` |

Cada role tem seu próprio `README.md` com a lista completa de variáveis. Todas aceitam
`<role>_supported_families` para restringir as famílias e validam o host antes de agir.

## Como o multi-SO funciona

Cada role segue o mesmo padrão:

1. **Valida** a família (`ansible_facts['os_family']` ∈ `RedHat` / `Debian`).
2. **Carrega** `vars/RedHat.yml` ou `vars/Debian.yml` via `include_vars` — é onde ficam
   os nomes de pacote/serviço, caminhos e URLs de repositório de cada família.
3. **Executa** a tarefa, ramificando por família quando necessário:

| Role | RedHat (RHEL/derivados) | Debian / Ubuntu |
|---|---|---|
| `linux_os_update` | `yum` / `dnf` / `dnf5`; reboot via `needs-restarting` | `apt` (dist-upgrade / `unattended-upgrade`); reboot via `/var/run/reboot-required` |
| `linux_packages` | `dnf`/`yum` + grupos `@` | `apt` (cache atualizado antes); grupos ignorados |
| `linux_ntp` | serviço `chronyd`, conf `/etc/chrony.conf` | serviço `chrony`, conf `/etc/chrony/chrony.conf` |
| `linux_zabbix_agent` | repo RPM + `firewalld` | repo APT (chave `signed-by`) + `ufw` |

> **Debian ≠ Ubuntu?** Ambos são da família `Debian` e compartilham `vars/Debian.yml`.
> As poucas diferenças (ex.: URL `debian`/`ubuntu` do repo Zabbix) são resolvidas com
> `ansible_facts['distribution']` dentro das variáveis.

## Uso no AAP

1. **Project** → aponte para este repositório. O AAP lê `collections/requirements.yml`.
2. **Inventory** → hosts no grupo `linux` (RHEL, Debian e/ou Ubuntu).
3. **Job Template** → um por playbook em `playbooks/`; exponha variáveis via **Survey**.
4. Segredos (PSK do Zabbix) → **Vault Credential** / `ansible-vault`.

## Uso local (teste)

```bash
ansible-galaxy collection install -r collections/requirements.yml
ansible-playbook -i inventories/exemplo/hosts.ini playbooks/ntp.yml --check
```
