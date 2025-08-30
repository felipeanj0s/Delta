
# Provisionamento Automatizado de Zabbix Proxy com Ansible

  

Este projeto automatiza, via **Ansible**, a implantação e configuração completa de um **Zabbix Proxy** e **Zabbix Agent 2** em servidores **Debian 12 (Bookworm)**, desde o hardening inicial até o registro final na plataforma Zabbix.

## 📌 Principais Características

  - **Execução Local:** O playbook é executado no próprio servidor de destino, simplificando o processo.
  - **Automação de Ponta a Ponta:** Provisiona desde a configuração de rede e segurança até a instalação e registro dos serviços Zabbix.
  - **Idempotente:** O playbook pode ser executado várias vezes com segurança, garantindo sempre o estado final desejado.
  - **Configuração por Localidade:** Utiliza uma estrutura de variáveis que facilita a customização e expansão para múltiplos POPs.
  - **Segurança Integrada:** Inclui hardening básico do servidor com configuração de firewall (UFW), Fail2Ban e customização do acesso SSH.
  - **Integração via API:** Registra automaticamente o Proxy e o Agent 2 como um host no Zabbix Server, de forma idempotente.

## 🏛️ Arquitetura de Execução

A automação acontece inteiramente dentro do Host de Destino. Um operador acessa o servidor, clona este repositório e executa o playbook, que então configura a máquina localmente. A única comunicação externa é com a API do Zabbix Server.

```mermaid
graph LR;
    Operador --> ("1. Provisiona e acessa a VM com Debian 12") --> HostDestino;

    subgraph HostDestino["Máquina Virtual do PoP"]
        A("2. git clone & cd dev-zbx");
        B("3. ansible-playbook prov_zbxproxy.yml --limit ce -K");
        C{"4. Configuração Aplicada<br>Rede, Segurança, Zabbix"};
        D("5.Zabbix Agent & Zabbix Proxy");
        A --> B --> C --> D;
    end
    
    D --> ZabbixServer{"Zabbix Server Central"} 
    
```

## 📜 Entendendo as Roles

A lógica da automação é modularizada em roles, cada uma com uma responsabilidade clara:

| Role | Descrição |
| :--- | :--- |
| `setup_context` | **Ponto de Partida.** Identifica a qual grupo o host pertence no inventário (ex: `[ce]`) e carrega o arquivo de variáveis correspondente (ex: `ce.yml`). |
| `net_security` | **Camada de Base.** Realiza o hardening do servidor: configura hostname, rede, firewall (UFW), Fail2Ban e personaliza o acesso SSH. |
| `zabbix_proxy` | **Aplicação Principal.** Instala e configura o serviço Zabbix Proxy, gerenciando sua chave PSK de forma idempotente. |
| `zabbix_agent` | **Aplicação Auxiliar.** Instala e configura o serviço Zabbix Agent 2, que monitorará o próprio host do Proxy. |
| `zabbix_server_register_proxy`| **Integração.** Comunica-se com a API do Zabbix Server para criar ou atualizar o registro do Proxy. |
| `zabbix_server_register_agent`| **Integração.** Comunica-se com a API para criar ou atualizar o host correspondente ao Agent 2. |

-----

## ✅ Pré-requisitos

Para executar a automação, o **servidor de destino** deve ter:

  - Debian 12 (Bookworm) recém-instalado.
  - Acesso à internet para baixar pacotes.
  - Um usuário com permissões `sudo`.
  - **`git` e `ansible-core`** instalados:
    ```bash
    sudo apt update && sudo apt install -y git ansible-core
    ```
  - **Coleção Ansible `community.general`**:
    ```bash
    ansible-galaxy collection install community.general
    ```
    > **Por que isso é necessário?**\<br\>
    > Esta coleção fornece o filtro `community.general.dict_kv`, utilizado na role `zabbix_server_register_agent`. Ele é essencial para transformar a lista de IDs de grupos/templates retornada pela API do Zabbix no formato de dicionário exato que a API exige para criar/atualizar um host.

-----

## ⚙️ Workflow de Provisionamento

Para provisionar um novo Zabbix Proxy, siga os passos abaixo no servidor de destino:

#### Passo 1: Obter o Projeto

Clone o repositório e entre no diretório:

```bash
git clone https://git.rnp.br/gt-monitoramento/poc-monitoramento.git
cd dev-zbxproxy/
```

#### Passo 2: Configurar o Inventário e Variáveis

1.  **Inventário (`inventory/hosts`):** O arquivo já vem pronto para execução local. Não precisa de alterações.
2.  **Variáveis Globais (`group_vars/all.yml`):** Ajuste os parâmetros do seu Zabbix Server.
3.  **Variáveis Locais (`group_vars/pops_configs/`):** Crie ou edite o arquivo `.yml` correspondente à sigla do estado que você está provisionando (ex: `ce.yml`).

> **Importante:** O nome do grupo no inventário (ex: `[ce]`) **deve** ser idêntico ao nome do arquivo de variáveis (ex: `ce.yml`). É essa convenção que permite à role `setup_context` carregar as configurações corretas.

#### Passo 3: Executar o Playbook

Use o `--limit` para especificar qual configuração de POP será aplicada.

```bash
# Exemplo para provisionar o POP do Ceará (usará a config de ce.yml)
ansible-playbook prov_zbxproxy.yml --limit ce -K
```

-----

## 🤔 Solução de Problemas (Troubleshooting)

  - **Falha na tarefa "Carregar variáveis específicas da localidade"**:

      - Verifique se o nome do grupo no inventário (`[ce]`) corresponde exatamente ao nome do arquivo de variáveis (`group_vars/pops_configs/ce.yml`).

  - **As tarefas de API falham com erro de autenticação**:

      - Verifique se o `zabbix_api_token` em `group_vars/all.yml` está correto e se tem as permissões necessárias no Zabbix.
      - Confirme se a `zabbix_server_url` está acessível a partir do servidor onde o playbook está sendo executado.

  - **Playbook falha com erro "variável não definida"**:

      - Confirme se todas as variáveis necessárias estão definidas em `group_vars/all.yml` e no arquivo de POP específico (`group_vars/pops_configs/<sigla>.yml`).

-----

## ⚠️ Limitações Conhecidas

  - **Interface do Proxy via API**: A API do Zabbix não permite adicionar a interface (IP) do Proxy no momento da sua criação.
  - **Ação Manual Necessária (Apenas para o Proxy)**: Após a automação, acesse a interface web do Zabbix → `Administration > Proxies`, selecione o proxy criado e adicione manualmente o **Proxy address** (IP). *Esta limitação não se aplica ao host do agente, que é criado com a interface já configurada.*

-----

## 👨‍💻 Autores

  - **GT Monitoramento 2025**