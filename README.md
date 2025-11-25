# 💬 Monitoramento e Alerta de Inatividade de Chats (n8n/Tiflux)

---

Este repositório contém o **workflow de automação** desenvolvido em **n8n** (o arquivo JSON) que implementa um **Sistema de Monitoramento em Tempo Real** para o fluxo de atendimento da plataforma **Tiflux**.

O objetivo principal é garantir a **alta qualidade** e a **rapidez nas respostas**, notificando automaticamente a equipe sobre chats inativos e não atribuídos, visando a melhoria do **SLA** e **FRT**.

## 🚀 Funcionalidades Principais

O workflow opera em um ciclo contínuo, monitorando, classificando e escalando problemas de atendimento de forma robusta.

### 1. Detecção e Classificação de Inatividade 🔍

O sistema executa uma verificação **a cada 5 minutos** durante o horário de trabalho e classifica o estado de cada chat (via nó `Switch`), direcionando-o para a ação de alerta correta:

* **Estado Sem Atribuição:** Clientes novos aguardando o primeiro contato de um técnico.
* **Estado Aguardando Técnico:** Cliente respondeu, e o técnico está demorando a dar a próxima resposta.
* **Estado Aguardando Cliente:** Técnico respondeu, aguardando o retorno do cliente (monitoramento para *follow-up* ou encerramento).

### 2. Escalonamento Automático e Inteligente 📧

O sistema utiliza a base de dados para implementar um sistema de alerta com dois níveis, evitando *spam* e garantindo que apenas os **chats críticos** sejam escalados:

* **Alerta 1 (Aviso):** Disparado após **25 minutos** de inatividade.
* **Alerta 2 (Crítico):** Disparado após **35 minutos** de inatividade (escala para gestão).
    > 💡 **Mecanismo Anti-Spam:** Uma tabela em **Supabase (PostgreSQL)** registra o status de alerta (`level 1` ou `level 2`) de cada chat. O sistema consulta esta tabela antes de enviar um novo e-mail, impedindo a duplicação de alertas.

### 3. Arquitetura de Integração e Log ✨

O projeto é *low-code* e altamente conectado, garantindo a persistência e a lógica dos dados:

* **Integração API:** Conexão direta com a **API Tiflux** para coleta de dados em tempo real.
* **Base de Logs:** Utilização do **Supabase/PostgreSQL** para registrar o status, o nível de alerta e os *logs* de monitoramento.
* **Limpeza Diária:** Um `Schedule Trigger` separado executa a limpeza da base de logs todos os dias às 5h da manhã.

---

## 🛠️ Como Utilizar

### Requisitos
* Instância ativa do **n8n**.
* Credenciais de API configuradas para:
    * **Tiflux API Key** (Leitura de chats).
    * **Supabase/PostgreSQL** (Tabela `chat_monitoring`).
    * **SMTP** (Envio de E-mail).

### Execução
1.  Importe o arquivo JSON do workflow (`Tiflux/Verificação_Inatividade_Chat.json`) para sua instância do n8n.
2.  Preencha as credenciais nos nós correspondentes.
3.  Ative o workflow.

### Fluxo de Uso
1.  O `Schedule Trigger` inicia a execução (configurado a **cada 5 minutos, das 6h às 14h**).
2.  O nó `HTTP Request` coleta todos os chats em atendimento do Tiflux.
3.  O nó `Estado Chat` (Switch) direciona cada chat para o *loop* de monitoramento apropriado (Técnico, Cliente ou Sem Atribuição).
4.  O fluxo verifica o tempo de inatividade e o status de alerta no Supabase, disparando a notificação por e-mail, se necessário.

---

## 🎯 Objetivo

Esta automação é essencial para **gestores de atendimento** e **times de suporte técnico** que precisam de um utilitário robusto para monitorar e escalar rapidamente problemas de inatividade, garantindo que nenhum cliente seja negligenciado e que o SLA seja cumprido.

> **⚠️ Importante:** Este workflow requer credenciais ativas e um ambiente de banco de dados (Supabase/PostgreSQL) configurado para persistência de dados.
