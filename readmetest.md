
---

# 🤖 Provisionamento Automatizado de Zabbix Proxy com Ansible

Automação completa para instalar, configurar e registrar um **Zabbix Proxy** (e o **Zabbix Agent 2** do próprio host) em **Debian 12 (Bookworm)**. O playbook executa **localmente** no servidor de destino e aplica rede, hardening e integração via API com o Zabbix Server central. ([GitHub][1])

## 📌 Sumário

* [Visão Geral](#visão-geral)
* [Arquitetura (alto nível)](#arquitetura-alto-nível)
* [Estrutura do Repositório](#estrutura-do-repositório)
* [Pré-requisitos](#pré-requisitos)
* [Como começar (passo a passo)](#como-começar-passo-a-passo)
* [Configuração por POP (variáveis locais)](#configuração-por-pop-variáveis-locais)
* [Execução](#execução)
* [Solução de Problemas](#solução-de-problemas)
* [Limitações/Observações](#limitaçõesobservações)
* [FAQ Rápido](#faq-rápido)
* [Créditos](#créditos)

---

## Visão Geral

* Provisiona **Zabbix Proxy (TLS/PSK)** e **Zabbix Agent 2** do próprio host.
* Faz **hardening** (UFW, Fail2Ban, SSH), configura **hostname** e **rede** (Netplan).
* Integra **automaticamente** no Zabbix Server via **API** para registrar **Proxy** e **Host do Agent**.
* Execução **idempotente** e **local** (no próprio servidor de destino). ([GitHub][1])

---

## Arquitetura (alto nível)

* Operador acessa a VM/servidor do POP, clona este repositório e roda o playbook.
* Toda configuração é aplicada **no host local**; a única comunicação externa é com **API/Trappers** do Zabbix Server para registro/telemetria. ([GitHub][1])

---

## Estrutura do Repositório

```
Delta/
├─ prov_zbxproxy.yml          # Play principal
├─ hosts                      # Inventário Ansible (grupos por POP)
├─ group_vars/
│  ├─ pops_configs/           # Variáveis específicas por POP (ex.: ce.yml, rj.yml, ...)
│  └─ (variáveis globais, se aplicável)
├─ roles/
│  ├─ setup_context/          # Descobre contexto do grupo e carrega variáveis
│  ├─ net_security/           # Hostname, netplan, UFW, Fail2Ban, SSH (hardening)
│  ├─ zabbix_proxy/           # Instala e configura o Zabbix Proxy + PSK
│  ├─ zabbix_agent/           # Instala e configura o Zabbix Agent 2
│  ├─ zabbix_server_register_proxy/  # Chama API p/ criar/atualizar Proxy no Server
│  └─ zabbix_server_register_agent/  # Chama API p/ criar/atualizar Host (Agent2)
```

> Os nomes e a divisão das roles seguem o que o próprio projeto descreve: **setup\_context, net\_security, zabbix\_proxy, zabbix\_agent, zabbix\_server\_register\_proxy e zabbix\_server\_register\_agent**. ([GitHub][1])

---

## Pré-requisitos

No **servidor de destino** (Debian 12):

```bash
sudo apt update
sudo apt install -y git ansible-core
ansible-galaxy collection install community.general
```

Versões Zabbix já validadas no projeto (ajuste se necessário):

* `zabbix-proxy-sqlite3=1:7.2.7-1+debian12`
* `zabbix-agent2=1:7.2.7-1+debian12` ([GitHub][1])

> ⚠️ O projeto depende de **ambientes/variáveis específicas** para cada POP (e possivelmente de PSKs/Token da API). Não é necessário executar nada fora do host alvo, mas garanta que as credenciais/PSK/Token estejam preenchidas nas variáveis.

---

## Como começar (passo a passo)

1. **Clonar este repositório no host alvo**

```bash
git clone https://github.com/felipeanj0s/Delta.git
cd Delta
```

2. **Definir o inventário** em `hosts`

   * Crie/edite um grupo por POP (ex.: `[ce]`) e aponte para `localhost` se a execução for local.

3. **Criar o arquivo de variáveis do POP**

   * Em `group_vars/pops_configs/`, crie **`<sigla_do_estado>.yml`** (ex.: `ce.yml`) com as variáveis do seu ambiente (vide seção abaixo). ([GitHub][1])

---

## Configuração por POP (variáveis locais)

Crie/edite `group_vars/pops_configs/<sigla>.yml` com, no mínimo:

```yaml
# Identidade do Proxy (nome único no ambiente)
zabbix_proxy_hostname: "ce-zabbix-rnp-ger-proxy01"

# Rede (Netplan)
pop_network_ipv4_address: "192.168.0.17/24"
pop_network_ipv4_gateway:  "192.168.0.9"
pop_network_dns_list:
  - "200.19.16.53"
  - "200.137.53.53"

# SSH
ssh_port: 25085

# (Exemplos) Credenciais/segredos que sua role espera:
# zabbix_api_url: "http://SEU_ZABBIX_SERVER/zabbix/api_jsonrpc.php"
# zabbix_api_token: "xxxxx"
# zabbix_psk_identity: "proxy-ce"
# zabbix_psk_key: "CHAVE_HEX_64"
```

> Dica de operação:
>
> * Tire **snapshot** da VM antes da primeira execução (mudanças de rede/firewall podem cortar o acesso).
> * Após rodar, o SSH ficará disponível **na nova porta** definida em `ssh_port`.
> * O UFW sobe restritivo, então **libere previamente** o gateway/gestão para evitar bloqueio. ([GitHub][1])

---

## Execução

Rode **no host alvo**:

```bash
ansible-playbook -i hosts prov_zbxproxy.yml --limit <sigla_do_estado> -K
```

Parâmetros úteis:

* `--limit <grupo>`: executa só para o grupo (ex.: `ce`)
* `-K`: solicita senha de `sudo`
* `-v | -vv | -vvv | -vvvv`: níveis de verbosidade para depuração ([GitHub][1])

---

## Solução de Problemas

| Sintoma                      | Diagnóstico                         | Ação sugerida                                                                       |
| ---------------------------- | ----------------------------------- | ----------------------------------------------------------------------------------- |
| Proxy não inicia             | `systemctl status zabbix-proxy`     | Verifique `ServerPort`, diretórios e permissões                                     |
| Erro TLS no Agent            | `/var/log/zabbix/zabbix_agent2.log` | Confirme **PSK** (identity/key) e limpe cache no Server                             |
| Host do Agent “Desconhecido” | Checar configuração na UI           | O host do **Agent 2** do proxy deve ser monitorado pelo **Server** (não pelo proxy) |

> Após o provisionamento, valide na UI do Zabbix:
> **Administração → Proxies** (ativo, PSK, online, “última vez visto” recente) e **Monitoramento → Hosts** (ZBX verde para o host). ([GitHub][1])

---

## Limitações/Observações

* A API do Zabbix não associa IP/DNS ao Proxy na criação; ajuste **manualmente** na UI após o primeiro registro (**Administração → Proxies**). ([GitHub][1])
* Este projeto foi pensado para execução **local** no host alvo (sem “control node” central).
* Certifique-se de manter as **variáveis globais** alinhadas às diretrizes da sua GER/Backbone quando existirem.

---

## FAQ Rápido

**Posso rodar várias vezes?**
Sim. O play é **idempotente** e deve convergir para o estado desejado. ([GitHub][1])

**Preciso abrir portas no firewall?**
A role de segurança já libera **SSH** (na porta configurada) e portas do **Zabbix**. Ajustes extras podem ser feitos nas variáveis do POP.

**Onde coloco Token/PSK?**
No arquivo `group_vars/pops_configs/<sigla>.yml` (ou em **vault**). Evite commitar segredos em texto puro.

**E se eu quiser SQLite vs. MySQL?**
O exemplo utiliza `zabbix-proxy-sqlite3`. Se for usar outra variante, alinhe as variáveis/templates da role `zabbix_proxy`.

---

## Créditos

Baseado no trabalho do **GT Monitoramento 2025** e nas roles/estrutura deste repositório. ([GitHub][1])

---

### Notas da análise

* O README antigo do repo já refletia a proposta (execução local, hardening, integração via API) e listava as roles/fluxo; eu corrigi **URL de clone** e normalizei a **organização por POP** e exemplos de comando, removendo referências a outro repositório/paths que constavam no texto original. ([GitHub][1])

Se quiser, eu já abro um **PR** com este README no seu repositório — ou te mando em outro formato (PDF/Markdown “bonitão” com sumário clicável).

[1]: https://github.com/felipeanj0s/Delta "GitHub - felipeanj0s/Delta"
