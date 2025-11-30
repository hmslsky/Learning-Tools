# Git 全流程操作指南

## 1. 仓库初始化与基础操作

### 1.1 新建仓库
```bash
git init -b <branch_name>            # 初始化指定名称的分支
git remote add origin <remote-url>   # 添加远程仓库地址
```

### 1.2 已有仓库，添加远程仓库
```bash
git remote add origin <remote-url>   # 添加远程仓库地址
git pull origin main                 # 拉取指定分支代码（推荐先pull后push）
```

### 1.3 代码提交规范
```bash
git add .                            # 添加所有文件到暂存区
git commit -m "init"                 # 提交代码（推荐明确描述修改内容）
git commit --amend                   # 修改上次提交信息
```

### 1.4 代码同步操作
```bash
git pull origin main                 # 拉取指定分支代码（推荐先pull后push）
git push -u origin main              # 首次推送设置上游分支
# 设置默认分支
git branch --set-upstream-to=origin/main main
git push --set-upstream origin main


```

## 2. 分支管理全流程
### 2.1 分支生命周期管理
```bash
# 创建与切换
git checkout -b feature/new-module   # 创建并切换到新分支
git switch main                      # 切换到已有分支

# 查看分支关系
git log --graph --oneline --all      # 图形化显示分支拓扑

# 分支同步
git fetch --all                      # 获取所有远程分支元数据
git push origin :old-branch          # 删除远程分支
```

### 2.2 临时存储技巧
```bash
git stash push -m "temp save"        # 暂存当前修改（包含未跟踪文件）
git stash list                       # 查看暂存列表
git stash pop stash@{0}              # 恢复指定暂存内容
```

## 3. 运维与配置管理
### 3.1 SSH密钥配置（Linux/Mac）
```bash
# 生成密钥（默认保存到 ~/.ssh/）
ssh-keygen -t ed25519 -C "your_email@example.com"

# 拷贝公钥到目标服务器
pbcopy < ~/.ssh/id_ed25519.pub        # Mac 复制到剪贴板
xclip -sel clip < ~/.ssh/id_ed25519.pub # Linux 复制

# 权限设置（必须）
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519

# 连接测试
ssh -T git@github.com                 # 测试 GitHub 连接
```

### 3.2 代理配置策略
```shell
# 临时代理（推荐方式）
export http_proxy=http://127.0.0.1:7890
export https_proxy=http://127.0.0.1:7890

# 持久化配置
git config --global http.proxy 'http://127.0.0.1:7890'
git config --global https.proxy 'https://127.0.0.1:7890'
# 取消代理
git config --global --unset http.proxy
git config --global --unset https.proxy

# 【重要！！！】
# http、https方式配置的代理对SSH无效，所以git@github.com方式不走代理，在中国大陆不稳定，建议使用https方式拉取推送
```

### 3.3 配置问题排查

#### 3.3.1 配置层级排查

```bash
# 配置层级检查（系统/全局/本地）
git config --list --show-scope --show-origin  # 显示配置作用域和来源文件
git config --system --list   # 查看系统级配置
git config --global --list   # 查看用户级配置
git config --local --list    # 查看仓库级配置
```

#### 3.3.2 SSH连接问题排查

```bash

# 01-查看已加载的SSH密钥列表
ssh-add -L                                      # 查看已加载的SSH密钥列表
ssh-add ~/.ssh/id_rsa                           # 加载SSH私有密钥

# 02-SSH连接深度诊断
GIT_SSH_COMMAND="ssh -vT" git ls-remote origin  # 详细调试SSH连接过程
ssh -vT git@github.com                          # 直接测试Git服务端连接

# 03-常见配置冲突检测
git check-attr -a -- *.txt                     # 检查文件属性过滤规则
git config --get-regexp '^(http|ssh)\.'        # 查看网络相关配置
git credential-manager diagnose                # 诊断凭据管理问题（Windows）

# 04-环境变量排查
env | grep -iE 'GIT|SSH|PROXY'                 # 检查相关环境变量
git config --get http.proxy                    # 查看当前仓库代理配置

# 05-缓存问题处理
git credential-cache exit                      # 清除凭据缓存
git update-index --really-refresh              # 强制刷新索引缓存
git config --unset credential.helper           # 临时禁用凭据帮助器

# 06-网络连接测试
git remote -v                                  # 验证远程仓库地址
git ls-remote                                  # 测试裸仓库连接能力
curl -v https://github.com                     # 直接测试HTTPS连接
```

## 4. 高级运维操作
### 4.1 历史记录清理
```bash
# 交互式清理提交历史
git rebase -i HEAD~5                 # 修改最近5次提交

# 彻底清理大文件（需强制推送）
git filter-repo --invert-paths --path large_file.dat
```

### 4.2 异常处理流程
```bash
# 撤销本地修改
git restore --staged <file>          # 取消暂存
git restore <file>                   # 丢弃工作区修改

# 回滚远程提交
git revert <commit-hash>             # 安全回滚方式
git push -f origin main              # 强制推送（慎用）
```

## 5. 常见问题解决方案
### 5.1 SSH连接问题
**现象**：`Permission denied (publickey)`  
✅ 解决方案：
1. 检查`~/.ssh/`目录权限是否为700
2. 确认公钥已添加到Git服务端
3. 执行`ssh-add ~/.ssh/id_ed25519`加载密钥

### 5.2 代理冲突问题
**现象**：克隆/推送时连接超时  
✅ 解决方案：
1. 临时关闭代理：`unset http_proxy https_proxy`
2. 检查防火墙设置
3. 测试不同网络环境

### 5.3 大文件提交问题
**现象**：`remote: error: File exceeds maximum allowed size`  
✅ 解决方案：
1. 使用`git filter-repo`清理历史记录
2. 配置.gitignore文件
3. 使用Git LFS管理大文件

### 5.4 分支污染问题
**现象**：合并后提交历史混乱  
✅ 解决方案：
1. 使用`git rebase`代替merge
2. 执行交互式变基：`git rebase -i`
3. 使用`git cherry-pick`选择性合并提交

## 6. GitLab服务管理
```bash
# 服务状态管理
gitlab-ctl start        # 启动服务
gitlab-ctl tail         # 查看实时日志
gitlab-ctl reconfigure  # 重载配置
```
