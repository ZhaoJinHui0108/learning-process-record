# Redis 学习笔记

## 目录
1. [Redis 简介](#redis-简介)
2. [Redis 使用场景](#redis-使用场景)
3. [Redis 数据类型](#redis-数据类型)
4. [Python 调用 Redis](#python-调用-redis)
5. [Docker 部署 Redis](#docker-部署-redis)
6. [Redis 日志查看](#redis-日志查看)

---

## Redis 简介

### 什么是 Redis
Redis（Remote Dictionary Server）是一个开源的、基于内存的、键值对存储数据库，由Salvatore Sanfilippo于2009年创建。它通常用作数据库、缓存和消息队列。

### Redis 的特点
- **高性能**: 基于内存，读写速度极快
- **丰富的数据类型**: 支持字符串、哈希、列表、集合、有序集合等
- **持久化**: 支持RDB和AOF两种持久化方式
- **高可用**: 支持主从复制、哨兵模式和集群模式
- **原子操作**: 所有操作都是原子性的
- **发布/订阅**: 支持消息发布和订阅功能
- **事务**: 支持有限的事务功能

### Redis vs 其他数据库
| 特性 | Redis | 关系型数据库 | Memcached |
|------|-------|--------------|------------|
| 数据类型 | 多种 | 行列结构 | 键值对 |
| 持久化 | 支持 | 支持 | 不支持 |
| 事务 | 有限 | 完全支持 | 不支持 |
| 性能 | 极高 | 中等 | 高 |
| 内存存储 | 是 | 否 | 可选 |

---

## Redis 使用场景

### 1. 缓存
这是Redis最常用的场景
```python
# 缓存用户信息
user_cache_key = f"user:{user_id}"
cached_user = redis.get(user_cache_key)

if cached_user:
    user = json.loads(cached_user)
else:
    user = db.get_user(user_id)
    redis.setex(user_cache_key, 3600, json.dumps(user))  # 缓存1小时
```

### 2. 会话存储
```python
# 存储用户会话
session_key = f"session:{session_id}"
redis.setex(session_key, 86400, json.dumps(session_data))
```

### 3. 实时排行榜
```python
# 使用有序集合实现排行榜
redis.zadd("ranking:2024", {"player1": 100, "player2": 95, "player3": 90})

# 获取前10名
top_players = redis.zrevrange("ranking:2024", 0, 9, withscores=True)

# 更新分数
redis.zincrby("ranking:2024", 10, "player1")
```

### 4. 分布式锁
```python
# 获取锁
lock_key = f"lock:order:{order_id}"
is_locked = redis.set(lock_key, "1", nx=True, ex=30)

if is_locked:
    try:
        # 执行业务逻辑
        process_order(order_id)
    finally:
        redis.delete(lock_key)
```

### 5. 消息队列
```python
# 生产消息
redis.lpush("queue:tasks", json.dumps(task_data))

# 消费消息
task = redis.rpop("queue:tasks")
if task:
    process_task(json.loads(task))
```

### 6. 限流
```python
# 滑动窗口限流
def rate_limit(user_id, limit=100, window=60):
    key = f"rate:{user_id}"
    now = time.time()
    
    # 删除窗口外的记录
    redis.zremrangebyscore(key, 0, now - window)
    
    # 添加当前请求
    redis.zadd(key, {str(now): now})
    
    # 设置过期
    redis.expire(key, window)
    
    # 检查数量
    count = redis.zcard(key)
    return count <= limit
```

### 7. 分布式会话
```python
# 存储用户登录信息
redis.hset(f"user:{user_id}", mapping={
    "username": username,
    "login_time": str(login_time),
    "ip": ip_address
})
redis.expire(f"user:{user_id}", 3600)
```

### 8. 实时计数器
```python
# 页面访问统计
redis.incr(f"page:views:{page_id}")
redis.incr(f"daily:views:{date}")

# 获取统计数据
views = redis.get(f"page:views:{page_id}")
```

---

## Redis 数据类型

### 1. 字符串 (String)
```python
# 基本操作
redis.set("name", "Alice")
redis.get("name")                    # "Alice"
redis.mset({"name": "Bob", "age": "25"})
redis.mget(["name", "age"])           # ["Bob", "25"]

# 数值操作
redis.set("counter", 10)
redis.incr("counter")                 # 11
redis.incrby("counter", 5)            # 16
redis.decr("counter")                 # 15
redis.decrby("counter", 3)            # 12

# 过期操作
redis.setex("temp", 60, "value")     # 60秒后过期
redis.setnx("lock", "1")             # 不存在时设置
redis.ttl("temp")                     # 查看剩余时间
```

### 2. 哈希 (Hash)
```python
# 基本操作
redis.hset("user:1001", "name", "Alice")
redis.hset("user:1001", "email", "alice@example.com")
redis.hget("user:1001", "name")       # "Alice"

# 批量操作
redis.hmset("user:1002", {"name": "Bob", "age": "25"})
redis.hmget("user:1002", ["name", "age"])

# 获取所有字段
redis.hgetall("user:1001")
# {"name": "Alice", "email": "alice@example.com"}

# 其他操作
redis.hkeys("user:1001")            # ["name", "email"]
redis.hvals("user:1001")            # ["Alice", "alice@example.com"]
redis.hlen("user:1001")             # 2
redis.hexists("user:1001", "name")  # True
redis.hincrby("user:1001", "age", 1)
```

### 3. 列表 (List)
```python
# 添加元素
redis.lpush("tasks", "task3")       # 从左边添加
redis.rpush("tasks", "task4")        # 从右边添加

# 获取元素
redis.lrange("tasks", 0, -1)        # 获取所有
redis.lindex("tasks", 0)             # 获取第一个
redis.llen("tasks")                  # 列表长度

# 删除元素
redis.lpop("tasks")                  # 左边弹出
redis.rpop("tasks")                  # 右边弹出
redis.lrem("tasks", 0, "task1")      # 删除所有task1
redis.ltrim("tasks", 0, 9)           # 保留前10个

# 阻塞操作
redis.brpop("tasks", timeout=5)     # 阻塞等待
```

### 4. 集合 (Set)
```python
# 基本操作
redis.sadd("tags", "python", "redis", "database")
redis.smembers("tags")               # 获取所有成员
redis.sismember("tags", "python")     # 检查是否存在
redis.scard("tags")                  # 成员数量

# 集合运算
redis.sadd("set1", 1, 2, 3)
redis.sadd("set2", 2, 3, 4)
redis.sinter("set1", "set2")         # 交集 [2, 3]
redis.sunion("set1", "set2")         # 并集 [1, 2, 3, 4]
redis.sdiff("set1", "set2")          # 差集 [1]

# 随机操作
redis.srandmember("tags")            # 随机获取一个
redis.srandmember("tags", 2)          # 随机获取多个
redis.spop("tags")                   # 随机弹出
```

### 5. 有序集合 (Sorted Set)
```python
# 基本操作
redis.zadd("ranking", {"Alice": 100, "Bob": 90, "Charlie": 95})

# 按分数范围获取
redis.zrange("ranking", 0, -1)                        # 按分数升序
redis.zrevrange("ranking", 0, 2)                      # 前3名
redis.zrangebyscore("ranking", 90, 100)               # 90-100分的

# 分数操作
redis.zincrby("ranking", 5, "Alice")                 # 增加分数
redis.zscore("ranking", "Alice")                      # 获取分数

# 排名操作
redis.zrank("ranking", "Alice")                      # 获取排名（升序）
redis.zrevrank("ranking", "Alice")                    # 获取排名（降序）

# 数量操作
redis.zcount("ranking", 90, 100)                     # 90-100分人数
redis.zcard("ranking")                                # 总人数
```

### 6. 位图 (Bitmap)
```python
# 设置某一位
redis.setbit("sign:2024:01", day_offset, 1)

# 获取某一位
redis.getbit("sign:2024:01", day_offset)

# 统计位数
redis.bitcount("sign:2024:01")                        # 打卡天数

# 位运算
redis.setbit("a", 0, 1)
redis.setbit("b", 0, 1)
redis.bitop("AND", "result", "a", "b")
```

### 7. HyperLogLog
```python
# 添加
redis.pfadd("uv:2024", "user1", "user2", "user3")

# 统计
redis.pfcount("uv:2024")

# 合并
redis.pfmerge("result", "uv:2024:01", "uv:2024:02")
```

### 8. 地理空间 (Geo)
```python
# 添加位置
redis.geoadd("cities", (116.40, 39.90, "Beijing"))
redis.geoadd("cities", (121.47, 31.23, "Shanghai"))

# 计算距离
redis.geodist("cities", "Beijing", "Shanghai", "km")

# 获取位置
redis.geopos("cities", "Beijing")

# 附近的人
redis.georadius("cities", 116.40, 39.90, 100, "km")
```

---

## Python 调用 Redis

### 安装
```bash
pip install redis
```

### 基本连接
```python
import redis

# 基本连接
r = redis.Redis(host='localhost', port=6379, db=0)

# 使用连接池
pool = redis.ConnectionPool(host='localhost', port=6379, db=0, max_connections=10)
r = redis.Redis(connection_pool=pool)

# 使用URL连接
r = redis.from_url('redis://localhost:6379/0')

# 密码连接
r = redis.Redis(host='localhost', port=6379, password='your_password')

# SSL连接
r = redis.Redis(host='localhost', port=6379, ssl=True, ssl_certfile='cert.pem', ssl_keyfile='key.pem')
```

### Redis 客户端方法对照表

| Python方法 | Redis命令 | 说明 |
|-----------|-----------|------|
| `r.get(key)` | GET | 获取值 |
| `r.set(key, value)` | SET | 设置值 |
| `r.setex(key, time, value)` | SETEX | 设置带过期时间 |
| `r.setnx(key, value)` | SETNX | 不存在时设置 |
| `r.delete(*keys)` | DEL | 删除 |
| `r.exists(*keys)` | EXISTS | 检查是否存在 |
| `r.expire(key, time)` | EXPIRE | 设置过期时间 |
| `r.ttl(key)` | TTL | 获取剩余生存时间 |
| `r.hget(name, key)` | HGET | 获取哈希字段 |
| `r.hset(name, key, value)` | HSET | 设置哈希字段 |
| `r.hmset(name, mapping)` | HMSET | 批量设置哈希 |
| `r.hgetall(name)` | HGETALL | 获取所有哈希字段 |
| `r.lpush(name, *values)` | LPUSH | 从左边添加 |
| `r.rpop(name)` | RPOP | 从右边弹出 |
| `r.sadd(name, *values)` | SADD | 添加集合成员 |
| `r.smembers(name)` | SMEMBERS | 获取所有成员 |
| `r.zadd(name, mapping)` | ZADD | 添加有序集合 |
| `r.zrange(name, start, end)` | ZRANGE | 按分数范围获取 |

### 完整示例

#### 示例1: 用户会话管理
```python
import redis
import json
from datetime import datetime

class UserSession:
    def __init__(self):
        self.redis_client = redis.Redis(host='localhost', port=6379, db=0)
    
    def create_session(self, user_id, username, ip_address):
        session_key = f"session:{user_id}"
        session_data = {
            "username": username,
            "login_time": datetime.now().isoformat(),
            "ip_address": ip_address
        }
        # 存储session，过期时间1小时
        self.redis_client.setex(
            session_key, 
            3600, 
            json.dumps(session_data)
        )
        return True
    
    def get_session(self, user_id):
        session_key = f"session:{user_id}"
        data = self.redis_client.get(session_key)
        if data:
            return json.loads(data)
        return None
    
    def delete_session(self, user_id):
        session_key = f"session:{user_id}"
        return self.redis_client.delete(session_key)
    
    def extend_session(self, user_id):
        session_key = f"session:{user_id}"
        return self.redis_client.expire(session_key, 3600)

if __name__ == '__main__':
    session = UserSession()
    
    # 创建会话
    session.create_session("user123", "alice", "192.168.1.100")
    
    # 获取会话
    print(session.get_session("user123"))
    
    # 删除会话
    session.delete_session("user123")
```

#### 示例2: 分布式锁
```python
import redis
import time
import uuid
from contextlib import contextmanager

class DistributedLock:
    def __init__(self, redis_client=None):
        if redis_client is None:
            self.redis_client = redis.Redis(host='localhost', port=6379)
        else:
            self.redis_client = redis_client
    
    @contextmanager
    def lock(self, lock_name, timeout=10, blocking=True, blocking_timeout=None):
        lock_key = f"lock:{lock_name}"
        lock_value = str(uuid.uuid4())
        acquired = False
        
        try:
            acquired = self.redis_client.set(
                lock_key, 
                lock_value, 
                nx=True,  # 不存在时才设置
                ex=timeout  # 过期时间
            )
            
            if not acquired and blocking:
                start_time = time.time()
                while not acquired:
                    if blocking_timeout and (time.time() - start_time) > blocking_timeout:
                        raise TimeoutError(f"Could not acquire lock: {lock_name}")
                    time.sleep(0.1)
                    acquired = self.redis_client.set(
                        lock_key, 
                        lock_value, 
                        nx=True, 
                        ex=timeout
                    )
            elif not acquired:
                raise TimeoutError(f"Could not acquire lock: {lock_name}")
            
            yield True
            
        finally:
            # 只有锁的值匹配时才删除（防止误删其他进程的锁）
            current_value = self.redis_client.get(lock_key)
            if current_value and current_value.decode() == lock_value:
                self.redis_client.delete(lock_key)

if __name__ == '__main__':
    lock = DistributedLock()
    
    try:
        with lock.lock("order:12345", timeout=30, blocking_timeout=10):
            print("获取锁成功，执行任务...")
            time.sleep(2)
            print("任务完成")
    except TimeoutError as e:
        print(f"获取锁失败: {e}")
```

#### 示例3: 缓存装饰器
```python
import redis
import json
import functools
import hashlib
import time

def cache_redis(ttl=300, prefix="cache"):
    def decorator(func):
        r = redis.Redis(host='localhost', port=6379, db=0)
        
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            # 生成缓存键
            cache_key = f"{prefix}:{func.__name__}:{hashlib.md5(str(args).encode()).hexdigest()}"
            
            # 尝试获取缓存
            cached = r.get(cache_key)
            if cached:
                return json.loads(cached)
            
            # 执行函数
            result = func(*args, **kwargs)
            
            # 存储缓存
            r.setex(cache_key, ttl, json.dumps(result))
            
            return result
        return wrapper
    return decorator

@cache_redis(ttl=60, prefix="user")
def get_user_info(user_id):
    # 模拟数据库查询
    time.sleep(1)
    return {"user_id": user_id, "name": "Alice", "email": "alice@example.com"}

if __name__ == '__main__':
    start = time.time()
    print(get_user_info(123))  # 第一次，耗时约1秒
    print(f"耗时: {time.time() - start:.2f}秒")
    
    start = time.time()
    print(get_user_info(123))  # 第二次，从缓存读取，耗时约0秒
    print(f"耗时: {time.time() - start:.2f}秒")
```

---

## Docker 部署 Redis

### 快速启动
```bash
# 启动单个Redis容器
docker run -d \
  --name redis \
  -p 6379:6379 \
  redis:latest

# 启动带持久化的Redis
docker run -d \
  --name redis \
  -p 6379:6379 \
  -v redis_data:/data \
  redis:latest \
  redis-server --appendonly yes

# 启动带密码的Redis
docker run -d \
  --name redis \
  -p 6379:6379 \
  redis:latest \
  redis-server --requirepass your_password

# 使用Docker Compose
```

### Docker Compose 配置

#### 基本配置
```yaml
# docker-compose.yml
version: '3.8'

services:
  redis:
    image: redis:7-alpine
    container_name: redis
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
    command: redis-server --appendonly yes
    restart: unless-stopped

volumes:
  redis_data:
```

#### 生产环境配置
```yaml
version: '3.8'

services:
  redis:
    image: redis:7-alpine
    container_name: redis-prod
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data
      - ./redis.conf:/usr/local/etc/redis/redis.conf
    command: redis-server /usr/local/etc/redis/redis.conf
    environment:
      - REDIS_PASSWORD=your_secure_password
    restart: unless-stopped
    networks:
      - redis-network
    healthcheck:
      test: ["CMD", "redis-cli", "ping"]
      interval: 10s
      timeout: 5s
      retries: 3

networks:
  redis-network:
    driver: bridge
```

#### 高可用配置（主从复制）
```yaml
version: '3.8'

services:
  redis-master:
    image: redis:7-alpine
    container_name: redis-master
    ports:
      - "6379:6379"
    volumes:
      - redis_master_data:/data
    command: redis-server --appendonly yes
    restart: unless-stopped

  redis-replica:
    image: redis:7-alpine
    container_name: redis-replica
    ports:
      - "6380:6379"
    command: redis-server --replicaof redis-master 6379 --appendonly yes
    depends_on:
      - redis-master
    volumes:
      - redis_replica_data:/data
    restart: unless-stopped

volumes:
  redis_master_data:
  redis_replica_data:
```

#### Redis Cluster 配置
```yaml
version: '3.8'

services:
  redis-node-1:
    image: redis:7-alpine
    container_name: redis-node-1
    ports:
      - "7001:7001"
    volumes:
      - redis_cluster_1:/data
    command: redis-server --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes
    restart: unless-stopped

  redis-node-2:
    image: redis:7-alpine
    container_name: redis-node-2
    ports:
      - "7002:7001"
    volumes:
      - redis_cluster_2:/data
    command: redis-server --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes
    restart: unless-stopped

  redis-node-3:
    image: redis:7-alpine
    container_name: redis-node-3
    ports:
      - "7003:7001"
    volumes:
      - redis_cluster_3:/data
    command: redis-server --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes
    restart: unless-stopped

volumes:
  redis_cluster_1:
  redis_cluster_2:
  redis_cluster_3:
```

### 常用 Docker 命令
```bash
# 启动Redis
docker start redis

# 停止Redis
docker stop redis

# 重启Redis
docker restart redis

# 进入Redis CLI
docker exec -it redis redis-cli

# 带密码连接
docker exec -it redis redis-cli -a your_password

# 查看Redis容器状态
docker ps | grep redis

# 查看Redis日志
docker logs redis

# 查看Redis容器详细信息
docker inspect redis

# 连接多个Redis实例
docker exec -it redis redis-cli -p 6379

# 重置Redis容器
docker rm -f redis && docker run -d --name redis -p 6379:6379 redis:latest
```

### Redis 配置文件示例
```conf
# redis.conf

# 网络配置
bind 0.0.0.0
port 6379
protected-mode no

# 持久化配置
appendonly yes
appendfilename "appendonly.aof"
appendfsync everysec

# RDB快照配置
save 900 1
save 300 10
save 60 10000

# 内存配置
maxmemory 2gb
maxmemory-policy allkeys-lru

# 日志配置
loglevel notice
logfile ""

# 客户端配置
maxclients 10000
```

---

## Redis 日志查看

### Docker 日志
```bash
# 查看实时日志
docker logs -f redis

# 查看最近100行日志
docker logs --tail 100 redis

# 查看指定时间范围的日志
docker logs --since "2024-01-01" --until "2024-01-02" redis

# 查看最近30分钟的日志
docker logs --since 30m redis

# 查找包含特定内容的日志
docker logs redis 2>&1 | grep "error"

# 导出日志到文件
docker logs redis > redis.log 2>&1
```

### Redis 日志位置
```bash
# 容器内日志
docker exec redis cat /var/log/redis/redis-server.log

# 本地映射目录日志
cat ./redis-logs/redis.log

# 使用redis-cli查看信息
docker exec redis redis-cli INFO
docker exec redis redis-cli INFO server
docker exec redis redis-cli INFO clients
docker exec redis redis-cli INFO memory
docker exec redis redis-cli INFO stats
docker exec redis redis-cli INFO replication
docker exec redis redis-cli INFO persistence
```

### 常用监控命令
```bash
# 进入Redis CLI
docker exec -it redis redis-cli

# 查看服务器信息
INFO

# 查看所有键的数量
DBSIZE

# 查看内存使用情况
INFO memory

# 查看客户端连接
CLIENT LIST

# 查看慢查询日志
SLOWLOG GET 10

# 实时监控操作
MONITOR

# 查看持久化状态
INFO persistence

# 查看复制状态
INFO replication

# 查看CPU使用情况
INFO cpu

# 查看统计信息
INFO stats
```

### 慢查询配置
```conf
# 慢查询阈值（毫秒）
slowlog-log-slower-than 10000

# 慢查询日志最大条数
slowlog-max-len 128
```

### Redis 性能监控
```python
import redis
import time

def monitor_redis():
    r = redis.Redis(host='localhost', port=6379)
    
    print("=== Redis 监控信息 ===\n")
    
    # 服务器信息
    info = r.info()
    print(f"Redis 版本: {info['redis_version']}")
    print(f"运行时间: {info['uptime_in_days']} 天")
    print()
    
    # 内存信息
    print(f"已使用内存: {info['used_memory_human']}")
    print(f"内存峰值: {info['used_memory_peak_human']}")
    print(f"内存碎片率: {info['mem_fragmentation_ratio']}")
    print()
    
    # 连接信息
    print(f"客户端连接数: {info['connected_clients']}")
    print(f"阻塞客户端数: {info['blocked_clients']}")
    print()
    
    # 统计信息
    print(f"总键数量: {r.dbsize()}")
    print(f"每秒请求数: {info['instantaneous_ops_per_sec']}")
    print(f"总连接数: {info['total_connections_received']}")
    print(f"总命令数: {info['total_commands_processed']}")
    print()
    
    # 持久化信息
    print(f"RDB最后保存时间: {info['rdb_last_save_time']}")
    print(f"AOF最后写入: {info['aof_last_write_status']}")
    print()
    
    # 慢查询
    slow_logs = r.slowlog_get(5)
    print(f"最近 {len(slow_logs)} 条慢查询:")
    for log in slow_logs:
        print(f"  - {log['command']} ({log['duration']}μs)")

if __name__ == '__main__':
    monitor_redis()
```

### 日志级别说明
| 级别 | 说明 |
|------|------|
| debug | 详细的调试信息（仅开发环境使用） |
| verbose | 冗长的信息 |
| notice | 生产环境推荐级别 |
| warning | 警告信息 |

### 日志格式
Redis日志格式通常为：
```
<timestamp> <level> <message>
例如：
1234567890.123456 # 0 oOOoOoOoOoOoOoOoOooOoOoOoOoooOo000Oo
1234567890.124567 # 0 Server started, Redis version 7.0.0
1234567890.124567 # 0 DB loaded from disk
1234567890.124568 # 0 Ready to accept connections
```
