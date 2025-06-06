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

## 🚀 Instalação e Configuração

### Pré-requisitos

* **Docker** e **Docker Compose** instalados.
* Uma **API Key do Google AI Studio** para usar o Gemini. Você pode obter uma [aqui](https://makersuite.google.com/app/apikey).
* Um número de WhatsApp para ser usado pelo bot.

### 1. Crie o arquivo `docker-compose.yml`

Crie o arquivo `docker-compose.yml` na raiz do projeto com o seguinte conteúdo:

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
      - "3000:3000"

  n8n:
    image: n8nio/n8n:latest
    platform: linux/amd64
    environment:
      WEBHOOK_URL: [http://host.docker.internal:5678](http://host.docker.internal:5678)
      N8N_HOST: host.docker.internal
      GENERIC_TIMEZONE: America/Sao_Paulo
      N8N_LOG_LEVEL: debug
      N8N_COMMUNITY_PACKAGES_ALLOW_TOOL_USAGE: true
    volumes:
      - n8n_data:/home/node/.n8n
    ports:
      - "5678:5678"

volumes:
  pgdata:
  waha_sessions:
  waha_media:
  n8n_data:
```

### 2. Execute o Projeto

Com o arquivo `docker-compose.yml` criado, execute o seguinte comando no seu terminal:

```bash
# Inicia todos os contêineres em segundo plano
docker-compose up -d
```

### 3. Próximos Passos

1.  **Acesse o WAHA:** Navegue até `http://localhost:3000` para conectar seu número de WhatsApp escaneando o QR Code através do endpoint `/api/sessions/start`.
2.  **Acesse o N8N:** Navegue até `http://localhost:5678` para criar sua conta de administrador e começar a construir seu fluxo de trabalho.
3.  **Construa o Workflow:**
    * Crie um nó **Webhook** para receber os dados do WAHA (o caminho da URL deve ser `/webhook/webhook`).
    * Adicione um nó **Google Gemini**, insira suas credenciais e configure o prompt e a entrada de texto.
    * Adicione um nó **HTTP Request** para enviar a resposta do Gemini de volta para o usuário através da API do WAHA.

---

## 📄 Licença

Este projeto está sob a licença MIT. Sinta-se à vontade para usar e modificar o código.
