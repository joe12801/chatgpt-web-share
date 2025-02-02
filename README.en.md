# ChatGPT Web Share

[![GitHub tag (latest by date)](https://img.shields.io/github/v/tag/moeakwak/chatgpt-web-share?label=container&logo=docker)](https://github.com/moeakwak/chatgpt-web-share/pkgs/container/chatgpt-web-share)
[![Github Workflow Status](https://img.shields.io/github/actions/workflow/status/moeakwak/chatgpt-web-share/docker-image.yml?label=build)](https://github.com/moeakwak/chatgpt-web-share/actions)
[![License](https://img.shields.io/github/license/moeakwak/chatgpt-web-share)](https://github.com/moeakwak/chatgpt-web-share/blob/main/LICENSE)

A web application that allows multiple users to share a ChatGPT account at the same time, developed using FastAPI and Vue3. It can be used for sharing or renting a ChatGPT account among friends. It supports ChatGPT Plus, setting conversation models, and user request limits.

**3.15 Update: Now supports GPT-4!** You can share a ChatGPT Plus account with your friends and use GPT-4 together.

![screenshot](screenshot.en.jpeg)

This readme was translated by ChatGPT.

## Features

- Uses the unofficial ChatGPT API, supports ChatGPT Plus accounts
- **Supports GPT-4** 🥳
- Supports selecting which ChatGPT model to use (sha or paid or gpt-4, if is plus account)
- A beautiful and concise web interface using [naive-ui](https://www.naiveui.com/)
  - multiple languages
  - dark mode
  - copying reply content as Markdown format with one click
  - showing images/tables/formulas/syntax highlighting in replies
  - Export conversation to beautiful markdown and PDF files 🤩 (new in v0.2.3)
- Creates multiple users to share a ChatGPT account
- Different users' ChatGPT conversations are separated and do not affect each other
- When multiple users request at the same time, they will be queued for processing
- Administrators can set users' maximum number of conversations and conversation time limits, etc.

## Using Proxy

Risk Warning: This project is currently using [revChatGPT](https://github.com/acheong08/ChatGPT) V1, which uses its reverse proxy to bypass Cloudflare verification, therefore it is subject to request limits and does not guarantee long-term stability. And it has been recently reported that OpenAI may deactivate accounts that use this method. Please use it at your own risk.

However, if you have a ChatGPT Plus account, you can use a [custom proxy](https://github.com/acheong08/ChatGPT-Proxy-V4) to bypass the request limit, which was already integrated into the docker container. See below for details.

## Deployment

### Using docker

It is recommended to use docker-compose for deployment. Create a new `docker-compose.yml` file with the following contents:

```yaml
version: "3"

services:
  chatgpt-share:
    image: ghcr.io/moeakwak/chatgpt-web-share:latest
    container_name: chatgpt-web-share
    restart: always
    network_mode: bridge
    ports:
      - 8080:80 # web port
    volumes:
      - ./data:/data # store database files
      - ./config.yaml:/app/backend/api/config/config.yaml # backend config file
      - ./logs:/app/logs # log files
```

In the same folder, create config.yaml with the following contents:

Create a `config.yaml` file in the same directory with the following content:

```yaml
print_sql: false
host: "127.0.0.1"
port: 8000
database_url: "sqlite+aiosqlite:////data/database.db"
run_migration: false

jwt_secret: "your jwt secret" # Used for generating JWT token, like a password
jwt_lifetime_seconds: 86400
cookie_max_age: 86400 # Login expiration time
user_secret: "your user secret" # Used for generating user password, like a password

sync_conversations_on_startup: true # Whether to synchronize ChatGPT conversations on startup, recommended to enable
create_initial_admin_user: true # Whether to create initial admin user
create_initial_user: false # Whether to create initial normal user
initial_admin_username: admin # Initial admin username
initial_admin_password: password # Initial admin password
initial_user_username: user # Initial normal username
initial_user_password: password # Initial normal password
ask_timeout: 600 # Timeout for ChatGPT requests, in seconds

chatgpt_access_token: "your access_token" # Need to get from ChatGPT
chatgpt_paid: true # Whether you are a ChatGPT Plus user

log_dir: /app/logs  # Log file directory
console_log_level: DEBUG
```

How to get `chatgpt_access_token`: After logging in to `chat.openai.com`, open https://chat.openai.com/api/auth/session and get the `accessToken` field.

It's highly recommended to use a plus account, so that you can use your own proxy. If you have a plus account, add the following to `config.yaml`:

```yaml
chatgpt_base_url: http://127.0.0.1:6062/api/
run_reverse_proxy: true
reverse_proxy_port: 6062
reverse_proxy_binary_path: /app/backend/ChatGPT-Proxy-V4
reverse_proxy_puid: "_puid value from cookie"
```

Note that `reverse_proxy_puid` needs to be obtained from your browser: Open https://chat.openai.com/, open the developer tools, find the `_puid` field in the cookies.

`reverse_proxy_binary_path` is the path to the executable file of the reverse proxy service. If using Docker, it is included in the image at the path `/app/backend/ChatGPT-Proxy-V4`.

`chatgpt_base_url` can also be set to the address of another reverse proxy service. If `run_reverse_proxy` is enabled, make sure the port of `chatgpt_base_url` matches `reverse_proxy_port`.

Finally, run `docker-compose up -d`.

#### Upgrading

To upgrade, run `docker-compose pull` and `docker-compose up -d`.

### Using Caddy

#### Frontend

You need to install nodejs and pnpm first, then run:

```bash
cd frontend
pnpm install
pnpm run build
```

#### Backend

You need to install poetry first and place config.yaml in the `backend/api/config` directory, then run:

```bash
cd backend
poetry install
poetry run python main.py
```

After installing Caddy, create a new Caddyfile and refer to the [Caddyfile](Caddyfile) for its content.

Use `caddy start` to start Caddy.

## Information Collection and Privacy Statement

Starting from version v0.2.16, this project utilizes Sentry to collect error information. By using this project, you agree to the Sentry privacy policy. Any anonymous information collected through Sentry will only be used for development and debugging purposes. We will never collect or store any of your private data, like username, password, access token, etc.

If you do not want to be tracked by Sentry, you can set the environment variable `VITE_DISABLE_SENTRY` to "yes" before build the frontend.
