# Role: linux_checkup

Check-up / inventário **read-only** do host, para rodar **antes** de qualquer outra role.
Não altera nada no sistema (comandos com `changed_when: false`, checagem de pacotes/serviços
via `package_facts`/`service_facts`). Suporta **RHEL 7/8/9/10**, **Debian** e **Ubuntu**.

## O que coleta (checklist)

Cada item vira um bloco em `tasks/checks/*.yml`:

| Item | Como verifica |
|---|---|
| **Login usuário/senha** | `sshd -T` → `PasswordAuthentication` |
| **Login chave SSH** | `sshd -T` → `PubkeyAuthentication` + `authorized_keys` do usuário |
| **Interface de rede** | facts: interfaces, IPs v4, rota default |
| **Conectividade** | ping no gateway + alvos TCP configuráveis (`wait_for`) |
| **Disco e LVM** | `mounts` (uso por FS) + `vgs`/`lvs` |
| **SO atualizado** | `dnf/yum check-update [--security]` / `apt-get -s dist-upgrade` + reboot pendente |
| **Proxy** | env + `/etc/dnf`,`/etc/yum*` (RHEL) / `/etc/apt/apt.conf*` (Debian) |
| **Firewall** | `firewall-cmd` (RHEL) / `ufw status` (Debian) + contagem nftables |
| **Pacotes instalados** | total (`package_facts`) + obrigatórios faltando |
| **Agent monitoramento** | Zabbix, Filebeat, Node Exporter, Falcon… (pacote + serviço) |
| **Agent NGT** | Nutanix Guest Tools (pacote/serviço + caminhos `/usr/local/nutanix`) |
| **PBIS / domínio** | `domainjoin-cli query` (PBIS) ou `realm list` / serviço SSSD |
| **DNS e IP** | nameservers/search, IPs, resolução de nome de teste |

Cobre **RHEL 7/8/9/10**, **Debian** e **Ubuntu** (detecção por família).
Ao final imprime um **relatório** (`debug`). Opcionalmente grava em arquivo no host.

## Variáveis principais (`defaults/main.yml`)

| Variável | Padrão | Descrição |
|---|---|---|
| `linux_checkup_refresh_cache` | `true` | Atualiza metadados/cache antes de contar updates |
| `linux_checkup_agents` | Zabbix2/Zabbix/NGT/Filebeat/Node Exporter/Falcon | Lista `{name, package, service}` a procurar |
| `linux_checkup_ping_gateway` | `true` | Faz ping no gateway padrão |
| `linux_checkup_tcp_targets` | `[]` | Alvos TCP a testar: `{name, host, port}` |
| `linux_checkup_dns_test_name` | `""` | Nome a resolver p/ testar DNS (vazio = domínio detectado) |
| `linux_checkup_required_packages` | `[]` | Pacotes que devem estar instalados (reporta faltantes) |
| `linux_checkup_expected_domain` | `""` | Domínio AD esperado (valida ingresso) |
| `linux_checkup_write_report` | `false` | Grava o relatório em arquivo (única tarefa que escreve) |
| `linux_checkup_report_path` | `/var/log/linux_checkup.txt` | Caminho do relatório (se habilitado) |

> Os caminhos varridos por proxy ficam em `vars/RedHat.yml` / `vars/Debian.yml`.
> A contagem de updates é aproximada (parse do output do gerenciador) — serve de
> indicador, não de número exato para SLA.

## Exemplo

Veja [`playbooks/checkup.yml`](../../playbooks/checkup.yml). No AAP, use como **primeiro**
Job Template de um Workflow, antes das roles que aplicam mudança.
