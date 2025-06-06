# 🤖 Agente de IA para WhatsApp com N8N, WAHA e Gemini

Este repositório contém a configuração para implantar um agente de IA para WhatsApp. O projeto utiliza **N8N** para orquestrar o fluxo de trabalho, a API **WAHA (WhatsApp HTTP API)** para comunicação com o WhatsApp, a **IA do Google (Gemini)** para processamento de linguagem natural e geração de respostas, e **PostgreSQL** para armazenar o histórico de interações. Tudo é executado em contêineres **Docker**.

## 📝 Sobre o Projeto

O objetivo deste projeto é criar um chatbot inteligente no WhatsApp. Quando um usuário envia uma mensagem, ela é recebida pela API WAHA e encaminhada para um workflow no N8N. O N8N processa a mensagem, a envia para a API do Google Gemini com um conjunto de instruções predefinidas, recebe a resposta gerada pela IA, armazena os dados da conversa em um banco de dados PostgreSQL e, finalmente, envia a resposta de volta ao usuário via WhatsApp.

---

## 🛠️ Tecnologias Utilizadas

* **[Docker](https://www.docker.com/) & [Docker Compose](https://docs.docker.com/compose/):** Para criar e gerenciar os contêineres dos serviços.
* **[N8N](https://n8n.io/):** Ferramenta de automação de fluxo de trabalho que orquestra toda a lógica.
* **[WAHA (WhatsApp HTTP API)](https://waha.devlike.pro/):** API não-oficial para enviar e receber mensagens do WhatsApp.
* **[Google Gemini API](https://ai.google.dev/):** Modelo de IA para entender e gerar as respostas.
* **[PostgreSQL](https://www.postgresql.org/):** Banco de dados relacional para salvar as interações.
* **[Redis](https://redis.io/):** Usado pelo N8N para gerenciamento de cache e filas.

---

## ⚙️ Como Funciona

O fluxo de dados da aplicação segue os seguintes passos:

1.  O **Usuário** envia uma mensagem para o número de WhatsApp configurado.
2.  O contêiner **WAHA** recebe a mensagem e a envia para o webhook do N8N.
3.  O **N8N** recebe a notificação via webhook e inicia o workflow.
4.  O nó do **Google Gemini** no N8N é acionado, enviando a mensagem do usuário juntamente com as instruções do sistema (prompt) para a IA.
5.  A **IA do Google** processa a entrada e gera uma resposta.
6.  O N8N salva a mensagem original e a resposta da IA no banco de dados **PostgreSQL**.
7.  O N8N envia a resposta gerada de volta para a API **WAHA**.
8.  A WAHA encaminha a resposta para o **Usuário** no WhatsApp.

```
Usuário ↔️ WhatsApp ↔️ WAHA ↔️ N8N ↔️ Google Gemini AI
                           ↕️
                       PostgreSQL
```

---

## 🚀 Instalação e Configuração

Siga os passos abaixo para executar o projeto.

### Pré-requisitos

* **Docker** e **Docker Compose** instalados.
* Uma **API Key do Google AI Studio** para usar o Gemini. Você pode obter uma [aqui](https://makersuite.google.com/app/apikey).
* Um número de WhatsApp para ser usado pelo bot.

### 1. Crie o arquivo `.env`

Na raiz do seu projeto, crie um arquivo chamado `.env` para guardar suas senhas e chaves. **Substitua os valores de exemplo pelos seus.**

```bash
# Credenciais para o banco de dados PostgreSQL
POSTGRES_USER=n8nuser
POSTGRES_PASSWORD=sua_senha_super_secreta_aqui
POSTGRES_DB=n8n_database

# Chave de criptografia para o N8N (use um valor longo e aleatório)
N8N_ENCRYPTION_KEY=sua_chave_de_criptografia_super_secreta_aqui

# Fuso horário para os serviços
TZ=America/Sao_Paulo
```

### 2. Crie o arquivo `docker-compose.yml`

Crie o arquivo `docker-compose.yml` na raiz do projeto com o seguinte conteúdo:

```yaml
version: '3.8'

services:
  postgres:
    image: postgres:15-alpine
    container_name: postgres_db
    restart: always
    environment:
      - POSTGRES_USER=<span class="math-inline">\{POSTGRES\_USER\}
\- POSTGRES\_PASSWORD\=</span>{POSTGRES_PASSWORD}
      - POSTGRES_DB=${POSTGRES_DB}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER} -d <span class="math-inline">\{POSTGRES\_DB\}"\]
interval\: 5s
timeout\: 5s
retries\:</18\> 10
redis\:
image\: redis\:latest
container\_name\: redis\_cache
restart\: always
volumes\:
\- redis\_data\:/data
n8n\:
image\: n8nio/n8n\:latest
container\_name\: n8n\_workflow
<19\>restart\: always
ports\:
\- "5678\:5678"
environment\:
\- DB\_TYPE\=postgresdb
\- <20\>DB\_POSTGRESDB\_HOST\=postgres
\- DB\_POSTGRESDB\_PORT\=5432
\- DB\_POSTGRESDB\_DATABASE\=</span>{POSTGRES_DB}
      - DB_POSTGRESDB_USER=<span class="math-inline">\{POSTGRES\_USER\}
\- DB\_POSTGRESDB\_PASSWORD\=</span>{POSTGRES_PASSWORD}
      - N8N_ENCRYPTION_KEY=<span class="math-inline">\{N8N\_ENCRYPTION\_KEY\}
\- GENERIC\_TIMEZONE\=</span>{TZ}
      - N8N_EMAIL_MODE=smtp
      - QUEUE_BULL_REDIS_HOST=redis
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    volumes:
      - n8n_data:/home/node/.n8n

  waha:
    image: devlikeapro/waha:latest
    container_name: waha_api
    restart: always
    ports:
      - "3000:3000"
    environment:
      # O webhook que o WAHA irá chamar no N8N.
      # O URL usa o nome do serviço 'n8n' e a porta interna '5678'.
      # '/webhook/minha-ia-whatsapp' é a URL do seu webhook no N8N. Mude se for diferente.
      - WAHA_WEBHOOK_URL=http://n8n:5678/webhook/minha-ia-whatsapp
    volumes:
      - waha_sessions:/app/sessions

volumes:
  postgres_data:
  redis_data:
  n8n_data:
  waha_sessions:
```

### 3. Execute o Projeto

Com os arquivos `docker-compose.yml` e `.env` na mesma pasta, execute o seguinte comando no seu terminal:

```bash
# Inicia todos os contêineres em segundo plano
docker-compose up -d
```

Após a execução, você pode:
* Acessar o **N8N** em `http://localhost:5678` para configurar seu workflow.
* Acessar a UI do **WAHA** em `http://localhost:3000` para escanear o QR Code e conectar seu WhatsApp.

### 4. Configure o Workflow no N8N

1.  Acesse o N8N, crie sua conta de administrador e inicie um novo workflow.
2.  Crie um nó de **Webhook**. Ele irá gerar uma URL de teste e uma de produção. Use a URL de produção na variável `WAHA_WEBHOOK_URL` do `docker-compose.yml` (lembre-se de que o webhook deve usar o nome do serviço `n8n`, como no exemplo).
3.  Adicione um nó do **Google Gemini** e configure-o com sua API Key em *Credentials*.
4.  Adicione um nó de **PostgreSQL** para salvar os dados no banco. Use as credenciais do seu arquivo `.env` e o nome do host `postgres`.
5.  Adicione um nó **HTTP Request** para enviar a resposta de volta para a API do WAHA.

---

## 📄 Licença

Este projeto está sob a licença MIT. Sinta-se à vontade para usar e modificar o código.
