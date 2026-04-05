# GitHub 平台使用详解

## 目录
1. [GitHub基础概念](#github基础概念)
2. [仓库操作](#仓库操作)
3. [Pull Request (合并请求)](#pull-request-合并请求)
4. [代码审查](#代码审查)
5. [Issue (问题追踪)](#issue-问题追踪)
6. [项目管理](#项目管理)
7. [GitHub Actions 流水线](#github-actions-流水线)
8. [保护分支与权限](#保护分支与权限)
9. [GitHub Pages](#github-pages)
10. [.github 目录详解](#github-目录详解)
11. [团队协作](#团队协作)
12. [安全相关功能](#安全相关功能)

---

## GitHub基础概念

### 核心术语
| 术语 | 说明 |
|------|------|
| Repository (Repo) | 代码仓库，存储项目和代码的地方 |
| Fork | 派生他人的仓库到自己账户 |
| Clone | 将远程仓库克隆到本地 |
| Push | 将本地提交推送到远程 |
| Pull | 从远程拉取更新 |
| Branch | 分支，用于开发新功能或修复bug |
| Merge | 合并分支 |
| Commit | 提交，一次代码修改记录 |
| Pull Request (PR) | 请求合并代码的请求 |
| Review | 代码审查 |
| Issue | 问题追踪，用于报告bug或讨论功能 |
| Wiki | 项目文档 |
| Release | 版本发布 |

---

## 仓库操作

### 创建仓库

#### 通过网页创建
1. 登录 GitHub，点击右上角 `+` 号，选择 `New repository`
2. 填写仓库信息：
   - **Owner**: 选择仓库所有者（个人或组织）
   - **Repository name**: 仓库名称（建议使用小写字母和连字符）
   - **Description**: 仓库描述（可选）
   - **Public/Private**: 公开或私有
   - **Add a README file**: 是否添加README文件
   - **Add .gitignore**: 选择.gitignore模板
   - **Choose a license**: 选择开源许可证
3. 点击 `Create repository`

#### 通过命令行创建
```bash
# 在GitHub上创建仓库后，克隆到本地
git clone https://github.com/username/repository.git

# 或初始化现有项目
git init
git remote add origin https://github.com/username/repository.git
git push -u origin main
```

### 仓库设置

#### 基本设置（Settings）
- **Repository name**: 修改仓库名称
- **Description**: 修改描述
- **URL**: 仓库访问地址
- **Visibility**: 公开/私有切换

#### 分支默认设置
- 设置默认分支（main/master）
- 设置新分支的命名规则

#### 主题和功能
- 启用/禁用 Issues
- 启用/禁用 Wikis
- 启用/禁用 Projects
- 启用/禁用 Discussions

---

## Pull Request (合并请求)

### PR 的作用
- 代码审查的载体
- 记录变更讨论
- 追踪代码变更历史
- 自动化测试和质量把控

### 创建 PR

#### 通过网页创建
1. **方式一：在分支对比页面创建**
   - 切换到要提交PR的分支
   - 点击 `Compare & pull request` 按钮
   
2. **方式二：通过 Pull Requests 页面创建**
   - 点击 `Pull Requests` 标签
   - 点击 `New pull request` 按钮
   - 选择源分支和目标分支
   - 填写PR信息

#### PR 页面填写内容
```
Title (标题)
-----------
简短描述PR的目的

Body (正文)
-----------
## 做了什么
- 功能点1
- 功能点2

## 为什么做
- 原因说明

## 如何测试
- 测试步骤

Closes #123  (关联Issue)
```

#### 通过命令行创建
```bash
# 推送分支后，创建PR
git push -u origin feature-branch

# 使用gh命令创建PR（需要安装GitHub CLI）
gh pr create --title "Add new feature" --body "## What
这是详细描述"
```

### PR 页面操作

#### 查看 PR 状态
- **Open**: PR打开，等待审查
- **Closed**: PR已关闭（可能合并或拒绝）
- **Merged**: PR已合并

#### PR 页面功能区
```
┌─────────────────────────────────────────────────────────────┐
│ [Title]                                   [Status: Open]    │
├─────────────────────────────────────────────────────────────┤
│ ✓ Checks passed │ 2 reviewers │ 1 commit │ 10 changed      │
├─────────────────────────────────────────────────────────────┤
│ [Conversation] [Commits] [Checks] [Files Changed]          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   PR描述内容区域                                             │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│ Reviewers: [+] Add reviewer                                  │
│ Labels: [+] Add labels                                       │
│ Projects: [+] Add to project                                │
│ Milestone: [+] Set milestone                                │
│ Linked issues: [+] Link an issue                            │
│                                                             │
│                                                             │
│                      [Merge pull request]                   │
│                      [Squash and merge]                     │
│                      [Rebase and merge]                     │
└─────────────────────────────────────────────────────────────┘
```

#### Merge 选项（合并方式）
| 方式 | 说明 | 适用场景 |
|------|------|----------|
| **Create a merge commit** | 保留所有提交历史 | 多个有意义的小提交 |
| **Squash and merge** | 所有提交压缩为一个 | 多个小的修复提交 |
| **Rebase and merge** | 变基后直接合并 | 保持线性历史 |

#### PR 页面按钮操作
- **Merge pull request**: 合并PR
- **Squash and merge**: 压缩合并
- **Rebase and merge**: 变基合并
- **Close pull request**: 关闭PR
- **Reopen pull request**: 重新打开PR

---

## 代码审查

### 审查流程
```
开发者提交PR → 审查者审查代码 → 提出问题/建议 → 
开发者修改 → 审查者确认 → 合并代码
```

### 添加审查者

#### 通过网页添加
1. 在PR页面，点击 `Reviewers` 区域的 `+` 按钮
2. 搜索并选择审查者
3. 可以添加多个审查者

#### 审查者权限
- **Read**: 可以查看代码
- **Triage**: 可以管理PR，但不能直接修改
- **Write**: 可以审查和合并

### 审查操作

#### 添加评论
1. 点击 `Files Changed` 标签
2. 找到要评论的文件
3. 点击行号旁边的 `+` 按钮
4. 输入评论内容
5. 点击 `Add single comment` 或 `Start a review`

#### 审查类型
| 类型 | 说明 | 颜色标识 |
|------|------|----------|
| **Comment** | 提出问题或建议，不阻止合并 | 灰色 |
| **Approve** | 审核通过，建议合并 | 绿色 |
| **Request changes** | 要求修改，阻止合并 | 红色 |

#### 提交审查
1. 完成所有代码行评论后
2. 点击 `Review Changes` 按钮
3. 选择审查类型：
   - **Comment**: 只评论
   - **Approve**: 批准合并
   - **Request changes**: 要求修改
4. 添加总体评论
5. 点击 `Submit review`

### 代码审查技巧

#### 审查者应该关注
- **功能正确性**: 代码是否实现了需求
- **代码质量**: 是否遵循代码规范
- **性能问题**: 是否有性能隐患
- **安全问题**: 是否有安全漏洞
- **测试覆盖**: 是否有足够的测试
- **可维护性**: 代码是否易于维护

#### 评论模板
```
## 问题类型

[blocking] 这是一个阻止合并的问题

[optional] 这是可选的优化建议

[nitpick] 这是小问题，不影响合并

[question] 我想确认一下这个逻辑
```

### 处理审查反馈

#### 开发者操作
1. 查看审查者提出的所有评论
2. 根据反馈修改代码
3. 在原评论下回复说明
4. 提交修改并推送
5. 通知审查者已处理

---

## Issue (问题追踪)

### 创建 Issue

#### 通过网页创建
1. 点击 `Issues` 标签
2. 点击 `New issue` 按钮
3. 填写Issue信息：
   - **Title**: 标题（简明扼要）
   - **Body**: 详细描述
   - **Labels**: 标签
   - **Projects**: 所属项目
   - **Assignees**: 负责人
   - **Milestone**: 里程碑

#### Issue 模板
```markdown
## 问题描述
清晰描述遇到的问题

## 复现步骤
1. 步骤1
2. 步骤2
3. 步骤3

## 期望行为
描述期望的结果

## 实际行为
描述实际发生的情况

## 环境信息
- OS: [操作系统]
- Browser: [浏览器]
- Version: [版本号]

## 截图/日志
如有相关截图或日志，请粘贴在此
```

### Issue 标签（Labels）
| 标签 | 颜色 | 说明 |
|------|------|------|
| bug | 红色 | Bug报告 |
| enhancement | 绿色 | 功能增强 |
| documentation | 蓝色 | 文档改进 |
| help wanted | 橙色 | 需要帮助 |
| good first issue | 紫色 | 适合新手 |
| question | 黄色 | 问题咨询 |
| duplicate | 灰色 | 重复Issue |
| invalid | 灰色 | 无效Issue |
| wontfix | 灰色 | 不会修复 |

### Issue 操作
- **Open/Close**: 打开或关闭Issue
- **Lock/Unlock**: 锁定Issue防止评论
- **Pin/Unpin**: 置顶Issue
- **Delete**: 删除Issue（仅仓库管理员）
- **Transfer**: 转移到其他仓库

### 关联 PR 和 Issue
```markdown
# 在Issue中引用PR
Closes #123
Fixes #123
Resolves #123
Close #123
```

---

## 项目管理

### GitHub Projects

#### 创建项目
1. 点击 `Projects` 标签
2. 点击 `New project`
3. 选择模板：
   - **Board**: 看板视图（类似Trello）
   - **Table**: 表格视图
   - **Roadmap**: 路线图视图

#### 看板（Board）视图
```
┌──────────────┬──────────────┬──────────────┬──────────────┐
│   To Do      │   In Progress │   In Review  │    Done     │
├──────────────┼──────────────┼──────────────┼──────────────┤
│ ┌──────────┐ │ ┌──────────┐  │ ┌──────────┐  │ ┌──────────┐ │
│ │ Issue #1 │ │ │ Issue #2 │  │ │ Issue #3 │  │ │ Issue #4 │ │
│ └──────────┘ │ └──────────┘  │ └──────────┘  │ └──────────┘ │
│ ┌──────────┐ │              │              │              │
│ │ Issue #5 │ │              │              │              │
│ └──────────┘ │              │              │              │
└──────────────┴──────────────┴──────────────┴──────────────┘
```

#### 自定义字段
- 文本字段
- 数字字段
- 日期字段
- 单选字段
- 迭代字段（Sprint）
- 关系字段

### Milestones（里程碑）

#### 创建里程碑
1. 点击 `Milestones`
2. 点击 `New milestone`
3. 填写：
   - **Title**: 标题
   - **Description**: 描述
   - **Due date**: 截止日期

#### 查看里程碑进度
- 显示关联的Issue和PR数量
- 显示完成百分比
- 显示逾期的问题

---

## GitHub Actions 流水线

### 基本概念
| 概念 | 说明 |
|------|------|
| Workflow | 工作流，一个自动化流程 |
| Job | 作业，工作流中的一个步骤 |
| Step | 步骤，作业中的具体操作 |
| Action | 动作，可复用的工作流单元 |
| Runner | 运行器，执行Job的环境 |
| Event | 触发事件 |

### 创建 Workflow

#### 目录结构
```
.github/
└── workflows/
    ├── ci.yml
    └── cd.yml
```

#### CI 工作流示例
```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.10'
    
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    
    - name: Run tests
      run: |
        pytest --cov=./tests
    
    - name: Upload coverage
      uses: codecov/codecov-action@v3
      with:
        file: ./coverage.xml

  lint:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    - name: Run lint
      run: |
        pip install flake8
        flake8 .
```

### 常用 Actions

#### 官方 Actions
| Action | 说明 |
|--------|------|
| actions/checkout | 检出代码 |
| actions/setup-python | 设置Python环境 |
| actions/setup-node | 设置Node.js环境 |
| actions/cache | 缓存依赖 |
| actions/upload-artifact | 上传构建产物 |
| actions/download-artifact | 下载构建产物 |

#### 市场 Actions
```yaml
# 使用市场Action
- uses: aws-actions/configure-aws-credentials@v2
  with:
    aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
    aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}

# 使用Docker Action
- name: Build and push Docker images
  uses: docker/build-push-action@v4
  with:
    push: true
    tags: user/app:latest
```

### 触发条件（on）

```yaml
on:
  # push时触发
  push:
    branches: [ main ]
    tags: [ 'v*' ]
    paths: [ 'src/**' ]
  
  # PR时触发
  pull_request:
    branches: [ main ]
    types: [ opened, synchronize ]
  
  # 定时触发
  schedule:
    - cron: '0 0 * * *'  # 每天午夜
  
  # 手动触发
  workflow_dispatch:
    inputs:
      environment:
        description: 'Environment to deploy'
        required: true
        default: 'staging'
  
  # 外部事件触发
  repository_dispatch:
    types: [ deploy ]
```

### 环境变量和密钥

```yaml
jobs:
  build:
    env:
      NODE_ENV: production
      API_URL: https://api.example.com
    
    steps:
    - name: Deploy
      env:
        SECRET_KEY: ${{ secrets.SECRET_KEY }}
      run: |
        deploy --key "$SECRET_KEY"
```

### 常用工作流模板

#### Docker 构建和推送
```yaml
name: Build and Push Docker Image

on:
  push:
    branches: [ main ]
    tags: [ 'v*' ]

jobs:
  build:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    
    - name: Login to Docker Hub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}
    
    - name: Build and push
      uses: docker/build-push-action@v4
      with:
        push: true
        tags: |
          user/app:latest
          user/app:${{ github.sha }}
        cache-from: type=registry,ref=user/app:buildcache
        cache-to: type=registry,ref=user/app:buildcache,mode=max
```

#### 多环境部署
```yaml
jobs:
  deploy:
    runs-on: ubuntu-latest
    environment: ${{ inputs.environment }}
    
    steps:
    - name: Deploy to ${{ inputs.environment }}
      run: |
        echo "Deploying to ${{ inputs.environment }}"
```

---

## 保护分支与权限

### 保护分支规则

#### 设置路径
1. 进入仓库 `Settings`
2. 点击 `Branches` 左侧菜单
3. 点击 `Add branch protection rule`

#### 配置选项
| 选项 | 说明 |
|------|------|
| Require pull request reviews | 要求PR审查 |
| Dismiss stale reviews | 忽略旧的审查 |
| Required approving reviewers | 要求指定数量的审查者 |
| Require status checks | 要求通过状态检查 |
| Require branches to be up to date | 要求分支是最新的 |
| Require signed commits | 要求签名提交 |
| Require linear history | 要求线性历史 |
| Include administrators | 管理员也受规则限制 |
| Do not allow bypassing | 不允许绕过规则 |

### Branch Protection Rules 示例

```yaml
# 保护 main 分支
Rule name: main
✓ Require pull request reviews before merging
  Required number of reviewers: 2
  ✓ Dismiss stale reviews
  ✓ Require review from Code Owners
✓ Require status checks to pass before merging
  ✓ continuous-integration (CI)
  ✓ codecov/patch
✓ Require branches to be up to date before merging
✓ Require signed commits
✓ Include administrators
✓ Do not allow bypassing
```

### 代码所有者（Code Owners）

#### 创建 CODEOWNERS 文件
```
# .github/CODEOWNERS

# 默认所有者
* @owner1 @owner2

# 特定文件的所有者
/docs/* @docs-team
*.py @python-team
/src/* @frontend-team
/tests/* @qa-team

# 特定分支的所有者
main @release-team
```

### 仓库权限级别

| 角色 | 权限 |
|------|------|
| **Read** | 克隆、pull、查看 |
| **Triage** | 管理Issue和PR |
| **Write** | 推送、删除分支、管理PR |
| **Maintain** | 管理仓库设置（除敏感） |
| **Admin** | 完全控制 |

---

## GitHub Pages

### 启用 GitHub Pages

1. 进入仓库 `Settings`
2. 左侧菜单选择 `Pages`
3. **Source** 部分：
   - 选择分支（main/gh-pages）
   - 选择目录（/root 或 /docs）
4. 点击 `Save`

### 主题选择
1. 在 `Pages` 设置页面
2. 点击 `Change theme`
3. 选择主题
4. 可自定义CSS

### 自定义域名

1. 在 `Pages` 设置页面
2. **Custom domain** 输入域名
3. 配置DNS记录：
   ```
   Type: A
   Name: @
   Value: 185.199.108.153, 185.199.109.153, 185.199.110.153, 185.199.111.153
   
   Type: CNAME
   Name: www
   Value: username.github.io
   ```
4. 启用 HTTPS

---

## .github 目录详解

### 目录结构
```
.github/
├── ISSUE_TEMPLATE/
│   ├── bug_report.md
│   ├── feature_request.md
│   └── config.yml
├── PULL_REQUEST_TEMPLATE.md
├── CODEOWNERS
├── workflows/
│   ├── ci.yml
│   └── cd.yml
├── FUNDING.yml
├── SECURITY.md
├── CONTRIBUTING.md
└── support.md
```

### ISSUE_TEMPLATE（Issue模板）

#### bug_report.md
```markdown
---
name: Bug report
about: 报告一个Bug帮助我们改进
title: '[Bug] '
labels: bug
assignees: ''
---

## Bug描述
清晰描述遇到的Bug

## 复现步骤
1.
2.
3.

## 期望行为

## 实际行为

## 环境信息
- OS: [e.g. macOS, Windows]
- Version: [e.g. 1.0.0]

## 截图
```

#### feature_request.md
```markdown
---
name: Feature request
about: 提出新功能建议
title: '[Feature] '
labels: enhancement
assignees: ''
---

## 功能描述

## 使用场景

## 建议的解决方案

## 其他上下文
```

#### config.yml
```yaml
blank_issues_enabled: false
contact_links:
  - name: 讨论
    url: https://github.com/orgs/community/discussions
    about: 在讨论区提问
```

### PULL_REQUEST_TEMPLATE.md
```markdown
## 描述
简短描述这个PR做了什么

## 变更类型
- [ ] Bug修复
- [ ] 新功能
- [ ] 代码重构
- [ ] 文档更新
- [ ] 其他

## 测试
- [ ] 添加了测试
- [ ] 不需要测试

## 截图（如有UI变更）

## Checklist
- [ ] 代码遵循项目的代码规范
- [ ] 自查通过
- [ ] 相关文档已更新
- [ ] 关联的Issue已关闭

## 关联Issue
Closes #123
```

### FUNDING.yml（赞助链接）
```yaml
github: # GitHub Sponsors 用户名
patreon: # Patreon 用户名
open_collective: # Open Collective 名称
ko_fi: # Ko-fi 用户名
tidelift: # Tidelift 包名
community_bridge: # Community Bridge 项目名
issuehunt: # IssueHunt 用户名
otechie: # Otechie 用户名
lfx_crowdfunding: # LFX Crowdfunding 项目名
custom: # 自定义链接
```

### SECURITY.md（安全政策）
```markdown
# 安全政策

## 支持的版本
| 版本 | 支持状态 |
|------|----------|
| 1.0.x | 支持 |
| 0.9.x | 不支持 |

## 报告安全漏洞

请通过以下方式报告安全漏洞：

**私有的方式报告**（推荐）
- 点击 `Security` 标签
- 点击 `Report a vulnerability`

**邮件方式**
- security@example.com

我们会在 24 小时内回复，并在 7 天内提供修复计划。
```

### CONTRIBUTING.md（贡献指南）
```markdown
# 贡献指南

## 开发环境

1. Fork 本仓库
2. 克隆你的Fork
3. 安装依赖
4. 创建功能分支

## 提交规范

请遵循 Conventional Commits 规范：
- `feat:` 新功能
- `fix:` Bug修复
- `docs:` 文档更新
- `style:` 代码格式
- `refactor:` 重构
- `test:` 测试
- `chore:` 构建/工具

## Pull Request 流程

1. 确保所有测试通过
2. 更新相关文档
3. 提交PR并描述变更
```

---

## 团队协作

### 组织（Organization）

#### 创建组织
1. 点击 `+` → `New organization`
2. 选择计划（免费/团队/企业）
3. 配置组织信息

#### 组织角色
| 角色 | 权限 |
|------|------|
| Owner | 完全控制 |
| Member | 默认权限 |
| Billing Manager | 管理账单 |
| Outside Collaborator | 外部协作者 |

### 团队（Team）

#### 创建团队
1. 进入组织页面
2. 点击 `Teams` → `New team`
3. 填写信息：
   - Team name
   - Description
   - Parent team（可选）
   - Privacy（公开/私有）

#### 团队权限继承
```
Organization
├── @owners (完全权限)
├── @developers
│   ├── @frontend
│   └── @backend
└── @qa
```

### Repository Permissions（仓库权限）

#### 组织级别权限
- **Public repositories**: 可见
- **Private repositories**: 仅成员可见
- **Internal repositories**: 组织内可见

### GitHub Discussions（讨论区）

#### 启用 Discussions
1. 仓库 Settings → Features → Discussions
2. 点击 `Set up your Discussions`

#### 讨论分类
- Announcements（公告）
- Q&A（问答）
- Ideas（想法）
- General（综合）

---

## 安全相关功能

### Security（安全功能）

#### Private Vulnerability Reporting（私有漏洞报告）
1. Settings → Features → Private vulnerability reporting
2. 启用后，研究者可以直接报告漏洞

#### Dependabot Alerts（依赖警报）
- 自动扫描漏洞依赖
- 生成修复建议
- 创建修复PR

#### Secret Scanning（密钥扫描）
- 检测代码中的密钥
- 通知仓库管理员
- 阻止推送包含密钥的提交

### Security Policies

#### 查看安全政策
- 仓库 `Security` 标签
- 显示安全政策链接
- 显示支持的版本

### Access Management

#### 查看仓库访问
- Settings → Access
- 显示所有有权限的用户和团队
- 显示各自的权限级别

### 双重认证（2FA）

#### 强制要求 2FA
1. Organization Settings → Authentication
2. 勾选 `Require two-factor authentication`

---

## 常用快捷键

### 全局快捷键
| 快捷键 | 功能 |
|--------|------|
| `s` | 聚焦搜索框 |
| `g` + `p` | 跳转到 Pull Requests |
| `g` + `i` | 跳转到 Issues |
| `c` | 创建新 Issue/PR |
| `?` | 显示快捷键帮助 |

### 在仓库页面
| 快捷键 | 功能 |
|--------|------|
| `t` | 文件查找器 |
| `w` | 切换分支 |
| `y` | 显示文件永久链接 |
| `i` | 显示/隐藏 Issue 侧边栏 |
| `u` | 返回 Issue 列表 |
| `j` | 下一项 |
| `k` | 上一项 |

### 在编辑器
| 快捷键 | 功能 |
|--------|------|
| `Ctrl + Enter` | 提交 |
| `Ctrl + K` | 插入链接 |
| `Ctrl + Shift + M` | 插入代码块 |
| `Tab` | 缩进 |
| `Shift + Tab` | 取消缩进 |
