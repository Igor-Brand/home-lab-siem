
------------------------------
# 📑 Guia de Implementação: Lab SIEM/EDR com Wazuh + DFIR IRIS
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
 ## SIEM Lab - Fase 2: Resposta a Incidentes com DFIR IRIS
Esta etapa marca a evolução do laboratório, integrando a capacidade de detecção do Wazuh com a gestão de incidentes da plataforma DFIR IRIS. O objetivo principal é automatizar o fluxo de trabalho, transformando alertas críticos do SIEM em casos investigativos dentro do IRIS de forma imediata.
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/3631a3ed-ff79-40a6-a6fe-f9c571fd12de" />
Evidência da Integração (PoC): Demonstração da correlação em tempo real entre o evento detectado pelo Wazuh Manager (fundo) e o incidente gerado automaticamente no DFIR IRIS (frente). Note que o alerta Apparmor DENIED (Rule ID: 52002) foi propagado via Webhook, comprovando a comunicação bem-sucedida entre os diferentes nós da infraestrutura (IP .23 para IP .26).

 ## Gerenciamento Remoto via PowerShell
Devido à ausência de integração de mouse no console da máquina virtual do Wazuh, utilizei o PowerShell no host Windows como ferramenta central de configuração. Essa abordagem permitiu:
Configuração de API: Provisionamento ágil das chaves de API e webhooks no ossec.conf.
Gestão de Arquivos: Edição remota de scripts de integração sem dependência da interface gráfica da VM.
Automação de Serviços: Reinício e validação dos serviços do Manager via terminal, garantindo a persistência das configurações.
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/4203ff9c-97b8-4553-99a0-6e09d4d9f4c0" />
Gerenciamento Remoto via PowerShell (SSH): Evidência da sessão SSH estabelecida a partir do host Windows para o servidor Wazuh Manager. A imagem demonstra a superação da limitação de interface da VM, confirmando a identidade do host, o endereço IP (.23) e o status operacional (Running) de todos os serviços do SIEM necessários para a integração com o DFIR IRIS.

---
### 🔍 1. Simulação de Ataque: Brute Force SSH
O primeiro teste de estresse do ambiente foi um ataque de força bruta contra o serviço SSH do servidor Wazuh. Utilizei o Kali Linux com a ferramenta Hydra para tentar adivinhar a senha do usuário wazuh-user.
Detalhes do Teste:
Ferramenta: THC-Hydra v9.6.
Alvo: 192.168.0.23 (Wazuh Manager).
Dicionário: lista_de_senha.txt.
Objetivo: Gerar múltiplas falhas de autenticação em um curto período para disparar os gatilhos do SIEM.
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/86b552aa-a2ab-4e89-8c68-f9ee979de250" />

#### 🚨 2. Detecção e Análise no Wazuh
O motor de análise do Wazuh identificou imediatamente o padrão anômalo de tentativas de login. O log de eventos registrou múltiplas falhas seguidas (Rule ID: 5710 e 5712), elevando o nível do alerta devido à persistência do ataque.
Identificação do Atacante: O SIEM capturou o IP de origem do Kali Linux.
Classificação: O evento foi classificado como uma tentativa de invasão (Authentication Failure).
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/0d938321-44ac-4b87-aa9a-a38e79a210b3" />
Análise de Eventos no Dashboard (Wazuh Discover): Visualização detalhada do log de segurança após o ataque simulado com Hydra. A imagem evidencia a captura do IP de origem do atacante (.24), o usuário alvo (wazuh-user) e a mensagem do sistema (PAM authentication failures), demonstrando a capacidade do SIEM em decodificar e indexar tentativas de invasão em tempo real.

##### 📑 3. Escalonamento Automático para o DFIR IRIS
Graças à integração configurada na Fase 2, o alerta crítico de Brute Force não ficou apenas no log. O Wazuh Integratord disparou um webhook para o DFIR IRIS, criando automaticamente um caso de investigação.
Dados integrados no incidente:
Descrição completa do ataque.
Timestamp preciso do evento.
IP de origem e destino, permitindo que o analista de SOC inicie a contenção imediatamente.
<img width="1366" height="768" alt="image" src="https://github.com/user-attachments/assets/cf90704a-9a2d-43db-8d72-0a1ca8f574d2" />
Gestão de Incidentes no DFIR IRIS: Visualização do alerta de Brute Force convertido automaticamente em um caso de investigação. A plataforma exibe o mapeamento para o framework MITRE ATT&CK, a severidade do evento (Level 10) e os logs brutos enviados pelo Wazuh, permitindo que a equipe de resposta (CSIRT) tenha todo o contexto necessário para a contenção do ataque.
