# Nginx 学习笔记

## 目录
1. [Nginx 简介](#nginx-简介)
2. [Nginx 使用场景](#nginx-使用场景)
3. [Nginx 核心概念](#nginx-核心概念)
4. [Nginx 配置详解](#nginx-配置详解)
5. [Docker 部署 Nginx](#docker-部署-nginx)
6. [Nginx 日志查看](#nginx-日志查看)

---

## Nginx 简介

### 什么是 Nginx
Nginx（发音为"engine-x"）是一款高性能的开源HTTP服务器和反向代理服务器，由Igor Sysoev于2004年首次发布。它也可以用作IMAP/POP3代理服务器。

### Nginx 的特点
- **高性能**: 事件驱动架构，支持高并发连接
- **低内存消耗**: 相比Apache，内存占用更低
- **稳定性强**: 运行稳定，很少出现崩溃
- **热部署**: 支持不停机更新配置
- **模块化设计**: 功能可通过模块扩展
- **负载均衡**: 内置负载均衡功能
- **反向代理**: 支持正向和反向代理
- **SSL/TLS**: 支持HTTPS

### Nginx vs Apache
| 特性 | Nginx | Apache |
|------|-------|--------|
| 架构 | 事件驱动 | 进程/线程驱动 |
| 并发性能 | 极高 | 中等 |
| 内存消耗 | 低 | 较高 |
| 静态内容 | 非常高效 | 高效 |
| 动态内容 | 需要FastCGI | 原生支持 |
| 配置文件 | 简洁 | 复杂 |
| 模块支持 | 静态编译 | 动态加载 |

---

## Nginx 使用场景

### 1. Web 服务器
```nginx
# 最基本的Web服务器配置
server {
    listen 80;
    server_name example.com;
    root /var/www/html;
    
    location / {
        index index.html index.htm;
    }
}
```

### 2. 反向代理
```nginx
# 将请求转发到后端应用服务器
server {
    listen 80;
    server_name api.example.com;
    
    location / {
        proxy_pass http://127.0.0.1:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 3. 负载均衡
```nginx
# 负载均衡到多个后端服务器
upstream backend {
    least_conn;  # 最少连接数
    server 127.0.0.1:8001;
    server 127.0.0.1:8002;
    server 127.0.0.1:8003;
}

server {
    listen 80;
    server_name myapp.com;
    
    location / {
        proxy_pass http://backend;
    }
}
```

### 4. SSL/TLS 终端
```nginx
# HTTPS服务器
server {
    listen 443 ssl http2;
    server_name example.com;
    
    ssl_certificate /etc/nginx/ssl/cert.pem;
    ssl_certificate_key /etc/nginx/ssl/key.pem;
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;
    
    location / {
        root /var/www/html;
    }
}
```

### 5. 静态资源服务
```nginx
# 静态资源服务器
server {
    listen 80;
    server_name static.example.com;
    root /var/www/static;
    
    # 静态资源缓存
    location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
        expires 30d;
        add_header Cache-Control "public, immutable";
    }
    
    # Gzip压缩
    location ~* \.(js|css)$ {
        gzip on;
        gzip_types text/css application/javascript;
    }
}
```

### 6. API 网关
```nginx
# API网关配置
server {
    listen 80;
    server_name api.example.com;
    
    # 用户API
    location /api/users/ {
        proxy_pass http://user-service/;
        rewrite ^/api/users/(.*)$ /$1 break;
    }
    
    # 订单API
    location /api/orders/ {
        proxy_pass http://order-service/;
        rewrite ^/api/orders/(.*)$ /$1 break;
    }
    
    # 产品API
    location /api/products/ {
        proxy_pass http://product-service/;
        rewrite ^/api/products/(.*)$ /$1 break;
    }
}
```

### 7. 限流
```nginx
# 限流配置
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=10r/s;

server {
    listen 80;
    
    location /api/ {
        limit_req zone=api_limit burst=20 nodelay;
        proxy_pass http://backend;
    }
}
```

### 8. 重写规则
```nginx
# URL重写
server {
    listen 80;
    server_name example.com;
    
    # 将 /news/123 重写为 /article.php?id=123
    location /news/ {
        rewrite ^/news/(\d+)$ /article.php?id=$1 last;
    }
    
    # 将 /products/category/item 重写为 /item.php?category=category&item=item
    location /products/ {
        rewrite ^/products/([^/]+)/([^/]+)$ /item.php?category=$1&item=$2 last;
    }
}
```

---

## Nginx 核心概念

### 进程架构
```
┌─────────────────────────────────────┐
│           Master Process            │
│         (管理/监控进程)              │
├─────────────────────────────────────┤
│    Worker Process 1                 │
│    Worker Process 2                 │
│    Worker Process 3                 │
│    Worker Process N                 │
│         (工作进程，处理请求)         │
└─────────────────────────────────────┘
```

### 配置文件结构
```nginx
# 全局块
worker_processes 4;
error_log /var/log/nginx/error.log;

events {
    # events块
    worker_connections 1024;
}

http {
    # http块
    include /etc/nginx/mime.types;
    
    # upstream块
    upstream backend {
        server 127.0.0.1:8000;
    }
    
    # server块
    server {
        # server块
        listen 80;
        server_name example.com;
        
        # location块
        location / {
            root /var/www/html;
        }
        
        location /api/ {
            proxy_pass http://backend;
        }
    }
}
```

### 常用变量
| 变量 | 说明 |
|------|------|
| `$host` | 请求的主机名 |
| `$remote_addr` | 客户端IP地址 |
| `$remote_port` | 客户端端口 |
| `$request_method` | 请求方法(GET/POST等) |
| `$request_uri` | 完整请求URI |
| `$uri` | 当前URI(不含参数) |
| `$args` | 请求参数 |
| `$http_user_agent` | 用户代理 |
| `$http_cookie` | Cookie信息 |
| `$status` | 响应状态码 |
| `$body_bytes_sent` | 发送的字节数 |
| `$request_time` | 请求处理时间 |

### 负载均衡算法
| 算法 | 说明 | 配置 |
|------|------|------|
| 轮询 | 依次分配 | 默认 |
| 最少连接 | 连接数最少优先 | `least_conn` |
| IP哈希 | 同一IP同一服务器 | `ip_hash` |
| 权重 | 按权重分配 | `weight` |
| 响应时间 | 响应最快优先 | `fair` |
| URL哈希 | 按URL hash | `hash` |

---

## Nginx 配置详解

### 基本配置
```nginx
# nginx.conf

# 全局配置
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log warn;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
    use epoll;  # Linux高效事件模型
    multi_accept on;
}

http {
    # 基础配置
    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    
    # 日志格式
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
    
    access_log /var/log/nginx/access.log main;
    
    # 性能配置
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    
    # Gzip压缩
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_comp_level 6;
    gzip_types text/plain text/css text/xml application/json application/javascript application/rss+xml application/atom+xml image/svg+xml;
    
    # upstream配置
    upstream api_backend {
        server 127.0.0.1:8001;
        server 127.0.0.1:8002;
        server 127.0.0.1:8003;
    }
    
    # 虚拟主机配置
    server {
        listen 80;
        server_name example.com www.example.com;
        
        root /var/www/html;
        index index.html index.htm;
        
        # 字符集
        charset utf-8;
        
        # 访问控制
        allow 192.168.1.0/24;
        allow 10.0.0.0/8;
        deny all;
        
        # 日志
        access_log /var/log/nginx/example.com.access.log main;
        error_log /var/log/nginx/example.com.error.log;
        
        # Location配置
        location / {
            try_files $uri $uri/ /index.html;
        }
        
        # 静态资源
        location /static/ {
            alias /var/www/static/;
            expires 30d;
            add_header Cache-Control "public, immutable";
        }
        
        # 反向代理
        location /api/ {
            proxy_pass http://api_backend;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
            
            # 超时配置
            proxy_connect_timeout 60s;
            proxy_send_timeout 60s;
            proxy_read_timeout 60s;
            
            # 缓冲配置
            proxy_buffering on;
            proxy_buffer_size 4k;
            proxy_buffers 8 4k;
        }
        
        # FastCGI (PHP)
        location ~ \.php$ {
            fastcgi_pass unix:/var/run/php-fpm/php-fpm.sock;
            fastcgi_index index.php;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }
        
        # 禁止访问
        location ~ /\. {
            deny all;
            access_log off;
            log_not_found off;
        }
        
        # 错误页面
        error_page 404 /404.html;
        error_page 500 502 503 504 /50x.html;
    }
    
    # HTTPS配置
    server {
        listen 443 ssl http2;
        server_name example.com;
        
        ssl_certificate /etc/nginx/ssl/cert.pem;
        ssl_certificate_key /etc/nginx/ssl/key.pem;
        
        ssl_protocols TLSv1.2 TLSv1.3;
        ssl_ciphers ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256;
        ssl_prefer_server_ciphers off;
        
        ssl_session_cache shared:SSL:10m;
        ssl_session_timeout 1d;
        
        location / {
            root /var/www/html;
        }
    }
}
```

### location 匹配规则
```nginx
# 精确匹配
location = / {
    # 只匹配 /
}

# 前缀匹配
location / {
    # 匹配所有以 / 开头的URI
}

# ^~ 优先前缀匹配（不检查正则）
location ^~ /static/ {
    # 匹配以 /static/ 开头的URI，停止搜索正则
}

# 正则匹配
location ~ \.php$ {
    # 区分大小写的正则
}

location ~* \.(jpg|jpeg|png)$ {
    # 不区分大小写的正则
}

# 命名location
location @fallback {
    proxy_pass http://backend;
}
```

### 常用配置示例

#### 前后端分离项目
```nginx
server {
    listen 80;
    server_name myapp.com;
    
    # 前端静态资源
    location / {
        root /var/www/myapp/dist;
        try_files $uri $uri/ /index.html;
    }
    
    # API代理
    location /api/ {
        proxy_pass http://backend-server:8000/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
    
    # WebSocket代理
    location /ws/ {
        proxy_pass http://websocket-server:8080/;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

#### 多域名配置
```nginx
# 主站
server {
    listen 80;
    server_name example.com www.example.com;
    root /var/www/example;
}

# 子域名
server {
    listen 80;
    server_name blog.example.com;
    root /var/www/blog;
}

server {
    listen 80;
    server_name shop.example.com;
    root /var/www/shop;
}
```

#### 灰度发布
```nginx
# 基于Cookie的灰度
map $cookie_version $backend {
    ~^1\.     http://new-backend;
    default   http://old-backend;
}

server {
    location / {
        proxy_pass $backend;
    }
}

# 或基于IP
map $remote_addr $backend {
    ~^192\.168\.  http://new-backend;
    default       http://old-backend;
}
```

---

## Docker 部署 Nginx

### 基本配置

#### Dockerfile
```dockerfile
FROM nginx:alpine
COPY nginx.conf /etc/nginx/nginx.conf
COPY ssl/ /etc/nginx/ssl/
COPY html/ /var/www/html/
EXPOSE 80 443
CMD ["nginx", "-g", "daemon off;"]
```

#### docker-compose.yml
```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    container_name: nginx
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - ./html:/usr/share/nginx/html
      - ./logs:/var/log/nginx
      - ./ssl:/etc/nginx/ssl
    restart: unless-stopped
    networks:
      - web-network

networks:
  web-network:
    driver: bridge
```

### 生产环境配置

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    container_name: nginx-prod
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
      - ./conf.d:/etc/nginx/conf.d:ro
      - ./html:/usr/share/nginx/html:ro
      - ./ssl:/etc/nginx/ssl:ro
      - ./logs:/var/log/nginx
    environment:
      - NGINX_HOST=example.com
      - NGINX_PORT=80
    restart: unless-stopped
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    networks:
      - web-network
    depends_on:
      - webapp

  webapp:
    image: my-webapp:latest
    container_name: webapp
    expose:
      - "8000"
    networks:
      - web-network

networks:
  web-network:
    driver: bridge
```

### 带SSL证书的配置

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    container_name: nginx-ssl
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx-ssl.conf:/etc/nginx/nginx.conf:ro
      - ./ssl/fullchain.pem:/etc/nginx/ssl/fullchain.pem:ro
      - ./ssl/privkey.pem:/etc/nginx/ssl/privkey.pem:ro
      - ./html:/usr/share/nginx/html:ro
      - ./logs:/var/log/nginx
    restart: unless-stopped
```

### 负载均衡配置

```yaml
version: '3.8'

services:
  nginx:
    image: nginx:alpine
    container_name: nginx-lb
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx-lb.conf:/etc/nginx/nginx.conf:ro
      - ./logs:/var/log/nginx
    restart: unless-stopped
    networks:
      - app-network

  app1:
    image: myapp:latest
    container_name: app1
    networks:
      - app-network

  app2:
    image: myapp:latest
    container_name: app2
    networks:
      - app-network

  app3:
    image: myapp:latest
    container_name: app3
    networks:
      - app-network

networks:
  app-network:
    driver: bridge
```

### 常用 Docker 命令

```bash
# 启动Nginx
docker start nginx

# 停止Nginx
docker stop nginx

# 重启Nginx
docker restart nginx

# 进入Nginx容器
docker exec -it nginx sh

# 测试配置语法
docker exec nginx nginx -t

# 重新加载配置
docker exec nginx nginx -s reload

# 查看Nginx进程
docker exec nginx ps aux

# 查看Nginx日志
docker logs -f nginx

# 从容器复制文件
docker cp nginx:/etc/nginx/nginx.conf ./nginx.conf

# 复制到容器
docker cp ./nginx.conf nginx:/etc/nginx/nginx.conf
```

---

## Nginx 日志查看

### 日志类型

Nginx有两类日志：
1. **access.log**: 访问日志，记录所有请求
2. **error.log**: 错误日志，记录错误信息

### 默认日志位置
```
/var/log/nginx/access.log    # 访问日志
/var/log/nginx/error.log     # 错误日志
```

### Docker 日志

```bash
# 查看访问日志
docker logs -f nginx

# 查看错误日志
docker logs -f nginx 2>&1

# 查看最近100行
docker logs --tail 100 nginx

# 查看指定时间范围
docker logs --since "2024-01-01T00:00:00" nginx

# 查找错误
docker logs nginx 2>&1 | grep error

# 查找特定IP的请求
docker logs nginx 2>&1 | grep "192.168.1.1"

# 查找404请求
docker logs nginx 2>&1 | grep " 404 "

# 统计IP访问次数
docker logs nginx 2>&1 | awk '{print $1}' | sort | uniq -c | sort -rn | head -20

# 统计请求状态码
docker logs nginx 2>&1 | awk '{print $9}' | sort | uniq -c
```

### 日志格式配置

```nginx
# 自定义日志格式
log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                '$status $body_bytes_sent "$http_referer" '
                '"$http_user_agent" "$http_x_forwarded_for" '
                'rt=$request_time uct="$upstream_connect_time" '
                'uht="$upstream_header_time" urt="$upstream_response_time"';

access_log /var/log/nginx/access.log main;
```

### 常用日志分析命令

```bash
# 查看访问最多的IP
docker exec nginx sh -c 'cat /var/log/nginx/access.log | awk "{print \$1}" | sort | uniq -c | sort -rn | head -10'

# 查看最常访问的URL
docker exec nginx sh -c 'cat /var/log/nginx/access.log | awk "{print \$7}" | sort | uniq -c | sort -rn | head -10'

# 查看响应时间最长的请求
docker exec nginx sh -c 'cat /var/log/nginx/access.log | awk "{if(\$NF ~ /[0-9]+/) print \$NF, \$0}" | sort -rn | head -10'

# 查看带宽使用
docker exec nginx sh -c 'cat /var/log/nginx/access.log | awk "{sum+=\$10} END {print sum/1024/1024 " MB"}'

# 实时查看访问状态码统计
docker exec nginx sh -c 'tail -f /var/log/nginx/access.log | awk "{print \$9}" | sort | uniq -c'

# 分析爬虫访问
docker exec nginx sh -c 'cat /var/log/nginx/access.log | grep -i "bot\|spider\|crawler" | awk "{print \$1}" | sort | uniq -c'
```

### 日志轮转配置

```nginx
# /etc/logrotate.d/nginx
/var/log/nginx/*.log {
    daily
    missingok
    rotate 14
    compress
    delaycompress
    notifempty
    create 0640 nginx adm
    sharedscripts
    postrotate
        docker exec nginx nginx -s reload
    endscript
}
```

### 监控脚本示例

```bash
#!/bin/bash
# nginx_monitor.sh

echo "=== Nginx Status ==="
docker exec nginx nginx -v

echo -e "\n=== Connection Stats ==="
docker exec nginx sh -c 'cat /var/log/nginx/access.log | wc -l'

echo -e "\n=== Recent Errors ==="
docker exec nginx sh -c 'tail -5 /var/log/nginx/error.log'

echo -e "\n=== Status Code Distribution ==="
docker exec nginx sh -c 'awk "{print \$9}" /var/log/nginx/access.log | sort | uniq -c'

echo -e "\n=== Top 10 IPs ==="
docker exec nginx sh -c 'awk "{print \$1}" /var/log/nginx/access.log | sort | uniq -c | sort -rn | head -10'
```
