# Docker 学习笔记

## 目录
1. [Docker 基础概念](#docker-基础概念)
2. [Docker 基本命令](#docker-基本命令)
3. [Dockerfile 使用](#dockerfile-使用)
4. [Docker Compose](#docker-compose)
5. [镜像相关](#镜像相关)
6. [容器相关](#容器相关)
7. [日志查看](#日志查看)
8. [镜像包结构](#镜像包结构)
9. [容器内文件结构](#容器内文件结构)

---

## Docker 基础概念

### 什么是 Docker
Docker是一个开源的容器化平台，用于开发、部署和运行应用程序。它使用容器技术来打包应用程序及其依赖项，实现快速、一致的部署。

### Docker 核心概念
| 概念 | 说明 |
|------|------|
| **Image (镜像)** | 应用程序的只读模板 |
| **Container (容器)** | 镜像的运行实例 |
| **Volume (卷)** | 持久化数据存储 |
| **Network (网络)** | 容器间通信 |
| **Registry (仓库)** | 存储和分发镜像 |

### 容器 vs 虚拟机
| 特性 | 容器 | 虚拟机 |
|------|------|--------|
| 启动时间 | 秒级 | 分钟级 |
| 资源消耗 | 低 | 高 |
| 隔离性 | 进程级 | 完全隔离 |
| 性能 | 接近原生 | 有一定损耗 |
| 系统支持 | 仅Linux | 全平台 |

---

## Docker 基本命令

### 镜像操作
```bash
# 搜索镜像
docker search nginx

# 拉取镜像
docker pull nginx:latest
docker pull ubuntu:22.04

# 查看本地镜像
docker images
docker image ls

# 删除镜像
docker rmi nginx:latest
docker image prune -a  # 删除所有未使用的镜像

# 构建镜像
docker build -t myapp:1.0 .

# 标签镜像
docker tag myapp:1.0 myregistry.com/myapp:1.0

# 推送镜像
docker push myregistry.com/myapp:1.0

# 查看镜像详情
docker inspect nginx:latest

# 查看镜像历史
docker history nginx:latest
```

### 容器操作
```bash
# 创建并启动容器
docker run -it --name mycontainer ubuntu:latest /bin/bash
docker run -d --name webserver nginx:latest

# 参数说明
# -i: 交互式
# -t: 分配伪终端
# -d: 后台运行
# --name: 容器名称
# -p: 端口映射 (主机端口:容器端口)
# -v: 卷挂载 (主机路径:容器路径)
# -e: 环境变量
# --network: 网络模式
# --rm: 容器停止后自动删除

# 列出运行中的容器
docker ps

# 列出所有容器（包括已停止）
docker ps -a

# 启动/停止/重启容器
docker start mycontainer
docker stop mycontainer
docker restart mycontainer

# 进入运行中的容器
docker exec -it mycontainer /bin/bash
docker exec -it webserver sh

# 查看容器详情
docker inspect mycontainer

# 查看容器日志
docker logs -f mycontainer
docker logs --tail 100 mycontainer

# 复制文件
docker cp local/file mycontainer:/container/path
docker cp mycontainer:/container/file local/path

# 暂停/恢复容器
docker pause mycontainer
docker unpause mycontainer

# 删除容器
docker rm mycontainer
docker rm -f mycontainer  # 强制删除运行中的容器
docker container prune    # 删除所有已停止的容器

# 查看容器资源使用
docker stats
docker stats mycontainer

# 查看容器进程
docker top mycontainer

# 重命名容器
docker rename old_name new_name

# 更新容器配置
docker update --memory 512m --cpus 1 mycontainer
```

### 网络操作
```bash
# 创建网络
docker network create mynetwork
docker network create --driver bridge mybridge

# 列出网络
docker network ls

# 连接容器到网络
docker network connect mynetwork mycontainer

# 断开容器网络
docker network disconnect mynetwork mycontainer

# 删除网络
docker network rm mynetwork

# 查看网络详情
docker network inspect mynetwork
```

### 卷操作
```bash
# 创建卷
docker volume create myvolume

# 列出卷
docker volume ls

# 查看卷详情
docker volume inspect myvolume

# 删除未使用的卷
docker volume prune

# 使用卷创建容器
docker run -v myvolume:/data nginx
```

---

## Dockerfile 使用

### 基本结构
```dockerfile
# 基础镜像
FROM python:3.11-slim

# 维护者信息
LABEL maintainer="your.email@example.com"

# 设置工作目录
WORKDIR /app

# 复制文件
COPY requirements.txt .
COPY . .

# 安装依赖
RUN pip install --no-cache-dir -r requirements.txt

# 设置环境变量
ENV APP_ENV=production
ENV PORT=8080

# 暴露端口
EXPOSE 8080

# 创建用户
RUN useradd -m appuser
USER appuser

# 启动命令
CMD ["python", "app.py"]
```

### 常用指令

#### FROM
```dockerfile
FROM python:3.11-slim
FROM nginx:alpine
FROM node:18-alpine
```

#### RUN
```dockerfile
RUN apt-get update && apt-get install -y \
    curl \
    git \
    vim \
    && rm -rf /var/lib/apt/lists/*

RUN pip install --no-cache-dir flask==2.0.0
```

#### COPY vs ADD
```dockerfile
# COPY - 推荐使用
COPY requirements.txt /app/
COPY ./src /app/src/

# ADD - 可以处理URL和tar文件
ADD https://example.com/file.tar.gz /app/
ADD app.tar.gz /app/
```

#### CMD vs ENTRYPOINT
```dockerfile
# CMD - 可被覆盖
CMD ["python", "app.py"]
CMD echo "Hello"

# ENTRYPOINT - 不可被覆盖，常用于可执行容器
ENTRYPOINT ["python", "app.py"]
```

### 多阶段构建
```dockerfile
# 第一阶段：构建
FROM node:18 AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# 第二阶段：运行
FROM node:18-alpine
WORKDIR /app
COPY --from=builder /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
CMD ["node", "dist/index.js"]
```

### .dockerignore
```
node_modules
npm-debug.log
.git
.gitignore
.env
dist
coverage
*.md
```

### 最佳实践
```dockerfile
# 使用特定版本而不是latest
FROM python:3.11-slim

# 使用国内镜像源
RUN pip install -i https://pypi.tuna.tsinghua.edu.cn/simple -r requirements.txt

# 合并RUN指令减少层数
RUN apt-get update && \
    apt-get install -y curl vim && \
    apt-get clean

# 使用 WORKDIR 而不是 cd
WORKDIR /app

# 使用 COPY 而不是 ADD
COPY . .

# 设置非root用户
RUN useradd -m appuser && chown -R appuser:appuser /app
USER appuser

# 使用 exec form 避免shell处理
CMD ["python", "-m", "http.server", "8000"]
```

---

## Docker Compose

### 基本结构
```yaml
version: '3.8'

services:
  web:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8000:8000"
    environment:
      - ENV=production
      - DB_HOST=postgres
    depends_on:
      - db
      - redis
    volumes:
      - ./data:/app/data
    networks:
      - app-network
    restart: unless-stopped

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: user
      POSTGRES_PASSWORD: password
    volumes:
      - db_data:/var/lib/postgresql/data
    networks:
      - app-network
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U user -d myapp"]
      interval: 10s
      timeout: 5s
      retries: 5

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    networks:
      - app-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3

volumes:
  db_data:

networks:
  app-network:
    driver: bridge
```

### 常用命令
```bash
# 启动所有服务
docker-compose up -d

# 启动特定服务
docker-compose up -d web

# 停止所有服务
docker-compose down

# 停止并删除卷
docker-compose down -v

# 重新构建并启动
docker-compose up -d --build

# 查看日志
docker-compose logs -f
docker-compose logs -f web

# 查看服务状态
docker-compose ps

# 执行命令
docker-compose exec web python manage.py migrate

# 扩展服务
docker-compose up -d --scale web=3

# 暂停/恢复服务
docker-compose pause
docker-compose unpause

# 查看服务详情
docker-compose top
```

---

## 镜像相关

### 镜像层结构
```
镜像由多个只读层组成
┌─────────────────────────┐
│         Layer 5         │  (CMD/ENTRYPOINT)
├─────────────────────────┤
│         Layer 4         │  (ENV/EXPOSE)
├─────────────────────────┤
│         Layer 3         │  (RUN)
├─────────────────────────┤
│         Layer 2         │  (COPY)
├─────────────────────────┤
│         Layer 1         │  (WORKDIR)
├─────────────────────────┤
│         Layer 0         │  (FROM base image)
└─────────────────────────┘
```

### 导出和导入
```bash
# 导出镜像为tar文件
docker save -o myimage.tar myapp:1.0

# 导出所有镜像
docker save -o allimages.tar $(docker images -q)

# 导入tar文件为镜像
docker load -i myimage.tar

# 导出容器为镜像
docker commit mycontainer myapp:1.0
```

---

## 容器相关

### 容器生命周期
```
Created → Running → Stopped → Deleted
              ↓
           Paused
```

### 资源限制
```bash
# 内存限制
docker run -d --name web --memory 512m nginx

# CPU限制
docker run -d --name web --cpus 1.5 nginx

# 内存和CPU同时限制
docker run -d --name web \
  --memory 512m \
  --memory-swap 1g \
  --cpus 1.5 \
  --cpuset-cpus 0-1 \
  nginx
```

### 健康检查
```bash
# 在Dockerfile中定义
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD curl -f http://localhost/ || exit 1

# 在docker run中指定
docker run -d --name web \
  --health-cmd="curl -f http://localhost/ || exit 1" \
  --health-interval=30s \
  --health-timeout=10s \
  --health-retries=3 \
  nginx
```

---

## 日志查看

### 容器日志
```bash
# 实时查看日志
docker logs -f mycontainer

# 查看最近100行
docker logs --tail 100 mycontainer

# 查看指定时间范围
docker logs --since "2024-01-01T00:00:00" mycontainer
docker logs --since 30m mycontainer

# 查看错误日志
docker logs mycontainer 2>&1 | grep -i error

# 导出日志到文件
docker logs mycontainer > mycontainer.log 2>&1
```

### 系统日志
```bash
# 查看Docker daemon日志
journalctl -u docker.service

# 查看所有容器资源使用
docker stats

# 查看特定容器资源
docker stats mycontainer --no-stream

# 查看容器进程
docker top mycontainer
```

### 日志驱动配置
```bash
# 查看当前日志驱动
docker info | grep "Logging Driver"

# 使用json-file驱动
docker run -d --log-driver json-file --log-opt max-size=10m --log-opt max-file=3 nginx

# 使用syslog驱动
docker run -d --log-driver=syslog --log-opt syslog-address=tcp://localhost:514 nginx
```

---

## 镜像包结构

### 镜像存储位置
- Linux: `/var/lib/docker/`
- Windows: `C:\ProgramData\Docker`

### 镜像目录结构
```
/var/lib/docker/
├── image/
│   └── overlay2/
│       ├── distributions.json
│       └── imagedb
├── overlay2/
│   └── (镜像层数据)
├── containers/
│   └── (容器数据)
├── volumes/
│   └── (卷数据)
└── network/
    └── (网络数据)
```

### 镜像内容查看
```bash
# 导出镜像查看内容
docker save nginx:latest -o nginx.tar
tar -tvf nginx.tar

# 进入容器查看文件系统
docker run -it nginx ls -la /
```

---

## 容器内文件结构

### 典型容器文件系统
```
/
├── bin/              # 二进制文件
├── boot/             # 启动文件
├── dev/              # 设备文件
├── etc/              # 配置文件
│   ├── nginx/
│   │   └── nginx.conf
│   ├── apt/
│   │   └── sources.list
│   └── ssl/
├── home/             # 用户目录
├── lib/              # 系统库
├── media/            # 媒体文件
├── mnt/              # 挂载点
├── opt/              # 可选应用
├── proc/             # 进程信息
├── root/             # root目录
├── run/              # 运行文件
├── sbin/             # 系统二进制
├── srv/              # 服务数据
├── sys/              # 系统信息
├── tmp/              # 临时文件
├── usr/              # 用户程序
│   ├── bin/
│   ├── games/
│   ├── include/
│   ├── lib/
│   └── sbin/
└── var/              # 变量数据
    ├── log/
    │   └── nginx/
    ├── www/
    │   └── html/
    ├── cache/
    └── lib/
```

### Ubuntu容器结构
```
/etc/                 # 系统配置
├── apt/sources.list
├── nginx/nginx.conf
├── passwd
├── group
└── hostname

/var/                # 应用数据
├──/log/nginx/
└──/www/html/

/usr/bin/            # 用户命令
/bin/                # 基本命令
```

### Alpine容器结构
```
/etc/                # 配置文件
├── apk/
│   └── world
├── nginx/
│   └── nginx.conf
└── passwd

/usr/bin/            # 程序
/bin/                # 链接到/usr/bin
/lib/                # 库文件
```

### 常用容器路径
| 镜像 | 配置文件路径 | 日志路径 | 数据路径 |
|------|-------------|---------|----------|
| nginx | `/etc/nginx/` | `/var/log/nginx/` | `/var/www/html/` |
| mysql | `/etc/mysql/` | `/var/log/mysql/` | `/var/lib/mysql/` |
| postgres | `/etc/postgresql/` | `/var/log/postgresql/` | `/var/lib/postgresql/` |
| redis | `/etc/redis/` | `/var/log/redis/` | `/var/lib/redis/` |
| mongodb | `/etc/mongodb/` | `/var/log/mongodb/` | `/var/lib/mongodb/` |
| node | - | - | `/app/` |
| python | - | - | `/app/` |
