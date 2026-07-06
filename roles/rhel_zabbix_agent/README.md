# Role: rhel_zabbix_agent

Instala e configura o **zabbix-agent2** (repositório oficial Zabbix) em hosts **RHEL 7/8/9/10**.
A URL do repositório é montada com a versão maior detectada do host.

## Variáveis principais (`defaults/main.yml`)

| Variável | Padrão | Descrição |
|---|---|---|
| `rhel_zabbix_agent_major` | `7.0` | Linha do repositório Zabbix (deve suportar o RHEL alvo) |
| `rhel_zabbix_agent_package` | `zabbix-agent2` | `zabbix-agent2` ou `zabbix-agent` |
| `rhel_zabbix_agent_server` | `127.0.0.1` | Server/proxy (checks passivos) |
| `rhel_zabbix_agent_server_active` | `127.0.0.1` | Server/proxy (checks ativos) |
| `rhel_zabbix_agent_hostname` | `{{ inventory_hostname }}` | Nome do host no Zabbix |
| `rhel_zabbix_agent_host_metadata` | `""` | Metadados para autoregistration |
| `rhel_zabbix_agent_tls_connect` / `_accept` | `unencrypted` | `unencrypted` ou `psk` |
| `rhel_zabbix_agent_manage_firewall` | `false` | Abre a porta no firewalld |

> **Compatibilidade de versão**: nem toda linha do Zabbix tem pacote para todo RHEL (ex.: RHEL 10 exige Zabbix ≥ 7.0). Ajuste `rhel_zabbix_agent_major` conforme o host.

## TLS/PSK

Para PSK: defina `..._tls_connect: psk`, `..._tls_accept: psk`, `..._tls_psk_identity`
e `..._tls_psk_secret` — **sempre com `ansible-vault`** (a task usa `no_log`).

## Dependências

`ansible.posix` (módulo `firewalld`) apenas se `rhel_zabbix_agent_manage_firewall=true`.

## Exemplo

Veja [`playbooks/zabbix-agent.yml`](../../playbooks/zabbix-agent.yml).
