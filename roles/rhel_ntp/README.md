# Role: rhel_ntp

Instala e configura **NTP via chrony** em hosts **RHEL 7/8/9/10**. Servidores padrão: **NTP.br**.

## Variáveis principais (`defaults/main.yml`)

| Variável | Padrão | Descrição |
|---|---|---|
| `rhel_ntp_servers` | NTP.br (`a..d.st1.ntp.br`) | Servidores NTP |
| `rhel_ntp_pools` | `[]` | Pools NTP (opcional) |
| `rhel_ntp_server_options` | `iburst` | Opções por linha `server` |
| `rhel_ntp_timezone` | `America/Sao_Paulo` | Timezone (`""` = não alterar) |
| `rhel_ntp_allow_networks` | `[]` | Redes autorizadas a receber tempo |
| `rhel_ntp_makestep_threshold` / `_limit` | `1.0` / `3` | Correção abrupta inicial |
| `rhel_ntp_rtcsync` | `true` | Sincroniza o RTC |

## Dependências

Coleção **`community.general`** (módulo `timezone`) — já na EE padrão do AAP.

## Exemplo

Veja [`playbooks/ntp.yml`](../../playbooks/ntp.yml).
