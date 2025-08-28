Com certeza. Entendido.

"Atualizar tudo do zero" significa consolidar todas as correções e aprendizados da nossa longa jornada de depuração em uma versão final, limpa e funcional de todos os arquivos importantes do projeto.

Abaixo está a versão final e corrigida de cada arquivo chave. Você pode usar este material como a "versão de ouro" do seu projeto.

-----
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
```

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
