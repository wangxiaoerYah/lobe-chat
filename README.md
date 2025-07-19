# Lobe-Chat

[![PFM-Upstream-Sync](https://github.com/PFM-PowerForMe/lobe-chat/actions/workflows/fork-sync.yml/badge.svg)](https://github.com/PFM-PowerForMe/lobe-chat/actions/workflows/fork-sync.yml)

## 简介
Lobe Chat - 一个开源的、现代设计的 AI 聊天框架。支持多种 AI 提供商（OpenAI / Claude 3 / Gemini / Ollama / DeepSeek / Qwen）、知识库（文件上传 / 知识管理 / RAG）、多模态（插件 / 工具）和 Thinking。一键免费部署您的私人 ChatGPT / Claude / DeepSeek 应用程序。

## 如何部署?

1. 前置条件:
```shell
mkdir -p /etc/containers/systemd
podman network create deploy
```

2. 部署 PGvector [PGvector](https://github.com/pgvector/pgvector)
```shell
mkdir -p /opt/podman-data/env
nvim /opt/podman-data/env/pgvector.env
```

```
POSTGRES_DB=lobe_db
POSTGRES_USER=lobe_db_user
POSTGRES_PASSWORD=your_passwd
```
---
```shell
nvim /etc/containers/systemd/pgvector.container
```

```
# /etc/containers/systemd/pgvector.container

[Unit]
Description=The pgvector container
Wants=network-online.target
After=network-online.target

[Container]
AutoUpdate=registry
ContainerName=pgvector
Timezone=local
Network=deploy
Volume=pgvector-data:/var/lib/postgresql/data
EnvironmentFile=/opt/podman-data/env/pgvector.env
Image=docker.io/pgvector/pgvector:pg17

[Service]
Restart=on-failure
RestartSec=30s
StartLimitInterval=30
TimeoutStartSec=900
TimeoutStopSec=70

[Install]
WantedBy=multi-user.target default.target
```

3. 部署 LobeChat
```shell
mkdir -p /opt/podman-data/env
nvim /opt/podman-data/env/lobe-chat.env
```

```
HTTP_PROXY=${HTTP_PROXY}
HTTPS_PROXY=${HTTP_PROXY}
DATABASE_URL=postgresql://${DB_USER}:${DB_PASSWORD}@pgvector:5432/${DB_NAME}
APP_URL=${APP_URL}
KEY_VAULTS_SECRET=${KEY_VAULTS_SECRET}
NEXT_AUTH_SECRET=${KEY_VAULTS_SECRET}
NEXT_AUTH_SSO_PROVIDERS=auth0
NEXTAUTH_URL=${APP_URL}/api/auth
AUTH_AUTH0_ID=${AUTH_AUTH0_ID}
AUTH_AUTH0_SECRET=${AUTH_AUTH0_SECRET}
AUTH_AUTH0_ISSUER=${AUTH_AUTH0_ISSUER}
S3_ENABLE_PATH_STYLE=1
S3_PUBLIC_DOMAIN=${S3_PUBLIC_DOMAIN}
S3_ENDPOINT=${S3_PUBLIC_DOMAIN}
S3_BUCKET=${S3_BUCKET}
S3_ACCESS_KEY_ID=${S3_ACCESS_KEY_ID}
S3_SECRET_ACCESS_KEY=${S3_SECRET_ACCESS_KEY}
```
---
```shell
nvim /etc/containers/systemd/lobe-chat.container
```

```
# /etc/containers/systemd/lobe-chat.container

[Unit]
Description=The lobe-chat container
Wants=pgvector.service
After=pgvector.service

[Container]
AutoUpdate=registry
ContainerName=lobe-chat
Timezone=local
Network=deploy
EnvironmentFile=/opt/podman-data/env/lobe-chat.env
PublishPort=127.0.0.1:6005:3210
Image=ghcr.io/powerforme/lobe-chat:latest

[Service]
Restart=on-failure
RestartSec=30s
StartLimitInterval=30
TimeoutStartSec=900
TimeoutStopSec=70

[Install]
WantedBy=multi-user.target default.target
```