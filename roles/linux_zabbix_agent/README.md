# Role: linux_zabbix_agent

Instala e configura o **zabbix-agent2** (repositório oficial Zabbix) em hosts
**RHEL 7/8/9/10**, **Debian** e **Ubuntu**. A role detecta a família do SO e configura
o repositório do jeito certo:

| Família | Repositório | Chave | Firewall |
|---|---|---|---|
| RedHat | RPM (`yum_repository`) | `RPM-GPG-KEY-ZABBIX` | `firewalld` |
| Debian / Ubuntu | APT (`apt_repository`) | `zabbix-official-repo.key` (`signed-by`) | `ufw` |

## Variáveis principais (`defaults/main.yml`)

| Variável | Padrão | Descrição |
|---|---|---|
| `linux_zabbix_agent_major` | `7.0` | Linha do repositório Zabbix (deve suportar o SO alvo) |
| `linux_zabbix_agent_package` | `zabbix-agent2` | `zabbix-agent2` ou `zabbix-agent` |
| `linux_zabbix_agent_server` | `127.0.0.1` | Server/proxy (checks passivos) |
| `linux_zabbix_agent_server_active` | `127.0.0.1` | Server/proxy (checks ativos) |
| `linux_zabbix_agent_hostname` | `{{ inventory_hostname }}` | Nome do host no Zabbix |
| `linux_zabbix_agent_host_metadata` | `""` | Metadados para autoregistration |
| `linux_zabbix_agent_tls_connect` / `_accept` | `unencrypted` | `unencrypted` ou `psk` |
| `linux_zabbix_agent_manage_firewall` | `false` | Abre a porta (firewalld/ufw) |

> **Compatibilidade de versão**: nem toda linha do Zabbix tem pacote para todo SO
> (ex.: RHEL 10 exige Zabbix ≥ 7.0). Ajuste `linux_zabbix_agent_major` conforme o host.
> As URLs de repositório/chave por família estão em `vars/RedHat.yml` e `vars/Debian.yml`.

## TLS/PSK

Para PSK: defina `..._tls_connect: psk`, `..._tls_accept: psk`, `..._tls_psk_identity`
e `..._tls_psk_secret` — **sempre com `ansible-vault`** (a task usa `no_log`).

## Dependências

- `ansible.posix` (módulo `firewalld`) — RHEL, se `manage_firewall=true`.
- `community.general` (módulo `ufw`) — Debian/Ubuntu, se `manage_firewall=true`.

## Exemplo

Veja [`playbooks/zabbix-agent.yml`](../../playbooks/zabbix-agent.yml).
