# Claude Code Troubleshooting

常见问题及解决方案。每次遇到新问题并解决后，更新此文件。

---

## 网络问题

### 症状：API 请求超时 / 连接失败

```bash
# 使用本地代理
export http_proxy=http://127.0.0.1:7897
export HTTP_PROXY=http://127.0.0.1:7897
export https_proxy=http://127.0.0.1:7897
export HTTPS_PROXY=http://127.0.0.1:7897
claude -p "your prompt" --dangerously-skip-permissions
```

---

## 权限问题

### 症状：反复弹出权限确认

使用 `--dangerously-skip-permissions` 跳过所有权限提示。

### 症状：认证失败

```bash
# 重新登录
claude /logout
claude  # 重新认证

# 彻底重置认证
rm -rf ~/.config/claude-code/auth.json
claude
```

---

## 性能问题

### 症状：上下文窗口满了，回答质量下降

- 使用 `/compact` 压缩上下文
- 开新会话处理新任务
- 用 `-p` 模式执行独立任务（每次干净上下文）

### 症状：CPU/内存占用高

- 大项目中将 build 目录加入 `.gitignore`
- 任务间重启 Claude Code
- 使用 `/compact` 减少上下文

---

## 搜索问题

### 症状：搜索/文件发现不工作

```bash
# 安装 ripgrep
sudo pacman -S ripgrep

# 设置环境变量
export USE_BUILTIN_RIPGREP=0
```

---

## 会话问题

### 症状：命令挂起/无响应

1. `Ctrl+C` 取消当前操作
2. 如果无响应，关闭终端重启

### 症状：想继续之前的对话

```bash
# 继续最近的对话
claude -c --dangerously-skip-permissions

# 恢复指定会话
claude -r "session-name" --dangerously-skip-permissions
```

---

## Agent Teams 问题

### 症状：团队模式不可用

```bash
# 需要启用实验性功能
export CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1
claude --dangerously-skip-permissions
```

### 症状：Teammates 之间冲突编辑同一文件

- 团队模式适合并行独立任务，不适合编辑同一文件
- 同文件编辑用单会话 + subagents

---

## 配置文件位置

| 文件 | 用途 |
|------|------|
| `~/.claude/settings.json` | 用户设置 |
| `.claude/settings.json` | 项目设置（提交到 git） |
| `.claude/settings.local.json` | 本地项目设置（不提交） |
| `~/.claude.json` | 全局状态 |
| `.mcp.json` | 项目 MCP 服务器 |
| `CLAUDE.md` | 项目指令文件 |

### 重置所有配置

```bash
rm ~/.claude.json
rm -rf ~/.claude/
rm -rf .claude/
rm .mcp.json
```

---

## Git 问题

### 症状：node_modules 被提交到 git

**原因：** 项目生成时 .gitignore 未生效或 node_modules 已被 tracked

**解决方案：**
```bash
# 从 git 中移除 node_modules（保留本地文件）
git rm -r --cached node_modules
# 确保 .gitignore 包含 node_modules
echo "node_modules" >> .gitignore
git add .gitignore
git commit --amend -m "your message"
```

**教训：** 生成项目后，第一次 commit 前务必检查 .gitignore 是否正确，用 `git status` 确认 node_modules 没被 staged。

**日期：** 2026-02-25

---

## 项目路径问题

### 症状：项目创建在错误的路径

**原因：** 未确认用户的默认开发路径就直接创建项目

**解决方案：**
- 用户默认开发路径: `/home/ming/data/Project/AIcode/`
- 项目总路径: `/home/ming/data/Project/`
- 迁移: `mv /old/path/project /home/ming/data/Project/AIcode/project`

**教训：** 创建新项目前，始终使用 `/home/ming/data/Project/AIcode/` 作为默认目录。

**日期：** 2026-02-25

---

## Claude Code 长提示问题

### 症状：长提示通过 `-p` 传入时连接超时或截断

**原因：** shell 参数长度限制 + 网络不稳定叠加

**解决方案：**
```bash
# 方法1: 将提示写入文件，用管道传入
cat prompt.md | claude -p "Follow the instructions in stdin" --dangerously-skip-permissions

# 方法2: 使用 @file 引用
claude -p "Follow instructions in @prompt.md" --dangerously-skip-permissions

# 方法3: 设置代理后重试
export https_proxy=http://127.0.0.1:7897
claude -p "your prompt" --dangerously-skip-permissions
```

**教训：** 复杂任务的提示写入文件再传入，比直接用 `-p` 传长字符串更稳定。

**日期：** 2026-02-25

---

## ngrok 后台运行问题

### 症状：ngrok 在后台启动后立即退出

**原因：** 使用 `&` 后台运行 + exec 的 shell 管理方式导致进程被回收

**解决方案：**
```bash
# 使用 nohup 保持 ngrok 运行
nohup ngrok http <port> > /dev/null 2>&1 &
sleep 3
# 通过 API 获取公开地址
curl -s http://127.0.0.1:4040/api/tunnels | python3 -c "import sys,json; d=json.load(sys.stdin); print(d['tunnels'][0]['public_url'])"
```

**教训：** 后台长运行进程用 `nohup` 而不是单纯 `&`。

**日期：** 2026-02-25

---

## 问题记录模板

遇到新问题时，按以下格式添加：

```markdown
## [问题类别]

### 症状：[描述症状]

**原因：** [根因分析]

**解决方案：**
[具体步骤]

**教训：** [避免再犯的要点]

**日期：** YYYY-MM-DD
```
