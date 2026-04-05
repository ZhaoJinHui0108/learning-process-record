# API 网关学习笔记

## 目录
1. [API 网关简介](#api-网关简介)
2. [Kong 网关详解](#kong-网关详解)
3. [使用场景](#使用场景)
4. [核心功能配置](#核心功能配置)
5. [Docker 部署 Kong](#docker-部署-kong)
6. [Kong 日志查看](#kong-日志查看)

---

## API 网关简介

### 什么是 API 网关
API网关是一个服务器，是系统的单一入口点。它类似于API的"前门"，所有客户端请求都通过API网关进入系统，API网关负责请求路由、组合、协议转换等功能。

### API 网关的作用
- **统一入口**: 为所有客户端提供统一的API访问点
- **认证授权**: 集中处理身份验证和权限控制
- **路由转发**: 根据请求路径转发到后端服务
- **负载均衡**: 分配请求到多个后端服务实例
- **限流熔断**: 保护后端服务免受过载
- **协议转换**: 支持多种协议(HTTP、gRPC、WebSocket等)
- **日志监控**: 记录所有请求日志
- **缓存处理**: 缓存频繁访问的数据
- **请求/响应转换**: 修改请求或响应的格式

### 主流 API 网关
| 网关 | 开发语言 | 特点 | 适用场景 |
|------|----------|------|----------|
| **Kong** | Lua/NGINX | 功能丰富，插件众多 | 中大型系统 |
| **APISIX** | Lua/NGINX | 高性能，云原生 | 云原生应用 |
| **TyK** | Go | 轻量级，易部署 | 中小型系统 |
| **Nginx** | C | 高性能，成熟稳定 | 反向代理 |
| **Traefik** | Go | 云原生，动态配置 | 容器环境 |
| **Zuul** | Java | Spring生态集成 | Spring Cloud |

---

## Kong 网关详解

### Kong 架构
```
┌─────────────────────────────────────────────────────────┐
│                        Kong Gateway                       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│   ┌─────────────────────────────────────────────────┐  │
│   │                  Admin API                        │  │
│   │          (管理Services、Routes、Plugins)          │  │
│   └─────────────────────────────────────────────────┘  │
│                           │                              │
│   ┌─────────────────────────────────────────────────┐  │
│   │                  Data Plane                      │  │
│   │              (处理请求/响应)                       │  │
│   └─────────────────────────────────────────────────┘  │
│                           │                              │
│   ┌─────────────────────────────────────────────────┐  │
│   │                    Plugins                        │  │
│   │  (认证、限流、监控、日志、转换等)                 │  │
│   └─────────────────────────────────────────────────┘  │
│                                                         │
└─────────────────────────────────────────────────────────┘
          │                                        │
          ▼                                        ▼
┌─────────────────┐                    ┌─────────────────┐
│   PostgreSQL    │                    │    Upstream     │
│   (配置存储)     │                    │   (后端服务)     │
└─────────────────┘                    └─────────────────┘
```

### Kong 核心概念

#### Service（服务）
```bash
# 创建Service
curl -X POST http://localhost:8001/services \
  --data "name=user-service" \
  --data "url=http://user-service:8000"

# 查看所有Service
curl http://localhost:8001/services

# 查看单个Service
curl http://localhost:8001/services/user-service

# 更新Service
curl -X PATCH http://localhost:8001/services/user-service \
  --data "url=http://new-user-service:8000"

# 删除Service
curl -X DELETE http://localhost:8001/services/user-service
```

#### Route（路由）
```bash
# 创建Route
curl -X POST http://localhost:8001/services/user-service/routes \
  --data "name=user-route" \
  --data "paths[]=/users" \
  --data "strip_path=false"

# 创建带多种匹配的Route
curl -X POST http://localhost:8001/services/user-service/routes \
  --data "name=api-route" \
  --data "paths[]=/api/v1" \
  --data "methods[]=GET" \
  --data "methods[]=POST" \
  --data "hosts[]=api.example.com"

# 查看Route列表
curl http://localhost:8001/routes

# 删除Route
curl -X DELETE http://localhost:8001/routes/api-route
```

#### Upstream（上游）
```bash
# 创建Upstream
curl -X POST http://localhost:8001/upstreams \
  --data "name=user-upstream"

# 添加Target（后端实例）
curl -X POST http://localhost:8001/upstreams/user-upstream/targets \
  --data "target=user-service-1:8000" \
  --data "weight=100"

curl -X POST http://localhost:8001/upstreams/user-upstream/targets \
  --data "target=user-service-2:8000" \
  --data "weight=100"

# 启用健康检查
curl -X POST http://localhost:8001/upstreams/user-upstream/health \
  --data "active.healthy.intervals=5" \
  --data "active.unhealthy.tcpfailures=3"
```

---

## 使用场景

### 1. 微服务入口
```
                    ┌──────────────┐
                    │  Kong Gateway │
                    └──────┬───────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
          ▼                ▼                ▼
   ┌────────────┐    ┌────────────┐    ┌────────────┐
   │User Service│    │Order Service│   │Pay Service │
   └────────────┘    └────────────┘    └────────────┘
```

### 2. 认证授权
```bash
# 启用JWT认证插件
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data "name=jwt" \
  --data "config.secret_is_base64=false"

# 启用Key Auth插件
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data "name=key-auth"
```

### 3. 限流
```bash
# 启用限流插件
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data "name=rate-limiting" \
  --data "config.minute=100" \
  --data "config.policy=local"

# 使用Redis的限流
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data "name=rate-limiting" \
  --data "config.minute=1000" \
  --data "config.policy=redis" \
  --data "config.redis.host=redis" \
  --data "config.redis.port=6379"
```

### 4. 请求转换
```bash
# 请求头转换
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data "name=request-transformer" \
  --data "config.add.headers[]=X-Custom-Header:value" \
  --data "config.add.headers[]=X-Request-ID:value"

# 请求参数转换
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data "name=request-transformer" \
  --data "config.add.querystring[]=api-version:v1"
```

---

## 核心功能配置

### 负载均衡配置
```bash
# 创建Upstream用于负载均衡
curl -X POST http://localhost:8001/upstreams \
  --data "name=backend-upstream" \
  --data "algorithm=least-connections"

# 添加多个Target
curl -X POST http://localhost:8001/upstreams/backend-upstream/targets \
  --data "target=backend-1:8000" \
  --data "weight=80"

curl -X POST http://localhost:8001/upstreams/backend-upstream/targets \
  --data "target=backend-2:8000" \
  --data "weight=80"

# 创建Service使用Upstream
curl -X POST http://localhost:8001/services \
  --data "name=backend-service" \
  --data "host=backend-upstream" \
  --data "path=/api"

# 创建Route
curl -X POST http://localhost:8001/services/backend-service/routes \
  --data "name=backend-route" \
  --data "paths[]=/backend"
```

### 认证配置
```bash
# OAuth2认证
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data "name=oauth2" \
  --data "config.scopes=read,write" \
  --data "config.token_expiration=7200"

# LDAP认证
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data "name=ldap-auth" \
  --data "config.ldap_host=ldap.example.com" \
  --data "config.ldap_port=389"

# Basic Auth
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data "name=basic-auth" \
  --data "config.hide_credentials=true"
```

### 限流和熔断配置
```bash
# 限流插件
curl -X POST http://localhost:8001/global/plugins \
  --data "name=rate-limiting" \
  --data "config.minute=100" \
  --data "config.limit=100"

# 响应限流（限制每秒请求数）
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data "name=response-ratelimiting" \
  --data "config.limit=100" \
  --data "config.limit_name=something"

# 熔断插件（需要购买企业版或使用社区插件）
```

### 日志和监控
```bash
# 访问日志到文件
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data "name=file-log" \
  --data "config.path=/var/log/kong/access.log"

# 访问日志到Syslog
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data "name=syslog" \
  --data "config.log_level=info"

# Prometheus监控
curl -X POST http://localhost:8001/plugins \
  --data "name=prometheus" \
  --data "config.per_consumer=true"

# 请求日志
curl -X POST http://localhost:8001/services/user-service/plugins \
  --data "name=http-log" \
  --data "config.http_endpoint=http://logging-server:8080/logs"
```

---

## Docker 部署 Kong

### 方式一：使用 docker-compose（推荐）

#### 数据库模式（PostgreSQL）
```yaml
# docker-compose.yml
version: '3.8'

services:
  kong-database:
    image: postgres:13
    container_name: kong-database
    environment:
      POSTGRES_DB: kong
      POSTGRES_USER: kong
      POSTGRES_PASSWORD: kong_password
    volumes:
      - kong_data:/var/lib/postgresql/data
    restart: unless-stopped
    networks:
      - kong-net

  kong:
    image: kong:3.4
    container_name: kong
    depends_on:
      - kong-database
    ports:
      - "8000:8000"    # HTTP代理
      - "8443:8443"    # HTTPS代理
      - "8001:8001"    # Admin HTTP
      - "8444:8444"    # Admin HTTPS
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_USER: kong
      KONG_PG_PASSWORD: kong_password
      KONG_PG_DB: kong
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_PROXY_ERROR_LOG: /dev/stderr
      KONG_ADMIN_ERROR_LOG: /dev/stderr
      KONG_ADMIN_LISTEN: 0.0.0.0:8001
    restart: unless-stopped
    networks:
      - kong-net
    volumes:
      - ./kong.yml:/usr/local/kong/kong.yml

networks:
  kong-net:
    driver: bridge

volumes:
  kong_data:
```

#### DB-less 模式（无数据库）
```yaml
# docker-compose.yml
version: '3.8'

services:
  kong:
    image: kong:3.4
    container_name: kong
    ports:
      - "8000:8000"
      - "8443:8443"
      - "8001:8001"
      - "8444:8444"
    environment:
      KONG_DATABASE: off
      KONG_DECLARATIVE_CONFIG: /usr/local/kong/kong.yml
      KONG_PROXY_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_ACCESS_LOG: /dev/stdout
      KONG_ADMIN_LISTEN: 0.0.0.0:8001
    restart: unless-stopped
    volumes:
      - ./kong.yml:/usr/local/kong/kong.yml
```

#### kong.yml 配置文件
```yaml
# kong.yml (DB-less模式配置)

_format_version: "3.0"

# Services
services:
  - name: user-service
    url: http://user-service:8000
    routes:
      - name: user-route
        paths:
          - /users
        strip_path: false
    plugins:
      - name: rate-limiting
        config:
          minute: 100
          policy: local
      - name: jwt
        config:
          secret_is_base64: false

  - name: order-service
    url: http://order-service:8000
    routes:
      - name: order-route
        paths:
          - /orders
        strip_path: false

# Upstreams (负载均衡)
upstreams:
  - name: backend-upstream
    algorithm: least-connections
    targets:
      - target: backend-1:8000
        weight: 100
      - target: backend-2:8000
        weight: 100
    healthcheck:
      active:
        type: http
        http_path: /health
        healthy:
          interval: 5
          successes: 2
        unhealthy:
          interval: 5
          tcpfailures: 3
```

### 方式二：单独容器启动

```bash
# 1. 启动PostgreSQL
docker run -d \
  --name kong-database \
  -p 5432:5432 \
  -e POSTGRES_DB=kong \
  -e POSTGRES_USER=kong \
  -e POSTGRES_PASSWORD=kong_password \
  postgres:13

# 2. 初始化Kong数据库
docker run --rm \
  --link kong-database \
  -e "KONG_DATABASE=postgres" \
  -e "KONG_PG_HOST=kong-database" \
  -e "KONG_PG_USER=kong" \
  -e "KONG_PG_PASSWORD=kong_password" \
  -e "KONG_PG_DB=kong" \
  kong:3.4 kong migrations bootstrap

# 3. 启动Kong
docker run -d \
  --name kong \
  -p 8000:8000 \
  -p 8443:8443 \
  -p 8001:8001 \
  -p 8444:8444 \
  --link kong-database \
  -e "KONG_DATABASE=postgres" \
  -e "KONG_PG_HOST=kong-database" \
  -e "KONG_PG_USER=kong" \
  -e "KONG_PG_PASSWORD=kong_password" \
  -e "KONG_PG_DB=kong" \
  -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
  -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
  kong:3.4
```

### 带Kong Manager和Dev Portal的配置
```yaml
version: '3.8'

services:
  kong-database:
    image: postgres:13
    environment:
      POSTGRES_DB: kong
      POSTGRES_USER: kong
      POSTGRES_PASSWORD: kong_password
    volumes:
      - kong_data:/var/lib/postgresql/data

  kong:
    image: kong:3.4
    depends_on:
      - kong-database
    ports:
      - "8000:8000"
      - "8443:8443"
      - "8001:8001"
      - "8444:8444"
    environment:
      KONG_DATABASE: postgres
      KONG_PG_HOST: kong-database
      KONG_PG_USER: kong
      KONG_PG_PASSWORD: kong_password
      KONG_PG_DB: kong
      KONG_ADMIN_LISTEN: 0.0.0.0:8001
      KONG_PLUGINS: bundled
      KONG_PROMETHEUS_PLUGIN: enabled
    volumes:
      - ./kong.yml:/usr/local/kong/kong.yml

volumes:
  kong_data:
```

### 常用 Docker 命令

```bash
# 启动所有服务
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看Kong日志
docker logs -f kong

# 进入Kong容器
docker exec -it kong sh

# 测试Kong是否正常运行
curl http://localhost:8001

# 查看Kong节点状态
curl http://localhost:8001/status

# 停止所有服务
docker-compose down

# 删除数据卷
docker-compose down -v
```

---

## Kong 日志查看

### 日志类型

Kong产生的日志：
1. **access.log**: 代理访问日志
2. **admin-api.log**: Admin API日志
3. **error.log**: 错误日志
4. **serf.log**: 集群日志（集群模式下）

### Docker 日志

```bash
# 查看所有日志
docker logs -f kong

# 查看错误日志
docker logs -f kong 2>&1 | grep error

# 查看最近100行
docker logs --tail 100 kong

# 导出日志
docker logs kong > kong.log 2>&1

# 查看特定时间范围
docker logs --since "2024-01-01T00:00:00" kong
```

### Admin API 查看配置

```bash
# 查看所有Service
curl http://localhost:8001/services

# 查看所有Route
curl http://localhost:8001/routes

# 查看所有Plugin
curl http://localhost:8001/plugins

# 查看已启用的插件
curl http://localhost:8001/plugins/enabled

# 查看所有Upstream
curl http://localhost:8001/upstreams

# 查看节点信息
curl http://localhost:8001

# 查看健康状态
curl http://localhost:8001/health
```

### 日志分析

```bash
# 进入Kong容器
docker exec -it kong sh

# 查看日志文件
ls -la /usr/local/kong/logs/

# 查看访问日志
cat /usr/local/kong/logs/access.log

# 查看错误日志
cat /usr/local/kong/logs/error.log

# 实时查看日志
tail -f /usr/local/kong/logs/access.log

# 分析请求统计
cat /usr/local/kong/logs/access.log | awk '{print $1}' | sort | uniq -c | sort -rn

# 查找慢请求
cat /usr/local/kong/logs/access.log | grep -E "latency|request_time" | head -20
```

### Prometheus 监控

```bash
# 启用Prometheus插件
curl -X POST http://localhost:8001/plugins \
  --data "name=prometheus"

# 获取监控指标
curl http://localhost:8001/metrics

# 常用指标
# kong_http_requests_total - 总请求数
# kong_http_requests_latency_ms - 请求延迟
# kong_upstream_latency_ms - 上游延迟
# kong_bandwidth_bytes - 带宽使用
# kong_connections_accepted - 接受的连接数
# kong_connections_active - 活跃连接数
# kong_connections_idle - 空闲连接数
```

### 健康检查

```bash
# 查看Upstream健康状态
curl http://localhost:8001/upstreams/backend-upstream/health

# 手动触发健康检查
curl -X POST http://localhost:8001/upstreams/backend-upstream/health

# 查看所有Target健康状态
curl http://localhost:8001/upstreams/backend-upstream/targets/all

# 禁用Target
curl -X DELETE http://localhost:8001/upstreams/backend-upstream/targets/backend-1:8000

# 启用Target
curl -X POST http://localhost:8001/upstreams/backend-upstream/targets \
  --data "target=backend-1:8000" \
  --data "weight=100"
```
