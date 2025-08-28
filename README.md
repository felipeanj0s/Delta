Com certeza. Entendido.

"Atualizar tudo do zero" significa consolidar todas as correções e aprendizados da nossa longa jornada de depuração em uma versão final, limpa e funcional de todos os arquivos importantes do projeto.

Abaixo está a versão final e corrigida de cada arquivo chave. Você pode usar este material como a "versão de ouro" do seu projeto.

-----

### 1\. `README.md` (Completo e Atualizado)

````markdown
# Projeto Delta: Provisionamento Automatizado de Zabbix Proxy com Ansible

![Ansible Version](https://img.shields.io/badge/ansible--core-2.15%2B-blue.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)

## Descrição

Este projeto utiliza Ansible para automatizar a implantação e configuração completa de um Zabbix Proxy em servidores baseados em Debian 12 (Bookworm). A automação foi projetada para ser executada localmente no host de destino e abrange a configuração de rede, hardening de segurança e a instalação dos serviços Zabbix.

## Estrutura do Projeto

| Arquivo / Diretório                  | Descrição                                                                                                   |
| ------------------------------------ | ----------------------------------------------------------------------------------------------------------- |
| `prov_zbxproxy.yml`                  | Playbook principal que orquestra a execução de todas as roles na ordem correta.                               |
| `hosts`                              | Arquivo de inventário que define os hosts de destino e seus grupos (ex: `[ce]`).                                |
| `group_vars/all.yml`                 | Contém as variáveis globais, aplicadas a todos os hosts no inventário (IP do Zabbix Server, token, etc.).       |
| `group_vars/pops_configs/`           | Diretório que armazena os arquivos de variáveis específicas para cada localidade (POP).                         |
| `roles/`                             | Diretório principal que contém todas as roles modulares da automação.                                         |
| `roles/setup_context/`               | Role responsável por carregar o arquivo de variáveis correto do POP com base no inventário.                    |
| `roles/net_security/`                | Role que aplica configurações essenciais de rede e segurança (UFW, Fail2Ban, SSH).                             |
| `roles/zabbix_proxy/`                | Role que instala e configura o serviço do Zabbix Proxy no host.                                               |
| `roles/zabbix_agent/`                | Role que instala e configura o Zabbix Agent 2 no host.                                                        |
| `roles/zabbix_server_register_proxy/` | Role que se comunica com a API do Zabbix Server para registrar o proxy.                                       |

## Pré-requisitos

As seguintes ferramentas devem estar instaladas no servidor de destino antes da execução:

-   Sistema Operacional: **Debian 12 (Bookworm)**
-   Acesso de um usuário com permissões `sudo`.
-   `git`
-   `python3-pip`
-   `ansible` (recomenda-se a versão mais recente instalada via `pip`)

## Configuração

1.  **Variáveis Globais (`group_vars/all.yml`)**: Ajuste as variáveis que são comuns a todos os ambientes, como `zabbix_server_ip`, `zabbix_server_url` e `zabbix_api_token`.
2.  **Variáveis de Localidade (`group_vars/pops_configs/`)**: Crie ou edite o arquivo YAML correspondente à sua localidade (ex: `ce.yml`). Preencha as variáveis de rede (`pop_network_interface`, `pop_network_ipv4_address`, `pop_network_ipv4_netmask`, etc.) e o `zabbix_proxy_hostname`.

## Instruções de Uso

A automação foi projetada para ser executada localmente no servidor que será provisionado.

**1. Clone o Repositório**

Clone o projeto para o servidor de destino.
```bash
git clone [https://github.com/felipeanj0s/Delta.git](https://github.com/felipeanj0s/Delta.git)
cd Delta/
````

**2. Execute o Playbook**

Use o comando abaixo para iniciar a automação. Lembre-se de substituir `ce` pelo grupo correto do seu host, conforme definido no arquivo `hosts`.

```bash
ansible-playbook -i hosts prov_zbxproxy.yml --limit ce --ask-become-pass
```

##### Análise do Comando de Execução

A tabela abaixo detalha cada parâmetro do comando:

| Parâmetro           | Descrição                                                                                                                              |
| ------------------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| `ansible-playbook`  | O comando executável do Ansible que interpreta e executa os playbooks.                                                                 |
| `-i hosts`          | A flag `-i` (ou `--inventory`) especifica qual arquivo de inventário usar. Este arquivo lista os hosts onde a automação será aplicada.   |
| `prov_zbxproxy.yml` | O nome do arquivo do playbook principal que será executado.                                                                            |
| `--limit ce`        | Restringe a execução do playbook apenas aos hosts pertencentes ao grupo `[ce]` dentro do inventário. Essencial para o contexto do POP. |
| `-K`                | Forma curta de `--ask-become-pass`. Solicita a senha do `sudo` no início da execução, necessária para tarefas que exigem privilégios.   |
| `-v`, `-vv`, `-vvv` | Controla o nível de detalhes (verbosidade) da saída.                                                                                   |

## Limitações Conhecidas

  - **Registro da Interface do Proxy**: Devido a uma inconsistência na API do Zabbix Server alvo, a automação cria o proxy com sucesso, mas não consegue adicionar a sua interface de rede (endereço IP).
  - **Ação Manual Necessária**: Após a execução bem-sucedida do playbook, é necessário acessar a interface web do Zabbix (`Administration > Proxies`), selecionar o proxy recém-criado e adicionar manualmente seu endereço IP no campo "Proxy address".

## Autor

  - **Felipe Anjos**

<!-- end list -->

````

---

### 2. `prov_zbxproxy.yml` (Playbook Principal)

```yaml
# ============================================================================================================================
# Provisionamento do Projeto - POP-CE
# Arquivo: prov_zbxproxy.yml
# Descrição: Este arquivo é responsável por provisionar a instalação do Zabbix Proxy, Zabbix Agent e suas configurações.
# ============================================================================================================================
---
- name: Configurar Zabbix Proxy e Agent
  hosts: all
  become: yes

  # --- PRE-TASKS: Verificação e Instalação de Dependências ---
  pre_tasks:
    - name: Verificar se a collection community.zabbix está instalada
      ansible.builtin.command: ansible-galaxy collection list community.zabbix
      register: zabbix_collection_check
      changed_when: false
      failed_when: false
      connection: local
      run_once: true

    - name: Instalar a collection community.zabbix se necessário
      ansible.builtin.command: ansible-galaxy collection install community.zabbix
      when: zabbix_collection_check.rc != 0
      connection: local
      run_once: true

  # --- TASKS: Execução das Roles ---
  tasks:
    - name: Executar role setup_context
      include_role:
        name: setup_context

    - name: Executar role net_security
      include_role:
        name: net_security

    - name: Executar role zabbix_proxy
      include_role:
        name: zabbix_proxy

    - name: Executar role zabbix_agent
      include_role:
        name: zabbix_agent
        
    - name: Executar role zabbix_server_register_proxy
      include_role:
        name: zabbix_server_register_proxy
````

-----

### 3\. `group_vars/pops_configs/ce.yml` (Exemplo de Variáveis)

```yaml
# ==============================================================================
# Variáveis Específicas do Projeto - POP-CE
# Arquivo: group_vars/pops_configs/ce.yml
# ==============================================================================

# --- Configuração do Nome para o Zabbix Proxy e Agent ---
zabbix_proxy_hostname: "ce-zabbix-rnp-ger-proxy01"
zabbix_agent_hostname: "{{ zabbix_proxy_hostname }}"

# --- Informações de rede do PoP-CE ---
pop_network_interface: "enX2"
# Variável principal com CIDR para uso geral (regras de firewall, etc.)
pop_network_ipv4_address: "192.168.0.17/24"
# Variável separada para a máscara, usada pelo template /etc/network/interfaces
pop_network_ipv4_netmask: "255.255.255.0"
pop_network_ipv6_address: ""
pop_network_ipv4_gateway: "192.168.0.9"
pop_network_ipv6_gateway: ""
pop_network_dns_list:
  - "200.129.0.41"
  - "200.129.0.42"
  - "200.19.16.53"
  - "200.137.53.53"

# --- Configurações de Segurança Especificas para o PoP-CE ---
ssh_port: 25085
```

-----

### 4\. `roles/net_security/templates/interfaces.j2` (Template de Rede)

```jinja
# ==============================================================================
# Template de Configuração de Rede - Debian 12
# Arquivo: roles/net_security/templates/interfaces.j2
# VERSÃO CORRIGIDA
# ==============================================================================

source /etc/network/interfaces.d/*

# A interface de loopback
auto lo
iface lo inet loopback

# A interface de rede primária
auto {{ pop_network_interface }}

# Configuração IPv4
iface {{ pop_network_interface }} inet static
  # Extrai apenas o IP da variável principal (ex: "192.168.0.17")
  address {{ pop_network_ipv4_address.split('/')[0] }}
  # Usa a nova variável para a máscara de rede
  netmask {{ pop_network_ipv4_netmask }}
  gateway {{ pop_network_ipv4_gateway }}
  dns-nameservers {{ pop_network_dns_list | join(' ') }}

{# --- Configuração IPv6 Condicional --- #}
{% if pop_network_ipv6_address is defined and pop_network_ipv6_address %}
iface {{ pop_network_interface }} inet6 static
  address {{ pop_network_ipv6_address.split('/')[0] }}
  gateway {{ pop_network_ipv6_gateway }}
{% endif %}
```

-----

### 5\. `roles/net_security/templates/motd_info.j2` (Script do MOTD)

```jinja
{% raw %}
#!/bin/bash
# ======================================================================
# GT Monitoramento 2025
# Script de Geração de Mensagem do Dia (MOTD) - Clean & Modern Edition
# ======================================================================

set -euo pipefail
IFS=$'\n\t'

# --- Cores ---
RED="\033[1;31m"
GREEN="\033[1;32m"
YELLOW="\033[1;33m"
BLUE="\033[1;34m"
CYAN="\033[1;36m"
RESET="\033[0m"

# --- Função para checar status de serviços ---
get_status() {
    local service=$1
    if systemctl list-unit-files | grep -q "^${service}.service"; then
        local state
        state=$(systemctl is-active "$service" || echo "inactive")
        case "$state" in
            active) echo -e "${GREEN}✔ ativo${RESET}" ;;
            inactive) echo -e "${RED}✘ inativo${RESET}" ;;
            *) echo -e "${YELLOW}${state}${RESET}" ;;
        esac
    else
        echo -e "${YELLOW}não instalado${RESET}"
    fi
}

# --- Variáveis do sistema ---
DEBIAN_VERSION=$(grep PRETTY_NAME /etc/os-release | cut -d= -f2 | tr -d '"')
HOSTNAME=$(hostname)
IPV4_ADDRS=($(hostname -I | tr ' ' '\n' | grep -v '^127\.0\.0\.1$'))
ZABBIX_AGENT="zabbix-agent2"
ZABBIX_PROXY="zabbix-proxy"
LAST_LOGIN=$(last -n 2 -w | awk 'NR==2 {print $1, $2, $3, $4, $5, $6}')
UPTIME=$(uptime -p | sed 's/up //')
USERS=$(who | wc -l)
MEMORY=$(free -h | awk '/^Mem:/ {printf "%s / %s (%.0f%%)\n",$3,$2,$3/$2*100}')
DISK_ROOT=$(df -h / | awk 'NR==2 {print $3 " / " $2 " (" $5 ")"}')
ZABBIX_AGENT_STATUS=$(get_status "$ZABBIX_AGENT")
ZABBIX_PROXY_STATUS=$(get_status "$ZABBIX_PROXY")

# --- Cabeçalho ---
clear
echo -e "\n${BLUE}################################################################${RESET}"
echo -e "        ${CYAN}💻 Bem-vindo ao servidor: ${GREEN}$HOSTNAME${RESET}"
echo -e "${BLUE}################################################################${RESET}\n"

# --- Informações detalhadas ---
echo -e " ${YELLOW}➡ Versão do SO:${RESET}           $DEBIAN_VERSION"
if [ ${#IPV4_ADDRS[@]} -eq 1 ]; then
    echo -e " ${YELLOW}➡ Endereço IPv4:${RESET}       ${IPV4_ADDRS[0]}"
else
    echo -e " ${YELLOW}➡ Endereços IPv4:${RESET}"
    for ip in "${IPV4_ADDRS[@]}"; do
        echo -e "       • $ip"
    done
fi
echo -e " ${YELLOW}➡ Último login:${RESET}          ${LAST_LOGIN:-N/A}"
echo -e " ${YELLOW}➡ Uptime:${RESET}                $UPTIME"
echo -e " ${YELLOW}➡ Usuários conectados:${RESET}  $USERS"
echo -e " ${YELLOW}➡ Memória usada:${RESET}         $MEMORY"
echo -e " ${YELLOW}➡ Disco raiz:${RESET}            $DISK_ROOT\n"
echo -e " ${YELLOW}➡ Zabbix Agent:${RESET}          $ZABBIX_AGENT_STATUS"
echo -e " ${YELLOW}➡ Zabbix Proxy:${RESET}          $ZABBIX_PROXY_STATUS\n"

# --- Rodapé ---
echo -e "${BLUE}################################################################${RESET}"
echo -e "   ${GREEN}Acesso Concedido.${RESET} ${CYAN}Toda atividade está sendo monitorada.${RESET}"
echo -e "${BLUE}################################################################${RESET}\n"
{% endraw %}
```

-----

### 6\. `roles/zabbix_agent/templates/zabbix_agent2.conf.j2` (Template do Agente)

```jinja
# ==============================================================================
# Template de Configuração do Zabbix Agent
# Arquivo: roles/zabbix_agent/templates/zabbix_agent2.conf.j2
# ==============================================================================

### GENERAL
PidFile={{ zabbix_agent_pidfile }}
LogFile={{ zabbix_agent_logfile }}
LogType={{ zabbix_agent_logtype }}
LogFileSize={{ zabbix_agent_logfile_size }}
DebugLevel={{ zabbix_agent_debug_level }}
SourceIP={{ zabbix_agent_source_ip }}

### PASSIVE CHECKS
ListenPort={{ zabbix_agent_listenport }}
Server={{ zabbix_agent_server }}

### ACTIVE CHECKS
ServerActive={{ zabbix_agent_serveractive_list | join(',') }}
Hostname={{ zabbix_agent_hostname }}
RefreshActiveChecks={{ zabbix_agent_refresh_active_checks }}
BufferSend={{ zabbix_agent_buffer_send }}
BufferSize={{ zabbix_agent_buffer_size }}
EnablePersistentBuffering={{ zabbix_agent_enable_persistent_buffering }}
{% if zabbix_agent_enable_persistent_buffering == 1 or zabbix_agent_enable_persistent_buffering == '1' %}
PersistentBufferPeriod={{ zabbix_agent_persistent_buffer_period }}
PersistentBufferFile={{ zabbix_agent_persistent_buffer_file }}
{% endif %}

### ADVANCED
Include={{ zabbix_agent_include_dir }}
#ControlSocket={{ zabbix_agent_control_socket }}

### TLS
TLSConnect={{ zabbix_agent_tls_connect }}
TLSAccept={{ zabbix_agent_tls_accept }}
TLSPSKIdentity={{ zabbix_agent_psk_identity }}
TLSPSKFile={{ zabbix_agent_psk_file }}

### PLUGINS
Plugins.Log.MaxLinesPerSecond={{ zabbix_agent_plugins_log_max_lines }}
#Plugins.Docker.Endpoint={{ zabbix_agent_plugins_docker_endpoint }}
#Plugins.Docker.Timeout={{ zabbix_agent_plugins_docker_timeout }}
```

-----

### 7\. `roles/zabbix_server_register_proxy/tasks/main.yml` (Registro na API)

```yaml
# ======================================================================
# GT Monitoramento 2025
# Arquivo: roles/zabbix_server_register_proxy/tasks/main.yml
# VERSÃO DE CONTORNO (WORKAROUND) PARA API INCONSISTENTE
# ======================================================================
---
# Este código cria o proxy sem a interface para contornar a API inconsistente do Zabbix Server.

- name: Garantir que o PIP esteja instalado
  become: yes
  ansible.builtin.apt:
    name: python3-pip
    state: present
    update_cache: yes

- name: Garantir que a biblioteca zabbix-api esteja instalada
  ansible.builtin.pip:
    name: zabbix-api
    state: present
    extra_args: --break-system-packages

- name: API | Verificar se o Proxy já existe
  ansible.builtin.uri:
    url: "{{ zabbix_server_url }}/api_jsonrpc.php"
    method: POST
    headers:
      Content-Type: "application/json-rpc"
      Authorization: "Bearer {{ zabbix_api_token }}"
    body_format: json
    body:
      jsonrpc: "2.0"
      method: "proxy.get"
      params:
        output: "extend"
        filter:
          name: ["{{ zabbix_proxy_hostname }}"]
      id: 1
    validate_certs: no
    follow_redirects: all
  register: zabbix_proxy_get_response
  failed_when: "'error' in zabbix_proxy_get_response.json"

- name: API | Criar o Proxy (sem interface) se não existir
  ansible.builtin.uri:
    url: "{{ zabbix_server_url }}/api_jsonrpc.php"
    method: POST
    headers:
      Content-Type: "application/json-rpc"
      Authorization: "Bearer {{ zabbix_api_token }}"
    body_format: json
    body:
      jsonrpc: "2.0"
      method: "proxy.create"
      params:
        name: "{{ zabbix_proxy_hostname }}"
        description: "Proxy {{ zabbix_proxy_hostname }} provisionado via Ansible"
        operating_mode: "{{ zabbix_proxy_mode | default(0) | int }}"
        tls_connect: "{{ zabbix_proxy_tls_connect | default(1) | int }}"
        tls_accept: "{{ zabbix_proxy_tls_accept | default(1) | int }}"
        tls_psk_identity: "{{ proxy_api_psk_identity }}"
        tls_psk: "{{ proxy_api_psk_key }}"
      id: 1
    validate_certs: no
    follow_redirects: all
  register: zabbix_proxy_create_response
  failed_when: "'error' in zabbix_proxy_create_response.json"
  when: (zabbix_proxy_get_response.json.result | default([])) | length == 0

- name: Final | Exibir resultado da operação
  ansible.builtin.debug:
    msg: "AVISO: Proxy '{{ zabbix_proxy_hostname }}' criado/verificado com sucesso. A interface (IP) deve ser configurada manualmente na interface web do Zabbix."
```