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
**2. Execute o playbook**

Substitua `ce` pelo grupo correspondente ao host no inventário:

```bash
ansible-playbook -i hosts prov_zbxproxy.yml --limit ce -K
```

### 🔍 Detalhe do comando

| Parâmetro           | Descrição                                                       |
| ------------------- | --------------------------------------------------------------- |
| `ansible-playbook`  | Executa o playbook especificado.                                |
| `-i hosts`          | Define o inventário a ser utilizado.                            |
| `prov_zbxproxy.yml` | Playbook principal da automação.                                |
| `--limit ce`        | Restringe a execução apenas aos hosts do grupo `[ce]`.          |
| `-K`                | Solicita a senha do `sudo` (equivalente a `--ask-become-pass`). |
| `-v`, `-vv`, `-vvv` | Ajusta o nível de verbosidade da saída (útil para depuração).   |

---

## ⚠️ Limitações

* **Registro da interface do proxy**: devido a uma limitação na API do Zabbix Server, o proxy é criado mas sua interface de rede (IP) não é adicionada.
* **Ação manual necessária**: após a execução, acesse o Zabbix Web → `Administration > Proxies`, selecione o proxy criado e adicione manualmente o **Proxy address** (IP).

---

## 👨‍💻 Autor

* **GT Monitoramento**

```

---

Quer que eu deixe essa versão mais **curta (guia rápido)** ou mantenho esse estilo **documentado e explicativo** pra time interno?
```
