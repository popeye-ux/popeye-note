---
title: 'Docker 入門：容器化你的應用程式'
description: '從零開始學習 Docker，了解容器化技術的基本概念與實際應用'
pubDate: '2024-01-15'
updatedDate: '2024-02-01'
heroImage: '../../assets/blog-placeholder-1.jpg'
tags: ['Docker', 'DevOps', '容器化']
---

Docker 改變了應用程式的部署方式，「在我電腦上明明能跑！」的問題從此成為歷史。本文將帶你從零開始了解 Docker 的核心概念。

## 什麼是 Docker？

Docker 是一個開放原始碼的容器化平台。**容器**是一種輕量級的虛擬化技術，它將應用程式及其所有依賴打包在一個可攜式的單元中。

容器 vs 虛擬機器（VM）：

| 特性 | 容器 | 虛擬機器 |
|------|------|----------|
| 啟動時間 | 秒級 | 分鐘級 |
| 資源使用 | 極少 | 較多 |
| 隔離程度 | 程序級 | 完整作業系統 |
| 可攜性 | 極高 | 較低 |

## 核心概念

### Image（映像檔）

Image 是容器的藍圖，包含了應用程式運行所需的所有內容：

```dockerfile
# 這就是一個 Dockerfile，用來建立 Image
FROM node:20-alpine

WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .

EXPOSE 3000
CMD ["node", "server.js"]
```

### Container（容器）

Container 是 Image 的執行實例：

```bash
# 建立並啟動容器
docker run -d -p 3000:3000 --name my-app my-image

# 查看執行中的容器
docker ps

# 停止容器
docker stop my-app

# 刪除容器
docker rm my-app
```

## 實際範例：Node.js 應用程式

**1. 建立 Dockerfile**

```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine
WORKDIR /app
COPY --from=builder /app/node_modules ./node_modules
COPY . .

USER node
EXPOSE 3000
CMD ["node", "index.js"]
```

**2. 建立 .dockerignore**

```
node_modules
.git
.env
*.log
```

**3. 建立並推送 Image**

```bash
# 建立 Image
docker build -t my-app:latest .

# 標記版本
docker tag my-app:latest my-app:1.0.0

# 推送到 Docker Hub
docker push username/my-app:latest
```

## Docker Compose

管理多個容器的最佳工具：

```yaml
# docker-compose.yml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=postgresql://postgres:password@db:5432/mydb
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_DB: mydb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 5s
      timeout: 5s
      retries: 5

volumes:
  postgres_data:
```

```bash
# 啟動所有服務
docker compose up -d

# 查看日誌
docker compose logs -f web

# 停止所有服務
docker compose down
```

## 常用指令速查

```bash
# 映像檔管理
docker images              # 列出本地映像檔
docker pull nginx          # 下載映像檔
docker rmi my-image        # 刪除映像檔

# 容器管理
docker ps -a               # 列出所有容器（包含停止的）
docker logs my-container   # 查看容器日誌
docker exec -it my-container sh  # 進入容器

# 清理系統
docker system prune -a     # 清除所有未使用的資源
```

## 結語

Docker 的學習曲線雖然稍陡，但一旦掌握，你會發現它大幅簡化了開發、測試和部署的工作流程。

下一步建議學習 Kubernetes，它是管理大規模容器化應用程式的最佳工具。
