# Provisionamento Automatizado de Zabbix Proxy com Ansible

![Ansible Version](https://img.shields.io/badge/ansible--core-2.15%2B-blue.svg)  
![License](https://img.shields.io/badge/license-MIT-green.svg)

## 📌 Descrição

O projeto automatiza, via **Ansible**, a implantação e configuração completa de um **Zabbix Proxy** em servidores **Debian 12 (Bookworm)**.  
A automação é executada **localmente no host de destino** e inclui:

- Configuração de rede e hardening de segurança  
- Instalação e configuração do Zabbix Proxy e Zabbix Agent 2  
- Registro do proxy no servidor Zabbix (com limitações)  
- Registro do Agent2 no servidor Zabbix (Em desenvolvimento)  


---

## 📂 Estrutura do Projeto

| Arquivo / Diretório                   | Descrição                                                                  |
| ------------------------------------- | ---------------------------------------------------------------------------|
| `prov_zbxproxy.yml`                   | Playbook principal que orquestra todas as roles.                           |
| `hosts`                               | Inventário Ansible com grupos de hosts (ex.: `[ce]`).                      |
| `group_vars/all.yml`                  | Variáveis globais (IP do Zabbix Server, URL, token da API, etc.).          |
| `group_vars/pops_configs/`            | Variáveis específicas por localidade (POP).                                |
| `roles/`                              | Diretório contendo todas as roles da automação.                            |
| `roles/setup_context/`                | Carrega variáveis corretas do POP com base no inventário.                  |
| `roles/net_security/`                 | Configuração de rede e segurança (UFW, Fail2Ban, SSH).                     |
| `roles/zabbix_proxy/`                 | Instala e configura o serviço Zabbix Proxy.                                |
| `roles/zabbix_agent/`                 | Instala e configura o Zabbix Agent 2.                                      |
| `roles/zabbix_server_register_proxy/` | Integração com a API do Zabbix para registrar o Proxy e o Agent2 no Server.|

---

## ✅ Pré-requisitos

No **servidor de destino** devem estar disponíveis:

   - Debian 12 (Bookworm)  
   - Usuário com permissões `sudo`  
   - `git`  
   - `ansible` 
   - SGBD: SQlite 3
   - Versão do Zabbix Proxy: `zabbix-proxy-sqlite3=1:7.2.7-1+debian12`
   - Versão do Zabbix Agent: `zabbix-agent2=1:7.2.7-1+debian12`

---

## ⚙️ Configuração

1. **Campo relacionado ao Zabbix Server (Central)**

   Variáveis globais (`group_vars/all.yml`)
   
   Seção 0: Variáveis Globais

   - `zabbix_server_ip`
   - `zabbix_server_url`  
   - `zabbix_api_token`


2. **Campo relacionado ao Zabbix Proxy (Local)**

   Variáveis por POP (`group_vars/pops_configs/`)  

   - Crie/edite o arquivo YAML correspondente (ex.: `ce.yml`);  
   - Defina os parâmetros de rede (`pop_network_interface`, `pop_network_ipv4_address`, `pop_network_ipv4_netmask`, etc.);  
   - Configure `zabbix_proxy_hostname`.  


3. **Recomendações**

   - Realizar um snapshot antes de executar o ansible-playbook para garantir um rollback caso algo tenha sido configurado incorretamente.
   - Utilizar duas interfaces de rede para não perder a conexão durante a execução. É importante se atentar que, após o script ser executado, a conexão ssh será feita apenas através da porta ssh configurada em `ssh_port` dentro das variáveis de rede especificas do pop. 
   - Além disso, as regras de UFW qu estão pré configuradas a rodar irão, por padrão, bloquear qualquer conexão de ip que não for listado nesse parâmetro ou relacionado ao Zabbix Server Central por padão, então é recomendado que o gateway configurado seja o Firewall da rede. 

4. **Execução**


   ```bash
   git clone https://git.rnp.br/gt-monitoramento/poc-monitoramento.git
   cd dev-zbxproxy/
   ```

   Substitua `sigla_do_estado` pela sigla correspondete ao estado:

   ```bash
   ansible-playbook -i hosts prov_zbxproxy.yml --limit sigla_do_estado -K
   ```

   ### 🔍 Detalhe do comando

   | Parâmetro           | Descrição                                                       |
   | ------------------- | --------------------------------------------------------------- |
   | `ansible-playbook`  | Executa o playbook especificado.                                |
   | `-i hosts`          | Define o inventário a ser utilizado.                            |
   | `prov_zbxproxy.yml` | Playbook principal da automação.                                |
   | `--limit ce`        | Restringe a execução apenas aos hosts do grupo Ex:`[ce]`.       |
   | `-K`                | Solicita a senha do `sudo` (equivalente a `--ask-become-pass`). |
   | `-v`, `-vv`, `-vvv` | Ajusta o nível de verbosidade da saída (útil para depuração).   |

   Ex:
   ```bash
   ansible-playbook -i hosts prov_zbxproxy.yml --limit ce -K -vvv
   ```

---

## ⚠️ Limitações

* **Registro da interface do proxy**: devido a uma limitação na API do Zabbix Server, o proxy é criado mas sua interface de rede (IP) não é adicionada.
* **Ação manual necessária**: após a execução, acesse o Zabbix Web → `Administration > Proxies`, selecione o proxy criado e adicione manualmente o **Proxy address** (IP).

---

## 👨‍💻 Autores

* **GT Monitoramento 2025**


