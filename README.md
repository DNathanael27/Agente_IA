# 🤖 Agente de IA para WhatsApp com N8N, WAHA e Gemini

Este repositório contém a configuração para implantar um agente de IA para WhatsApp. O projeto utiliza **N8N** para orquestrar o fluxo de trabalho, a API **WAHA (WhatsApp HTTP API)** para comunicação com o WhatsApp e a **IA do Google (Gemini)** para processamento de linguagem natural. O ambiente é totalmente gerenciado com **Docker**.

## 📝 Sobre o Projeto

O objetivo deste projeto é criar um chatbot inteligente no WhatsApp. Quando um usuário envia uma mensagem, ela é recebida pela API WAHA e encaminhada para um workflow no N8N. O N8N, usando sua base de dados interna (SQLite), processa a mensagem, a envia para a API do Google Gemini e retorna a resposta ao usuário via WAHA.

---

## 🛠️ Tecnologias Utilizadas

* **[Docker](https://www.docker.com/) & [Docker Compose](https://docs.docker.com/compose/):** Para criar e gerenciar os contêineres dos serviços.
* **[N8N](https://n8n.io/):** Ferramenta de automação de fluxo de trabalho que orquestra toda a lógica.
* **[WAHA (WhatsApp HTTP API)](https://waha.devlike.pro/):** API não-oficial para enviar e receber mensagens do WhatsApp.
* **[Google Gemini API](https://ai.google.dev/):** Modelo de IA para entender e gerar as respostas.
* **[PostgreSQL](https://www.postgresql.org/):** Disponível no ambiente, mas não conectado ao N8N nesta configuração.
* **[Redis](https://redis.io/):** Disponível no ambiente, mas não conectado ao N8N nesta configuração.

---

## ⚙️ Como Funciona

O fluxo de dados da aplicação segue os seguintes passos:

1.  O **Usuário** envia uma mensagem para o número de WhatsApp configurado.
2.  O contêiner **WAHA** recebe a mensagem e a envia para o webhook do N8N através do `host.docker.internal`.
3.  O **N8N** recebe a notificação via webhook e inicia o workflow.
4.  O nó do **Google Gemini** no N8N é acionado, enviando a mensagem do usuário para a IA.
5.  A **IA do Google** processa a entrada e gera uma resposta.
6.  O N8N envia a resposta gerada de volta para a API **WAHA**.
7.  A WAHA encaminha a resposta para o **Usuário** no WhatsApp.

```
Usuário ↔️ WhatsApp ↔️ WAHA ↔️ N8N (com SQLite) ↔️ Google Gemini AI
```

---

## 🚀 Instalação e Configuração

Siga os passos abaixo para executar o projeto.

### Pré-requisitos

* **Docker** e **Docker Compose** instalados (versões recentes são recomendadas).
* Uma **API Key do Google AI Studio** para usar o Gemini. Você pode obter uma [aqui](https://makersuite.google.com/app/apikey).
* Um número de WhatsApp para ser usado pelo bot.

### 1. Crie o arquivo `docker-compose.yml`

Crie o arquivo `docker-compose.yml` na raiz do projeto com o seguinte conteúdo que você forneceu:

```yaml
version: '3.8'

services:
  redis:
    image: redis:latest
    platform: linux/amd64
    command: redis-server --requirepass default
    environment:
      REDIS_USER: default
      REDIS_PASSWORD: default
    ports:
      - "6379:6379"

  postgres:
    image: postgres:latest
    platform: linux/amd64
    environment:
      POSTGRES_USER: default
      POSTGRES_PASSWORD: default
      POSTGRES_DB: default
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

  waha:
    image: devlikeapro/waha:latest
    platform: linux/amd64
    environment:
      WHATSAPP_HOOK_URL: [http://host.docker.internal:5678/webhook/webhook](http://host.docker.internal:5678/webhook/webhook)
      WHATSAPP_DEFAULT_ENGINE: GOWS
      WHATSAPP_HOOK_EVENTS: message
    volumes:
      - waha_sessions:/app/.sessions
      - waha_media:/app/.media
    ports:
      - "3000:30
