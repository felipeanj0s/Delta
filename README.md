# Projeto Delta: Provisionamento Automatizado de Zabbix Proxy com Ansible

![Ansible Version](https://img.shields.io/badge/ansible--core-2.15%2B-blue.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)

## Descrição

Este projeto utiliza Ansible para automatizar a implantação e configuração completa de um Zabbix Proxy em servidores baseados em Debian 12 (Bookworm). A automação foi projetada para ser executada localmente no host de destino e abrange a configuração de rede, hardening de segurança e a instalação dos serviços Zabbix.

## Funcionalidades

- **Instalação de Dependências**: Garante que as collections Ansible necessárias (`community.zabbix`) sejam instaladas antes da execução principal.
- **Carregamento de Contexto Dinâmico**: As variáveis de configuração (rede, hostname) são carregadas dinamicamente com base no grupo de inventário do host.
- **Configuração de Rede e Segurança**: A role `net_security` executa as seguintes ações:
  - Define o `hostname` do servidor.
  - Configura a interface de rede estática via `/etc/network/interfaces`.
  - Instala e configura o firewall UFW com regras específicas para SSH e Zabbix.
  - Instala e configura o Fail2Ban para proteção contra ataques de força bruta no SSH.
  - Realiza o hardening básico do serviço SSH.
- **Instalação dos Serviços Zabbix**:
  - **`zabbix_proxy`**: Instala e configura o serviço do Zabbix Proxy a partir dos repositórios oficiais.
  - **`zabbix_agent`**: Instala e configura o Zabbix Agent 2 para monitoramento do próprio host do proxy.
- **Registro no Servidor Zabbix**: A automação tenta registrar o proxy no Zabbix Server, mas possui limitações (ver abaixo).

## Estrutura do Projeto

.
├── group_vars/
│   ├── all.yml                       # Variáveis globais
│   └── pops_configs/
│       └── ce.yml                    # Variáveis específicas por POP (ex: Ceará)
│
├── roles/
│   ├── net_security/                 # Configuração de rede e segurança
│   ├── setup_context/                # Carregamento dinâmico de variáveis
│   ├── zabbix_agent/                 # Instalação e configuração do Agent 2
│   ├── zabbix_proxy/                 # Instalação e configuração do Proxy
│   └── zabbix_server_register_proxy/ # Registro na API do Zabbix Server
│
├── hosts                             # Arquivo de inventário Ansible
└── prov_zbxproxy.yml                 # Playbook principal


## Pré-requisitos

As seguintes ferramentas devem estar instaladas no servidor de destino antes da execução:

-   Sistema Operacional: **Debian 12 (Bookworm)**
-   Um usuário com permissões `sudo`.
-   `git`
-   `python3-pip`

## Configuração

1.  **Variáveis Globais (`group_vars/all.yml`)**: Ajuste as variáveis que são comuns a todos os ambientes, como `zabbix_server_ip`, `zabbix_server_url` e `zabbix_api_token`.
2.  **Variáveis de Localidade (`group_vars/pops_configs/`)**: Crie ou edite o arquivo YAML correspondente à sua localidade (ex: `ce.yml`). Preencha as variáveis de rede (`pop_network_interface`, `pop_network_ipv4_address`, `pop_network_ipv4_netmask`, etc.) e o `zabbix_proxy_hostname`.

## Instruções de Uso

A automação foi projetada para ser executada localmente no servidor que será provisionado.

1.  **Clone o repositório no servidor de destino:**
    ```bash
    git clone [https://github.com/felipeanj0s/Delta.git](https://github.com/felipeanj0s/Delta.git)
    cd Delta/
    ```

2.  **Instale o Ansible:**
    ```bash
    # (Opcional) Remova versões antigas se existirem
    sudo apt remove -y ansible ansible-core && sudo apt autoremove -y

    # Instale a versão mais recente via pip
    sudo pip3 install ansible --break-system-packages

    # Limpe o cache do terminal para encontrar o novo executável
    hash -r
    ```

3.  **Execute o Playbook:**
    Substitua `ce` no comando abaixo pelo grupo do seu host no arquivo `hosts`.
    ```bash
    ansible-playbook -i hosts prov_zbxproxy.yml --limit ce --ask-become-pass
    ```

## Limitações Conhecidas

-   **Registro da Interface do Proxy**: Devido a uma inconsistência na API do Zabbix Server alvo, a automação cria o proxy com sucesso, mas não consegue adicionar a sua interface de rede (endereço IP).
-   **Ação Manual Necessária**: Após a execução bem-sucedida do playbook, é necessário acessar a interface web do Zabbix (`Administration > Proxies`), selecionar o proxy recém-criado e adicionar manualmente seu endereço IP no campo "Proxy address".

## Autor

-   **Felipe Anjos | PoP-CE**