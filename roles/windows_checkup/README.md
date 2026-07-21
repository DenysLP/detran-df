# Role: windows_checkup

Check-up / inventário **read-only** de VMs **Windows**, para rodar **antes** de qualquer
mudança. Não altera nada (só coleta via WinRM + PowerShell). É o par da `linux_checkup`.
Ao final imprime um **relatório** legível e publica os dados via `set_stats` (para
encadear em Workflow no AAP). Cada item vira um bloco em `tasks/checks/*.yml`.

## Checklist coberto

| Item | Como verifica |
|---|---|
| **Login usuário/senha** | métodos de auth do WinRM (`WSMan:\localhost\Service\Auth` — Basic/Negotiate) |
| **Login chave SSH** | OpenSSH Server (`sshd`, porta 22, `administrators_authorized_keys`) |
| **Interface de rede** | `Get-NetAdapter` / `Get-NetIPConfiguration` |
| **Conectividade** | `Test-NetConnection` (gateway por padrão; alvos configuráveis) |
| **Disco e "LVM"** | `Get-Disk` + `Get-Volume` (+ `Get-StoragePool` = análogo a LVM) |
| **SO atualizado** | `win_updates -State searched` (Critical/Security/Rollups) |
| **Proxy** | `netsh winhttp show proxy` + WinINET (registro do usuário) |
| **Agent monitoramento** | serviços que casam `zabbix\|dynatrace\|splunk\|…` (configurável) |
| **Agent NGT** | serviço/produto Nutanix Guest Tools |
| **Regras de firewall** | `Get-NetFirewallProfile` + contagem de regras habilitadas |
| **Pacotes instalados** | registro de Uninstall (total + obrigatórios faltando) |
| **PBIS/domínio** | `Win32_ComputerSystem` (`PartOfDomain`/`Domain`) — no Windows o ingresso é nativo, **não** usa PBIS |
| **DNS e IP** | `Get-DnsClientServerAddress` + IPs/gateway das interfaces |

## Variáveis principais (`defaults/main.yml`)

| Variável | Padrão | Descrição |
|---|---|---|
| `windows_checkup_update_categories` | Critical/Security/Rollups | Categorias consultadas no `win_updates` |
| `windows_checkup_ping_targets` | `[]` (usa o gateway) | Alvos de `Test-NetConnection` (rede interna: evite 8.8.8.8) |
| `windows_checkup_monitor_pattern` | `zabbix\|dynatrace\|…` | Regex dos agentes de monitoramento |
| `windows_checkup_ngt_pattern` | `nutanix\|guest tools\|ngt` | Regex do NGT |
| `windows_checkup_required_programs` | `[]` | Programas obrigatórios (reporta faltantes) |
| `windows_checkup_expected_domain` | `""` | Domínio AD esperado (valida ingresso) |
| `windows_checkup_write_report` | `false` | Grava o relatório em arquivo no host |
| `windows_checkup_set_stats` | `true` | Publica os dados via `set_stats` (Workflow) |

## Requisitos

- Coleção **`ansible.windows`** (já em `collections/requirements.yml`).
- Conexão **WinRM** habilitada na VM (listener + firewall) e uma **credencial WinRM** no AAP.
- O usuário de conexão precisa ser **Administrador local** para várias consultas.

## Exemplo

Veja [`playbooks/windows-checkup.yml`](../../playbooks/windows-checkup.yml).
