# detran-df — Roles Ansible (AAP)

Roles Ansible para automação de hosts **RHEL 7, 8, 9 e 10**, pensadas para uso no
**Ansible Automation Platform (AAP)** — cada role vira um building block de Job Templates.
As roles detectam a versão/gerenciador (`yum`/`dnf`/`dnf5`) automaticamente.

## Estrutura

```
.
├── collections/requirements.yml   # coleções (ansible.posix, community.general)
├── inventories/exemplo/hosts.ini  # inventário de exemplo (grupo [rhel])
├── playbooks/                     # 1 playbook de exemplo por role
│   ├── os-update.yml
│   ├── packages.yml
│   ├── ntp.yml
│   └── zabbix-agent.yml
└── roles/
    ├── rhel_os_update/            # update do SO (yum/dnf/dnf5) + reboot controlado
    ├── rhel_packages/             # instalação/remoção de pacotes e grupos
    ├── rhel_ntp/                  # NTP via chrony (padrão NTP.br)
    └── rhel_zabbix_agent/         # zabbix-agent2 (repo oficial + TLS/PSK)
```

## Roles

| Role | Função | Versões | Playbook |
|---|---|---|---|
| `rhel_os_update` | Atualiza o SO, reboot `auto/true/false` | 7/8/9/10 | `playbooks/os-update.yml` |
| `rhel_packages` | Instala/remove pacotes e grupos | 7/8/9/10 | `playbooks/packages.yml` |
| `rhel_ntp` | Instala e configura chrony (NTP.br) | 7/8/9/10 | `playbooks/ntp.yml` |
| `rhel_zabbix_agent` | Instala e configura zabbix-agent2 | 7/8/9/10 | `playbooks/zabbix-agent.yml` |

Cada role tem seu próprio `README.md` com a lista completa de variáveis. Todas aceitam
`<role>_supported_majors` para restringir as versões, e validam o host antes de agir.

## Como o multi-versão funciona

- **Gerenciador de pacotes**: `rhel_os_update` ramifica por `ansible_facts['pkg_mgr']`
  (`yum` → RHEL 7, `dnf` → RHEL 8/9, `dnf5` → RHEL 10). As demais roles usam o módulo
  genérico `ansible.builtin.package`.
- **Zabbix**: a URL do repo é montada com `distribution_major_version` do host.
- **chrony**: padrão de NTP em todas as versões cobertas.

## Uso no AAP

1. **Project** → aponte para este repositório. O AAP lê `collections/requirements.yml`.
2. **Inventory** → hosts RHEL no grupo `rhel`.
3. **Job Template** → um por playbook em `playbooks/`; exponha variáveis via **Survey**.
4. Segredos (PSK do Zabbix) → **Vault Credential** / `ansible-vault`.

## Uso local (teste)

```bash
ansible-galaxy collection install -r collections/requirements.yml
ansible-playbook -i inventories/exemplo/hosts.ini playbooks/ntp.yml --check
```
