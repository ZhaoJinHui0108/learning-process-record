# learning-process-record

学习过程记录项目，包含各个技术主题的详细学习笔记。

## 项目结构

```
learning-process-record/
├── README.md                          # 项目说明文档
├── github-learning/                   # Git 和 GitHub 学习
│   ├── git-commands/                  # Git 常用命令
│   │   └── README.md
│   └── github-platform/              # GitHub 平台使用详解
│       └── README.md
├── redis/                            # Redis 数据库
│   └── README.md
├── kafka/                            # Kafka 消息队列
│   └── README.md
├── mq/                               # 消息队列（RabbitMQ）
│   └── README.md
├── nginx/                            # Nginx 服务器
│   └── README.md
├── gateway/                          # API 网关（Kong）
│   └── README.md
├── python/                           # Python 语言
│   ├── README.md
│   ├── async-examples/             # 异步编程示例
│   └── concurrent-examples/        # 并发编程示例
├── pytorch/                          # PyTorch 深度学习
│   └── README.md
├── typescript/                       # TypeScript 语言
│   └── README.md
├── docker/                           # Docker 容器化
│   └── README.md
├── sql/                              # SQL 数据库
│   └── README.md
└── ai/                              # AI 相关内容
    ├── transformer/                # Transformer 模型
    │   └── README.md
    ├── neural-networks/            # 神经网络基础
    │   └── README.md
    ├── rag/                        # RAG 检索增强生成
    │   └── README.md
    ├── langchain/                  # LangChain 框架
    │   └── README.md
    ├── prompt/                     # Prompt 工程
    │   └── README.md
    └── llm/                        # 大语言模型
        └── README.md
```

## 目录说明

### 1. Git 和 GitHub (github-learning)

- **git-commands/README.md**: Git 常用命令详解
  - 基础配置命令
  - 仓库操作
  - 分支管理
  - 远程仓库操作
  - 撤销和回退
  - 高级操作

- **github-platform/README.md**: GitHub 平台使用
  - Pull Request 创建和审查
  - 代码审查流程
  - Issue 和项目管理
  - GitHub Actions 流水线
  - 保护分支和权限
  - .github 目录配置

### 2. Redis

- **README.md**: Redis 完整学习笔记
  - Redis 简介和使用场景
  - 5种数据结构详解（String, Hash, List, Set, Sorted Set）
  - Python 调用示例
  - Docker 部署配置
  - 日志查看

### 3. Kafka

- **README.md**: Kafka 消息队列学习
  - Kafka 基本概念和架构
  - Topic, Producer, Consumer 核心概念
  - Python 生产者和消费者示例
  - Docker Compose 集群配置
  - 日志和监控

### 4. MQ (消息队列)

- **README.md**: RabbitMQ 消息队列
  - MQ 基本概念和模式
  - Exchange 类型（Direct, Fanout, Topic, Headers）
  - Python 调用示例
  - Docker 部署配置
  - 死信队列和消息确认

### 5. Nginx

- **README.md**: Nginx Web 服务器
  - 反向代理和负载均衡
  - SSL/TLS 配置
  - 静态资源服务
  - 限流和重写规则
  - Docker 部署
  - 日志分析

### 6. API 网关

- **README.md**: Kong API 网关
  - Service, Route, Upstream 概念
  - 认证和限流插件
  - Docker 部署（数据库模式和 DB-less 模式）
  - Admin API 和日志查看

### 7. Python

- **README.md**: Python 热门库和特性
  - 异步编程（asyncio, async/await）
  - 高并发（threading, multiprocessing）
  - 完整可运行的代码示例

- **async-examples/**: 异步编程示例
- **concurrent-examples/**: 并发编程示例

### 8. PyTorch

- **README.md**: PyTorch 深度学习
  - 神经网络基础组件
  - 线性回归、CNN、RNN 实例
  - 自编码器和迁移学习
  - 完整可运行的代码示例

### 9. TypeScript

- **README.md**: TypeScript 学习
  - 基础语法和类型系统
  - 泛型和高级类型
  - 面向对象特性
  - 项目结构和规范
  - 特殊配置文件说明

### 10. Docker

- **README.md**: Docker 容器化
  - 基本命令和概念
  - Dockerfile 编写
  - Docker Compose 使用
  - 镜像和容器结构
  - 日志查看

### 11. SQL

- **README.md**: SQL 数据库
  - 基础语法（SELECT, INSERT, UPDATE, DELETE）
  - JOIN 和子查询
  - 索引创建和使用
  - SQL 优化技巧
  - 性能分析和慢查询

### 12. AI

包含人工智能相关的完整学习内容：

- **transformer/README.md**: Transformer 模型详解
  - 自注意力机制
  - 位置编码
  - 完整可运行的翻译模型实例

- **neural-networks/README.md**: 神经网络基础
  - 简单神经网络实例
  - CNN 图像分类
  - RNN 序列预测
  - GAN 生成对抗网络

- **rag/README.md**: RAG 检索增强生成
  - RAG 架构和原理
  - 向量数据库和嵌入
  - 混合检索和重排序

- **langchain/README.md**: LangChain 框架
  - LLM 调用封装
  - Prompt 模板
  - Chains 和 Agents
  - Memory 记忆系统

- **prompt/README.md**: Prompt 工程
  - 基础技巧
  - Few-shot 和 CoT
  - 各种 Prompt 模板

- **llm/README.md**: 大语言模型
  - 主流 LLM 介绍
  - API 调用示例
  - 应用场景

## 使用说明

1. 每个目录下的 `README.md` 文件包含该主题的完整学习笔记
2. 涉及代码的示例都包含 `if __name__ == '__main__':` 入口
3. Docker 相关内容提供完整的 docker-compose.yml 配置
4. AI 相关内容包含可运行的 PyTorch/Transformer 实例

## 学习路径建议

### 基础阶段
1. Git 和 GitHub → 版本控制基础
2. Docker → 容器化基础
3. SQL → 数据库基础
4. Python → 编程基础

### 中级阶段
1. Redis → 缓存和会话
2. MQ → 消息队列
3. Nginx → Web 服务器
4. TypeScript → 类型化 JavaScript

### 高级阶段
1. Kafka → 大数据消息队列
2. API 网关 → 微服务架构
3. PyTorch → 深度学习基础

### AI 阶段
1. neural-networks → 神经网络基础
2. transformer → Transformer 架构
3. llm → 大语言模型
4. langchain/rag/prompt → LLM 应用开发

## 贡献

这是一个个人学习记录项目，欢迎提出改进建议。

## License

MIT License
