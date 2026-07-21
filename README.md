# aap-automacoes — Automação DETRAN-DF (AAP)

Automação Ansible para o DETRAN-DF, pensada para uso no **Ansible Automation Platform
(AAP)**. Cobre três frentes:

- **Roles Linux** (**RHEL 7/8/9/10**, **Debian**, **Ubuntu**) — detectam a família do SO
  (`os_family`) e carregam de `vars/<família>.yml` o pacote/serviço/caminho corretos.
- **Role Windows** (`windows_checkup`) — check-up read-only via **WinRM**.
- **Playbooks Nutanix** — validação e inventário do provider **Prism Central** via API REST.

Cada role/playbook é um building block de Job Templates.

## Estrutura

```
.
├── collections/requirements.yml   # ansible.posix, community.general, ansible.windows, nutanix.ncp
├── inventories/exemplo/hosts.ini  # inventário de exemplo (grupo [linux])
├── playbooks/
│   ├── checkup.yml                # check-up READ-ONLY Linux
│   ├── windows-checkup.yml        # check-up READ-ONLY Windows (WinRM)
│   ├── os-update.yml              # exemplos das roles Linux
│   ├── packages.yml
│   ├── ntp.yml
│   ├── zabbix-agent.yml
│   ├── nutanix-validate.yml       # ── Nutanix: valida conexão
│   ├── nutanix-healthcheck.yml    #    storage/CPU/memória + alertas críticos
│   ├── nutanix-list-storage.yml   #    lista storage containers (uso %)
│   ├── nutanix-list-nodes.yml     #    lista nodes (capacidade %)
│   ├── nutanix-list-templates.yml #    lista VM Templates + Imagens (filtro por nome/tipo)
│   ├── nutanix-list-alerts.yml    #    lista alertas ativos
│   ├── nutanix-check-template.yml #    guard pré-criação: template existe?
│   └── nutanix-check-vm-name.yml  #    guard pré-criação: nome de VM livre?
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

## Playbooks avulsos (sem role)

| Playbook | Função |
|---|---|
| `playbooks/nutanix-validate.yml` | Valida a conexão com o provider **Nutanix** (Prism Central) — autenticação + lista clusters. |
| `playbooks/nutanix-healthcheck.yml` | Health-check: espaço de storage, uso de CPU/memória por cluster e alertas críticos (com limiares). |
| `playbooks/nutanix-check-template.yml` | Guard de pré-criação: confirma que um **template** existe antes de criar VM a partir dele. |
| `playbooks/nutanix-check-vm-name.yml` | Guard de pré-criação: confirma que o **nome da VM** está livre (evita duplicidade). |
| `playbooks/nutanix-list-storage.yml` | Lista os **storage containers** com uso/capacidade (%) por container. |
| `playbooks/nutanix-list-nodes.yml` | Lista os **nodes** com CPU/memória (uso e livre %), cores e nº de VMs — onde há folga para criar VMs. |
| `playbooks/nutanix-list-templates.yml` | Lista **VM Templates + Imagens** (DISK_IMAGE/ISO_IMAGE — o que se usa p/ criar VMs). Filtra por nome (`nutanix_template_name`) e por tipo (`nutanix_image_types`). |
| `playbooks/nutanix-list-alerts.yml` | Lista os **alertas ativos** (não resolvidos), por severidade. |

> **Nutanix — credenciais** (nunca no repo; via Credencial do AAP / extra_vars / vault):
> `nutanix_host` (IP **sem** porta), `nutanix_port` (9440), `nutanix_username`, `nutanix_password`.
> Rodam em `hosts: localhost` chamando a API REST v3 / vmm v4.0.a1 via `ansible.builtin.uri`
> (os módulos `nutanix.ncp` v4 `_v2` dão 404 neste Prism Central). Variáveis extras úteis:
> `nutanix_template_name` (guards e listagem), `nutanix_vm_name` (guard de duplicidade),
> `nutanix_alert_severities`, `nutanix_image_types`. Todos publicam resultado via `set_stats`
> (para encadear em Workflow) e têm gate opcional.

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
2. **Inventory** → hosts no grupo `linux` (RHEL/Debian/Ubuntu) e `windows`; os playbooks
   Nutanix rodam em `localhost`.
3. **Credenciais**:
   - Linux → **Machine** (SSH), com `become`.
   - Windows → **Machine** (WinRM): usuário do domínio + `ansible_connection=winrm`.
   - Nutanix → **Custom Credential Type** que injeta `nutanix_host/port/username/password`
     como extra_vars (senha fica `$encrypted$`).
4. **Job Template** → um por playbook em `playbooks/`; exponha variáveis via **Survey**
   (ex.: `nutanix_vm_name`, `nutanix_template_name`, escopo de update).
5. Segredos (PSK do Zabbix, senha Nutanix) → **Vault Credential** / `ansible-vault`.

## Uso local (teste)

```bash
ansible-galaxy collection install -r collections/requirements.yml

# Linux (grupo [linux] do inventário)
ansible-playbook -i inventories/exemplo/hosts.ini playbooks/checkup.yml

# Nutanix (localhost) — credenciais via arquivo (nunca commitar) ou -e
ansible-playbook playbooks/nutanix-list-storage.yml -e @nutanix-creds.yml
```
