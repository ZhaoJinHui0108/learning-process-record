# Git 常用命令详解

## 目录
1. [基础配置命令](#基础配置命令)
2. [仓库初始化与克隆](#仓库初始化与克隆)
3. [文件操作命令](#文件操作命令)
4. [提交与历史](#提交与历史)
5. [分支操作](#分支操作)
6. [远程仓库操作](#远程仓库操作)
7. [暂存与恢复](#暂存与恢复)
8. [标签管理](#标签管理)
9. [日志与差异](#日志与差异)
10. [撤销与回退](#撤销与回退)
11. [高级操作](#高级操作)

---

## 基础配置命令

### 设置用户名和邮箱
```bash
# 设置全局用户名
git config --global user.name "Your Name"

# 设置全局邮箱
git config --global user.email "your.email@example.com"

# 设置当前仓库的用户名（覆盖全局设置）
git config --local user.name "Your Name"
git config --local user.email "your.email@example.com"
```

### 查看配置
```bash
# 查看所有配置
git config --list

# 查看特定配置
git config --global user.name

# 查看配置文件位置
git config --list --show-origin
```

### 设置别名
```bash
# 为常用命令设置别名
git config --global alias.st status
git config --global alias.co checkout
git config --global alias.br branch
git config --global alias.ci commit
git config --global alias.unstage 'reset HEAD --'

# 使用别名
git st   # 等同于 git status
git co   # 等同于 git checkout
```

---

## 仓库初始化与克隆

### 初始化新仓库
```bash
# 在当前目录初始化Git仓库
git init

# 在指定目录初始化仓库
git init my-project
cd my-project
git init
```

### 克隆远程仓库
```bash
# 克隆整个仓库
git clone https://github.com/username/repository.git

# 克隆到指定目录
git clone https://github.com/username/repository.git my-folder

# 克隆指定分支
git clone -b develop https://github.com/username/repository.git

# 浅克隆（只保留最近一次提交）
git clone --depth 1 https://github.com/username/repository.git

# 克隆所有分支
git clone --bare https://github.com/username/repository.git
```

---

## 文件操作命令

### 查看文件状态
```bash
# 查看工作区状态
git status

# 简洁模式
git status -s

# 显示更多信息
git status -v
```

### 添加文件到暂存区
```bash
# 添加单个文件
git add filename.txt

# 添加所有文件
git add .

# 添加所有已跟踪文件的修改（不包括新文件）
git add -u

# 交互式添加
git add -i

# 添加所有txt文件
git add *.txt

# 添加目录下的所有文件
git add docs/

# 添加所有修改和删除，但不包含新文件
git add -A
```

### 删除文件
```bash
# 从工作区和暂存区删除文件
git rm filename.txt

# 只从暂存区删除（保留工作区文件）
git rm --cached filename.txt

# 删除已修改但未暂存的文件（强制删除）
git rm -f filename.txt

# 删除目录
git rm -r my-folder/
```

### 重命名文件
```bash
# Git会自动追踪重命名
git mv oldname.txt newname.txt

# 相当于执行了以下操作：
# mv oldname.txt newname.txt
# git rm oldname.txt
# git add newname.txt
```

---

## 提交与历史

### 提交更改
```bash
# 提交暂存区的文件
git commit -m "提交信息"

# 提交所有已跟踪文件的修改
git commit -am "提交信息"

# 修改最后一次提交（追加修改或修改提交信息）
git commit --amend

# 提交但不add（需要先把修改add到暂存区）
git commit -m "message" --allow-empty

# 空提交（用于触发CI/CD）
git commit -m "trigger build" --allow-empty
```

### 查看提交历史
```bash
# 查看完整提交历史
git log

# 查看简洁的历史
git log --oneline

# 查看最近n次提交
git log -n 5

# 查看所有分支的历史
git log --all

# 图形化显示分支
git log --graph --oneline --all

# 查看特定文件的历史
git log filename.txt

# 查看每次提交的详细统计
git log --stat

# 格式化输出
git log --pretty=format:"%h %an %s"
```

### 格式化选项
```
%H  提交哈希
%h  简短的提交哈希
%an 作者名字
%ae 作者邮箱
%ad 作者提交日期
%s  提交说明
%b  提交正文
```

---

## 分支操作

### 查看分支
```bash
# 查看本地分支
git branch

# 查看所有分支（包括远程）
git branch -a

# 查看已合并到当前分支的分支
git branch --merged

# 查看未合并到当前分支的分支
git branch --no-merged

# 查看分支及其最新提交
git branch -v

# 查看跟踪的远程分支
git branch -vv
```

### 创建分支
```bash
# 创建新分支
git branch feature-branch

# 创建并切换到新分支
git checkout -b feature-branch

# 创建基于特定提交的分支
git branch feature-branch commit-hash

# 创建与远程分支同名的本地分支
git checkout --track origin/feature-branch

# 创建并切换到新分支（现代Git）
git switch -c feature-branch
```

### 切换分支
```bash
# 切换到已有分支
git checkout feature-branch

# 切换到上一个分支
git checkout -

# 切换到指定提交
git checkout commit-hash

# 创建并切换（新语法）
git switch feature-branch
git switch -          # 切换到上一个分支

# 切换到某个tag
git checkout v1.0.0
```

### 合并分支
```bash
# 将指定分支合并到当前分支
git merge feature-branch

# 合并并创建合并提交（禁用fast-forward）
git merge --no-ff feature-branch

# 合并但停止在冲突前（用于测试）
git merge --no-commit feature-branch

# 取消合并
git merge --abort
```

### 删除分支
```bash
# 删除已合并的分支
git branch -d feature-branch

# 强制删除分支
git branch -D feature-branch

# 删除远程分支
git push origin --delete feature-branch

# 删除已合并的远程分支
git push origin --delete feature-branch
```

### 变基（Rebase）
```bash
# 将当前分支变基到目标分支
git rebase main

# 变基并继续
git rebase --continue

# 跳过当前提交
git rebase --skip

# 取消变基
git rebase --abort

# 交互式变基（修改最近n次提交）
git rebase -i HEAD~3

# 将分支A变基到分支B上
git rebase main feature-branch
```

### 拣选（Cherry-pick）
```bash
# 拣选单个提交
git cherry-pick commit-hash

# 拣选多个提交
git cherry-pick commit1 commit2

# 拣选并继续
git cherry-pick --continue

# 拣选但不停在冲突
git cherry-pick --no-commit commit-hash

# 取消拣选
git cherry-pick --abort
```

---

## 远程仓库操作

### 查看远程仓库
```bash
# 查看远程仓库
git remote

# 查看远程仓库详细信息
git remote -v

# 查看某个远程仓库的详细信息
git remote show origin
```

### 添加远程仓库
```bash
# 添加远程仓库
git remote add origin https://github.com/username/repository.git

# 添加其他远程仓库
git remote add upstream https://github.com/original-owner/repository.git

# 重命名远程仓库
git remote rename origin upstream
```

### 推送与拉取
```bash
# 推送到远程仓库
git push origin main

# 推送到远程仓库的指定分支
git push origin feature-branch

# 推送所有分支
git push --all origin

# 推送并设置上游分支
git push -u origin feature-branch

# 强制推送（谨慎使用）
git push --force origin main

# 删除远程分支后推送
git push origin --delete old-branch

# 从远程仓库拉取
git pull origin main

# 拉取并变基（如果本地有提交）
git pull --rebase origin main

# 只拉取指定分支
git pull origin feature-branch

# 禁止快进合并
git pull --no-ff
```

### 抓取（Fetch）
```bash
# 从远程仓库抓取所有分支
git fetch origin

# 抓取所有远程仓库
git fetch --all

# 抓取指定分支
git fetch origin feature-branch

# 抓取并 prune（删除远程已删除的分支引用）
git fetch --prune origin

# 抓取所有标签
git fetch --tags
```

---

## 暂存与恢复

### git stash（暂存工作区）
```bash
# 暂存当前工作区的修改
git stash

# 暂存并添加说明
git stash save "修改说明"

# 暂存包括未跟踪的文件
git stash -u

# 暂存所有文件（包括忽略的文件）
git stash -a

# 查看暂存列表
git stash list

# 应用最新的暂存（保留暂存记录）
git stash apply

# 应用最新的暂存（删除暂存记录）
git stash pop

# 应用指定暂存
git stash apply stash@{0}

# 删除暂存
git stash drop stash@{0}

# 删除所有暂存
git stash clear

# 查看暂存的内容
git stash show
git stash show -p  # 查看详细diff
```

---

## 标签管理

### 创建标签
```bash
# 创建轻量标签
git tag v1.0.0

# 创建附注标签（推荐）
git tag -a v1.0.0 -m "版本1.0.0发布"

# 为特定提交创建标签
git tag -a v0.9.0 commit-hash -m "版本0.9.0"

# 签名标签（需要GPG密钥）
git tag -s v1.0.0 -m "签名版本"
```

### 查看和操作标签
```bash
# 查看所有标签
git tag

# 查看特定模式的标签
git tag -l "v1.*"

# 查看标签详细信息
git show v1.0.0

# 推送单个标签
git push origin v1.0.0

# 推送所有标签
git push --tags

# 删除本地标签
git tag -d v1.0.0

# 删除远程标签
git push origin --delete v1.0.0

# 验证标签签名
git tag -v v1.0.0
```

---

## 日志与差异

### git log 高级用法
```bash
# 查看某个人的提交
git log --author="John"

# 查看某个时间范围的提交
git log --since="2024-01-01"
git log --until="2024-12-31"

# 查看包含特定文件的提交
git log --filename.txt

# 查看两个版本之间的差异
git log hash1..hash2

# 查看分支之间的差异
git log main..feature-branch

# 搜索提交信息
git log --grep="fix bug"

# 统计修改行数
git log --numstat

# 简洁图表
git log --oneline --graph --decorate --all
```

### git diff 差异比较
```bash
# 查看工作区的修改
git diff

# 查看暂存区的修改
git diff --cached
git diff --staged

# 查看与指定分支的差异
git diff main

# 比较两个分支的差异
git diff main feature-branch

# 比较两次提交的差异
git diff abc123..def456

# 只显示文件名
git diff --name-only

# 显示统计信息
git diff --stat

# 格式化输出
git diff --stat=40,5
```

---

## 撤销与回退

### 撤销工作区的修改
```bash
# 撤销单个文件的修改
git checkout -- filename.txt

# 撤销所有文件的修改
git checkout -- .

# 使用新语法
git restore filename.txt
git restore .
```

### 撤销暂存区的修改
```bash
# 取消暂存（将文件从暂存区移回工作区）
git reset HEAD filename.txt

# 取消所有暂存
git reset HEAD

# 新语法
git restore --staged filename.txt
git restore --staged .
```

### 回退版本
```bash
# 回退到上一次提交（soft）
git reset --soft HEAD^

# 回退到上一次提交（mixed，默认）
git reset HEAD^

# 回退到上一次提交（hard，危险！）
git reset --hard HEAD^

# 回退到指定提交
git reset --hard commit-hash

# 回退到某个版本，但保留之后的提交为未提交状态
git reset --keep commit-hash

# 查看所有操作记录（用于恢复误删的提交）
git reflog

# 根据reflog恢复
git reset --hard HEAD@{5}
```

### 还原（Revert）
```bash
# 创建一个新提交来撤销指定提交的修改
git revert commit-hash

# 还原多个提交
git revert commit1..commit2

# 不自动提交
git revert -n commit-hash
```

### 检出（Checkout）
```bash
# 恢复单个文件到指定版本
git checkout commit-hash -- filename.txt

# 恢复文件到HEAD版本
git checkout HEAD -- filename.txt

# 创建新分支并检出
git checkout -b new-branch commit-hash
```

---

## 高级操作

### 清理
```bash
# 删除未跟踪的文件
git clean -f

# 删除未跟踪的目录
git clean -fd

# 交互式删除
git clean -i

# 查看将被删除的文件（不实际删除）
git clean -n

# 删除忽略的文件
git clean -X
```

### 搜索
```bash
# 在文件中搜索
git grep "search term"

# 显示行号
git grep -n "search term"

# 统计匹配次数
git grep -c "search term"

# 搜索在特定版本中
git grep "search term" v1.0.0

# 搜索正则表达式
git grep -E "pattern"
```

### 查找文件
```bash
# 查找包含特定内容的文件
git ls-files | grep "pattern"

# 列出Git管理的文件
git ls-files

# 列出被忽略的文件
git ls-files --others --ignored
```

### 子模块
```bash
# 添加子模块
git submodule add https://github.com/username/repo.git path/to/submodule

# 初始化子模块
git submodule init

# 更新子模块
git submodule update

# 更新所有子模块
git submodule update --init --recursive

# 删除子模块
git submodule deinit path/to/submodule
```

### 工作区打包
```bash
# 创建归档
git archive -o archive.tar.gz HEAD

# 创建zip归档
git archive -o archive.zip HEAD

# 只包含特定目录
git archive -o partial.zip HEAD:docs/
```

### 维护命令
```bash
# 垃圾回收
git gc

# 检查仓库完整性
git fsck

# 压缩对象
git prune

# 查看引用
git show-ref
```

### Git Bundle（打包传输）
```bash
# 创建bundle
git bundle create repo.bundle HEAD master

# 从bundle恢复
git clone repo.bundle
```

### bisect（二分查找）
```bash
# 开始二分查找
git bisect start

# 标记当前版本为坏版本
git bisect bad

# 标记已知好版本
git bisect good v1.0.0

# Git会自动切换到中间版本，测试后标记
git bisect good  # 或 git bisect bad

# 结束bisect
git bisect reset
```

---

## 常见工作流示例

### 典型功能开发流程
```bash
# 1. 从main创建功能分支
git checkout -b feature/new-feature

# 2. 开发并提交
git add .
git commit -m "Add new feature"

# 3. 定期从main拉取最新代码
git fetch origin
git rebase origin/main

# 4. 功能完成后，合并回main
git checkout main
git merge --no-ff feature/new-feature

# 5. 删除功能分支
git branch -d feature/new-feature

# 6. 推送到远程
git push origin main
```

### 处理冲突
```bash
# 1. 拉取远程更新
git pull origin main

# 2. 如果有冲突，Git会标记冲突文件
# 编辑冲突文件，解决冲突

# 3. 标记冲突已解决
git add conflicted-file.txt

# 4. 完成合并提交
git commit -m "Resolve merge conflict"

# 或使用mergetool
git mergetool
```

### 同步fork仓库
```bash
# 1. 添加上游仓库
git remote add upstream https://github.com/original-owner/repo.git

# 2. 获取上游更新
git fetch upstream

# 3. 切换到main分支
git checkout main

# 4. 合并上游的main分支
git merge upstream/main

# 5. 推送到自己fork的仓库
git push origin main
```
