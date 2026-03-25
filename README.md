
------------------------------
# 📑 Guia de Implementação: Lab SIEM/EDR com Wazuh
Este projeto documenta a configuração de um ambiente de monitoramento centralizado, utilizando o Wazuh para coleta e análise de eventos de segurança em um endpoint Linux.
------------------------------
## 1. Implementação do Wazuh Manager (Servidor)
Para o núcleo do SIEM, utilizei o Wazuh OVA, um appliance oficial que integra o Indexer, o Server e o Dashboard em uma única estrutura otimizada.

* Plataforma: Oracle VirtualBox.
* Rede: Configurada em modo Bridge para permitir que o servidor busque atualizações de assinaturas de vulnerabilidades e regras de detecção na base da Wazuh.


<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/048bd1af-241c-41b6-a99a-a4414e6952c1" />

------------------------------
## 2. Preparação do Endpoint (Agente Ubuntu)
Configurei uma segunda máquina virtual com Ubuntu para simular um servidor de produção. O objetivo é que este host envie toda a sua telemetria (logs, integridade de arquivos e processos) para o Manager.

* Rede: Em modo NAT.

------------------------------
### 3. Instalação Assistida do Agente (Método do Dashboard)
Para garantir a precisão da configuração, utilizei o fluxo de "Deploy New Agent" do painel do Wazuh. Este método gera um comando unificado que automatiza o download e injeta o IP do Manager diretamente no instalador.
Execução no Terminal do Ubuntu:
O comando abaixo realiza o download do pacote .deb (v4.14.4) e utiliza a variável WAZUH_MANAGER para registrar o agente no IP 192.168.0.23 de forma automática:

### Download e Instalação com Registro Automático
```
wget https://packages.wazuh.com && \
sudo WAZUH_MANAGER='192.168.0.23' dpkg -i ./wazuh-agent_4.14.4-1_amd64.deb

```
### Inicialização e Persistência do Serviço

```
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent

```

<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/d0cf9215-45d8-463a-8e26-993f4e7cf456" />


------------------------------
## 4. Validação e Monitoramento Ativo
Após a instalação, a conectividade foi validada através do console web do SIEM. O agente foi reconhecido instantaneamente, iniciando o fluxo de dados.

* Verificação: O host Ubuntu passou para o status "Active".
* Inventário: O Wazuh iniciou a varredura de pacotes instalados e vulnerabilidades conhecidas no endpoint.

<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/a6bdd0fa-ae26-49c8-bb26-2d9cae94a765" />

------------------------------


