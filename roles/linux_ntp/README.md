# Role: linux_ntp

Instala e configura **NTP via chrony** em hosts **RHEL 7/8/9/10**, **Debian** e **Ubuntu**.
Servidores padrão: **NTP.br**. A role detecta a família do SO (`os_family`) e carrega de
`vars/` o nome do serviço e os caminhos corretos:

| Família | Pacote | Serviço | Config |
|---|---|---|---|
| RedHat (RHEL/derivados) | `chrony` | `chronyd` | `/etc/chrony.conf` |
| Debian / Ubuntu | `chrony` | `chrony` | `/etc/chrony/chrony.conf` |

## Variáveis principais (`defaults/main.yml`)

| Variável | Padrão | Descrição |
|---|---|---|
| `linux_ntp_supported_families` | `[RedHat, Debian]` | Famílias aceitas |
| `linux_ntp_servers` | NTP.br (`a..d.st1.ntp.br`) | Servidores NTP |
| `linux_ntp_pools` | `[]` | Pools NTP (opcional) |
| `linux_ntp_server_options` | `iburst` | Opções por linha `server` |
| `linux_ntp_timezone` | `America/Sao_Paulo` | Timezone (`""` = não alterar) |
| `linux_ntp_allow_networks` | `[]` | Redes autorizadas a receber tempo |
| `linux_ntp_makestep_threshold` / `_limit` | `1.0` / `3` | Correção abrupta inicial |
| `linux_ntp_rtcsync` | `true` | Sincroniza o RTC |

> Variáveis específicas de família (pacote, serviço, `conf_path`, `driftfile`, `logdir`)
> ficam em `vars/RedHat.yml` e `vars/Debian.yml` e são carregadas automaticamente.

## Dependências

Coleção **`community.general`** (módulo `timezone`) — já na EE padrão do AAP.

## Exemplo

Veja [`playbooks/ntp.yml`](../../playbooks/ntp.yml).
