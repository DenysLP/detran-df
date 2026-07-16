# Role: linux_checkup

Check-up / inventário **read-only** do host, para rodar **antes** de qualquer outra role.
Não altera nada no sistema (comandos com `changed_when: false`, checagem de pacotes/serviços
via `package_facts`/`service_facts`). Suporta **RHEL 7/8/9/10**, **Debian** e **Ubuntu**.

## O que coleta

| Item | RHEL | Debian/Ubuntu |
|---|---|---|
| **Atualizações pendentes** (total + segurança) | `dnf/yum check-update [--security]` | `apt-get -s dist-upgrade` |
| **Reboot pendente** | `needs-restarting -r` | `/var/run/reboot-required` |
| **Proxy** | env + `/etc/dnf`,`/etc/yum*` | env + `/etc/apt/apt.conf*` |
| **Agentes** (Zabbix, Filebeat, Node Exporter, Falcon…) | pacote + serviço | pacote + serviço |
| **NTP sincronizado** | `timedatectl` | `timedatectl` |
| **Inventário geral** | SO, kernel, uptime, CPU/RAM, disco `/`, IP, FQDN, SELinux | idem |

Ao final imprime um **relatório** (`debug`). Opcionalmente grava em arquivo no host.

## Variáveis principais (`defaults/main.yml`)

| Variável | Padrão | Descrição |
|---|---|---|
| `linux_checkup_refresh_cache` | `true` | Atualiza metadados/cache antes de contar updates |
| `linux_checkup_agents` | Zabbix2/Zabbix/Filebeat/Node Exporter/Falcon | Lista `{name, package, service}` a procurar |
| `linux_checkup_write_report` | `false` | Grava o relatório em arquivo (única tarefa que escreve) |
| `linux_checkup_report_path` | `/var/log/linux_checkup.txt` | Caminho do relatório (se habilitado) |

> Os caminhos varridos por proxy ficam em `vars/RedHat.yml` / `vars/Debian.yml`.
> A contagem de updates é aproximada (parse do output do gerenciador) — serve de
> indicador, não de número exato para SLA.

## Exemplo

Veja [`playbooks/checkup.yml`](../../playbooks/checkup.yml). No AAP, use como **primeiro**
Job Template de um Workflow, antes das roles que aplicam mudança.
