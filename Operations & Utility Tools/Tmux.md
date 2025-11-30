# 终端复用器 - Tmux 完全指南

## 1. 核心概念
| 概念         | 描述                                                                 |
|--------------|----------------------------------------------------------------------|
| 会话(Session) | 工作环境的容器，支持后台持久化运行（`tmux new -s <名称>` 创建新会话） |
| 窗口(Window)  | 全屏工作空间，支持多标签页管理（`前缀键 + c` 创建新窗口）             |
| 窗格(Pane)    | 窗口的可分屏区域，支持多任务并行（`前缀键 + %` 垂直分割）             |
| 前缀键        | 命令触发器（默认 `Ctrl+b`，推荐修改为 `Ctrl+a`）                     |

## 2. 核心功能详解

### 2.1 会话管理
```bash
# 创建并命名会话
tmux new -s mysession

# 常用会话操作
tmux ls                  # 列出会话
tmux attach -t mysession # 进入会话
tmux kill-session -t 0   # 终止会话
```

### 2.2 窗口操作
| 快捷键      | 功能                     | 等效命令          |
|-------------|--------------------------|-------------------|
| 前缀键 + c  | 新建窗口                 | new-window        |
| 前缀键 + &  | 关闭当前窗口             | kill-window       |
| 前缀键 + ,  | 重命名窗口               | rename-window     |
| 前缀键 + p/n | 切换窗口                 | previous-window/next-window |

### 2.3 窗格管理
```bash
# 基础分屏
前缀键 + %   # 垂直分屏
前缀键 + "   # 水平分屏

# 进阶操作
前缀键 + z    # 最大化/还原窗格
前缀键 + 方向键 # 切换窗格
前缀键 + Ctrl+方向键 # 调整窗格尺寸
```

## 3. 高效复制模式

### 3.1 基础工作流
1. `前缀键 + [` 进入复制模式
2. 空格键 开始文本选择
3. 回车键 确认复制
4. `前缀键 + ]` 粘贴内容

### 3.2 高级技巧
```bash
# Vi模式增强配置（添加到 ~/.tmux.conf）
setw -g mode-keys vi
bind-key -T copy-mode-vi v send-keys -X begin-selection
bind-key -T copy-mode-vi y send-keys -X copy-selection-and-cancel

# 系统剪贴板集成（需安装xclip）
bind-key -T copy-mode-vi y send-keys -X copy-pipe-and-cancel "xclip -selection clipboard"
```

## 4. 配置指南

### 4.1 基础配置
```bash
# ~/.tmux.conf 示例
set -g prefix C-a        # 修改前缀键
set -g mouse on          # 启用鼠标支持
set -g base-index 1      # 窗口编号从1开始
```

### 4.2 状态栏美化
```bash
set -g status-bg colour234
set -g status-fg colour137
set -g status-left "#[bg=colour7] #S "
set -g status-right "#[fg=colour233] %Y-%m-%d %H:%M "
```

## 5. 快速参考表

| 分类       | 快捷键            | 功能描述               |
|------------|-------------------|------------------------|
| **会话**   | 前缀键 + d        | 分离会话               |
|            | 前缀键 + $        | 重命名会话             |
| **窗口**   | 前缀键 + 数字     | 跳转指定窗口           |
|            | 前缀键 + w        | 窗口列表               |
| **系统**   | 前缀键 + ?        | 查看所有快捷键         |
|            | 前缀键 + :        | 进入命令模式           |

## 6. 注意事项
1. 修改前缀键后需重新加载配置（`前缀键 + :` 输入 `source ~/.tmux.conf`）
2. 会话持久化功能需保持tmux服务运行
3. 推荐使用tmux-resurrect插件实现会话保存/恢复
```// End of Selection
```