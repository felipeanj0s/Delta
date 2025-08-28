# Projeto Delta: Provisionamento Automatizado de Zabbix Proxy com Ansible

![Ansible Version](https://img.shields.io/badge/ansible--core-2.15%2B-blue.svg)  
![License](https://img.shields.io/badge/license-MIT-green.svg)

## 📌 Descrição

O **Projeto Delta** automatiza, via **Ansible**, a implantação e configuração completa de um **Zabbix Proxy** em servidores **Debian 12 (Bookworm)**.  
A automação é executada **localmente no host de destino** e inclui:

- Configuração de rede e hardening de segurança  
- Instalação e configuração do Zabbix Proxy e Zabbix Agent 2  
- Registro do proxy no servidor Zabbix (com limitações)  

---

## 📂 Estrutura do Projeto

| Arquivo / Diretório                   | Descrição                                                                 |
| ------------------------------------- | ------------------------------------------------------------------------- |
| `prov_zbxproxy.yml`                   | Playbook principal que orquestra todas as roles.                           |
| `hosts`                               | Inventário Ansible com grupos de hosts (ex.: `[ce]`).                      |
| `group_vars/all.yml`                  | Variáveis globais (IP do Zabbix Server, URL, token da API, etc.).          |
| `group_vars/pops_configs/`            | Variáveis específicas por localidade (POP).                                |
| `roles/`                              | Diretório contendo todas as roles da automação.                            |
| `roles/setup_context/`                | Carrega variáveis corretas do POP com base no inventário.                  |
| `roles/net_security/`                 | Configuração de rede e segurança (UFW, Fail2Ban, SSH).                     |
| `roles/zabbix_proxy/`                 | Instala e configura o serviço Zabbix Proxy.                                |
| `roles/zabbix_agent/`                 | Instala e configura o Zabbix Agent 2.                                      |
| `roles/zabbix_server_register_proxy/` | Integração com a API do Zabbix Server para registrar o proxy.              |

---

## ✅ Pré-requisitos

No **servidor de destino** devem estar disponíveis:

- Debian 12 (Bookworm)  
- Usuário com permissões `sudo`  
- `git`  
- `python3-pip`  
- `ansible` (recomendado instalar via `pip`)  

---

## ⚙️ Configuração

1. **Variáveis globais (`group_vars/all.yml`)**  
   - Ajuste `zabbix_server_ip`, `zabbix_server_url` e `zabbix_api_token`.  

2. **Variáveis por POP (`group_vars/pops_configs/`)**  
   - Crie/edite o arquivo YAML correspondente (ex.: `ce.yml`);  
   - Defina os parâmetros de rede (`pop_network_interface`, `pop_network_ipv4_address`, `pop_network_ipv4_netmask`, etc.);  
   - Configure `zabbix_proxy_hostname`.  

---

## ▶️ Execução

**1. Clone o repositório**

```bash
git clone https://github.com/felipeanj0s/Delta.git
cd Delta/

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

  - **GT Monitoramento**

<!-- end list -->
