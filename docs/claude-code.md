# Claude Code 系统性笔记

> 记录日期:2026-08-29 · 环境:Windows 11 / VS Code / Node v24
>
> 基于官方文档整理:https://code.claude.com/docs/en/overview 。与 [claude-code-vscode-setup.md](claude-code-vscode-setup.md)(VS Code 插件接线、DeepSeek 后端)互补,本笔记讲 Claude Code **本身**。

## 1. 是什么

Claude Code 是 Anthropic 出的 **agentic coding 工具**:读取你的代码库、编辑文件、运行命令,并接入你现有的开发工具。它在终端里以**会话(repl session)**形式工作,你输入自然语言,Claude 自主调用工具做出改代码、执行命令等动作。

### 1.1 四种形态(核心都是同一个 CLI)

| 形态 | 说明 |
|------|------|
| Terminal CLI | 完整功能,终端内 `claude` 启动 |
| IDE 扩展 | VS Code、JetBrains;能看到你打开的文件、光标、选中内容 |
| 桌面 App | 桌面客户端形态 |
| Web | 浏览器里使用 |

> **注意**:IDE 扩展底层跑的就是同一个 CLI,所以 `.claude/settings.json` 里的配置对扩展**同样生效**。详见 vscode-setup 笔记。

### 1.2 核心特性

- **会话式 REPL**:`claude` 在项目目录启动,跨轮保持上下文。
- **内置工具系统**:Claude 自主使用 `Bash`、`Read`/`Write`/`Edit`、`Grep`、`Glob`、`WebFetch`、`WebSearch`、`NotebookEdit` 等。
- **Slash 命令**:会话内输 `/` 访问 `/help`、`/model`、`/init`、`/mcp`、`/hooks`、`/skills`、`/compact`、`/clear`、`/exit` 等。
- **MCP**:通过 MCP server 接入外部工具/数据源(Google Drive、Jira、Slack、自定义 API)。
- **Hooks**:在工具事件上自动跑的 shell 命令(如每次编辑后格式化、退出前跑测试)。
- **Skills**:以 Markdown 形式放在 `.claude/skills/` 里的按需能力,用 `/skill-name` 调用。
- **Subagents / Agent 团队**:派生出隔离的 Claude 实例并行干活。
- **Git 工作流**:直接暂存、写 commit、建分支、开 PR。
- **项目记忆 `CLAUDE.md`**:项目根目录的 Markdown,记录编码规范/架构约定,会话开始被读取。
- **非交互(print)模式**:`claude -p "..."` 单次执行,可 `--output-format json` 输出 JSON 供脚本用。
- **权限控制**:多种 permission mode 与 allow/deny 规则。

## 2. 安装与认证

### 2.1 原生安装(推荐,macOS / Linux / WSL)

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

Windows PowerShell:

```powershell
irm https://claude.ai/install.ps1 | iex
```

Windows CMD:

```cmd
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

### 2.2 其他方式

- **Homebrew**(macOS/Linux)、**WinGet**(Windows)
- **npm**(需 Node.js 18+):`npm install -g @anthropic-ai/claude-code`
- 包管理器:`apt` / `dnf` / `apk`(Debian、Fedora、RHEL、Alpine)

> **注意**:原生安装会**后台自动更新**。原生 Windows 上建议再装 **Git for Windows**,这样 Claude Code 才能用 `Bash` 工具。

### 2.3 认证与验证

```bash
claude auth login         # 交互式登录
claude --version          # 校验版本
claude doctor             # 更详细的健康检查
```

> 认证方式**四选一**:Claude Pro/Max 订阅、Anthropic Console(API 计费)、企业平台(AWS Bedrock、Vertex AI、Microsoft Foundry)。本仓库用的是 DeepSeek 兼容接口,配置见 vscode-setup 笔记。

## 3. 快速开始

```bash
cd your-project
claude
```

示例 prompt:「explain what this codebase does」「add a README section about installation」。

> 建议先跑 `/init` 生成 `CLAUDE.md`,把项目规范和记忆固化下来,每个会话开头都会被读取。

## 4. 核心概念

### 4.1 会话(session)

`claude` 启动一次就是一个 session。上下文跨轮保持;`/compact` 压缩历史、`/clear` 清空释放上下文。

### 4.2 内置工具

| 工具 | 用途 |
|------|------|
| `Bash` | 跑 shell 命令 |
| `Read` / `Write` / `Edit` | 读写/精确替换文件 |
| `Grep` | 内容检索 |
| `Glob` | 按模式匹配文件 |
| `WebFetch` / `WebSearch` | 抓网页 / 检索 |
| `NotebookEdit` | 编辑 Jupyter notebook |

### 4.3 权限模型(概览)

权限决定「哪些工具要问授权」。见第 7、10 节。核心是 allow/deny/ask 三态规则 + 全局 permission mode。

## 5. Slash 命令

在会话输入框输 `/` 激活,`/help` 列出全部。内置常用:

| 命令 | 说明 |
|------|------|
| `/init` | 生成 `CLAUDE.md`(及可选 skills/hooks) |
| `/memory` | 编辑记忆文件(全局/项目/本地) |
| `/config` | 打开设置面板 |
| `/hooks` | 查看钩子配置 |
| `/mcp` | 管理 MCP server(启用/禁用/重连) |
| `/permissions` | 管理工具的 allow/deny 规则 |
| `/plan` | 启用 plan 模式或查看/描述会话计划 |
| `/model` | 切当前会话模型 |
| `/commit` | 用 AI 生成的 message 提交 git |
| `/review` | 审查 PR |
| `/skills` | 列可用 skills |
| `/compact` | 压缩对话历史 |
| `/clear` | 清空对话、释放上下文 |
| `/login` / `/logout` | 登录/登出 |
| `/help` / `/exit` | 帮助 / 退出 |

**自定义 Slash 命令**:在 `.claude/commands/<name>.md` 里写可复用 prompt,用 `/<name>` 调用。

> **注意**:Slash 命令与 Agent Skills 是同一概念。Agent Skills 放在 `.claude/skills/<name>/SKILL.md`(带 YAML frontmatter),用 `/skill-name` 调用 —— 见第 9 节。

## 6. CLI 参数与脚本化

`claude` 可带参数,常用:

| 参数 | 说明 |
|------|------|
| `-p, --print` | 非交互(单次任务),适合脚本/自动化 |
| `-c` | 续接最近一次会话(常与 `-p` 连用) |
| `--output-format json` | 配合 `-p` 输出 JSON,CI/CD 友好 |
| `--permission-mode <mode>` | `default`/`plan`/`acceptEdits`/`bypassPermissions` |
| `--dangerously-skip-permissions` | 跳过全部授权,仅锁定的 CI 沙箱使用 |
| `--add-dir <path>` | 追加工作目录(monorepo 常用) |
| `--model <name>` | 指定模型(如 `sonnet`) |
| `--settings` | 指定 settings 文件 |

```bash
claude -p "explain this function"                 # 单次非交互
cat logs.txt | claude -p "summarize"              # 管道输入
claude -c -p "check types"                        # 续会话处理
claude -p "..." --output-format json              # JSON 输出,配脚本
claude --add-dir ../shared                        # monorepo 追加目录
claude --model sonnet                             # 选模型
```

> **注意**:配置**优先级**从高到低为:CLI 参数 > 环境变量 > 项目设置 > 全局设置 > 内置默认。CLI 参数永远赢。

## 7. 权限与安全

### 7.1 权限模式(permission mode)

| 模式 | 行为 |
|------|------|
| `default` | 所有编辑/执行先征询 |
| `plan` | 只读,不执行工具、不写文件 |
| `acceptEdits` | 自动接受文件改动,Bash 仍征询 |
| `bypassPermissions` | 自动接受一切(低安全,仅 CI) |

### 7.2 allow / deny / ask 规则

`permissions` 配置三个数组:允许、拒绝、征询。

- `[]` — 全部征询(最安全)
- `["Read", "Bash"]` — 自动放行这些工具
- `["*"]` — 跳过所有授权(危险)

规则可按**路径**或**命令**限定:`Read(./.env)`、`Bash(npm run test *)`。项目级 allow 规则需先接受 workspace 信任对话框才生效。

> **注意**:deny 规则**永远赢** —— 即使更低优先级存在 allow 规则,也以 deny 为准。对 `.env`、`secrets/`、`infra/prod/` 等敏感路径建议设 deny。

## 8. MCP 集成

MCP(Model Context Protocol)把外部工具/数据源接进 Claude Code。

### 8.1 会话内管理

```bash
/mcp                          # 打开管理面板
/mcp enable <server-name>     # 启用
/mcp disable <server-name>    # 禁用
/mcp reconnect <server-name>  # 重连
```

> **注意**:`/mcp` 改动**只在当前会话生效**;永久添加用 `claude mcp add` 或直接改 MCP 配置文件。

### 8.2 CLI 命令

```bash
claude mcp add <name> <command>
claude mcp list
claude mcp remove <name>
claude mcp run <name>
```

### 8.3 配置结构

`.mcp.json` 或 settings.json 里的 `mcpServers` 块,用 `command`、`args`、`env` 定义;环境变量可用 `${VAR}` 引用(如 `${GITHUB_TOKEN}`)。

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["@modelcontextprotocol/server-github"],
      "env": { "GITHUB_TOKEN": "<YOUR_GITHUB_TOKEN>" }
    }
  }
}
```

常用服务器:GitHub(PR/issue)、Filesystem、Slack、Google Docs、Context7(内建文档库)。

## 9. Hooks / Skills / Subagents

### 9.1 Hooks(钩子)

发生在工具事件上的**确定性 shell 命令**。支持的事件:`SessionStart`、`UserPromptSubmit`、`PreToolUse`、`PostToolUse`、`Notification`、`Stop`、`SubagentStop`、`PreCompact` 等。

- `PreToolUse` / `PostToolUse` 支持匹配器:`"Bash"` 只匹配 Bash,`"Edit|Write"` 匹配任一,`"*"` 匹配全部。
- 配置位置(作用域由写在哪决定):`~/.claude/settings.json`(全部项目)、`.claude/settings.json`(单项目,可提交)、`.claude/settings.local.json`(单项目,gitignore)、插件 `hooks/hooks.json`。
- 脚本用退出码通信:`0`=成功/放行,`2`=致命拦截,其他=非阻塞错误;也可返回 JSON 做权限决策(`"decision": "block" | "approve"`)。

常见用例:保存后自动格式化、拦截敏感路径改动、记录工具调用审计日志。

### 9.2 Skills

放在 `.claude/skills/<name>/SKILL.md` 的按需能力,顶部带 YAML frontmatter(含 `description`),用 `/skill-name` 调用。适合打包可复用的工作流/方法论。

### 9.3 Subagents

派生隔离的 Claude 实例并行处理多线任务,常用于「多个区域并行调研/修改」。可配置自己的定义在 `.claude/agents/<name>.md`。

## 10. settings.json 配置参考

设置文件按作用域分层:

| 作用域 | 路径 | 说明 |
|--------|------|------|
| 全局(用户) | `~/.claude/settings.json` | 对所有项目生效 |
| 项目 | `.claude/settings.json` | 可提交进仓库,团队共享 |
| 本地 | `.claude/settings.local.json` | gitignore,个人覆盖 |
| 托管策略 | 组织管理 | 优先级最高,不可被覆盖 |

**优先级从高到低**:托管策略 > CLI 参数 > 本地 > 项目 > 用户。权限规则**合并**而非覆盖,deny 永远优先。

### 10.1 常用字段

```json
{
  "env": {
    "ANTHROPIC_API_KEY": "<YOUR_API_KEY>",
    "MCP_TIMEOUT": "60000"
  },
  "model": "deepseek-v4-flash-vision-exp",
  "modelPicker": {
    "options": [
      { "model": "deepseek-v4-flash", "label": "DeepSeek V4 Flash" },
      { "model": "deepseek-v4-flash-vision-exp", "label": "DeepSeek V4 Flash Vision Exp", "description": "Vision-capable experimental" }
    ],
    "replaceBuiltInOptions": true
  },
  "permissions": {
    "allow": ["Read", "Bash(npm run test *)"],
    "deny": ["Read(./.env)", "Read(./secrets/)"],
    "ask": ["Edit"]
  },
  "hooks": { "PostToolUse": [] },
  "enableAllProjectMcpServers": false
}
```

- `env` — 设会话环境变量,避免包 wrapper 脚本。token 等凭据建议放 `settings.local.json`,别进共享/提交的文件。
- `model` / `modelPicker` — 默认模型与下拉选项(本仓库接 DeepSeek 时用)。
- `permissions` — allow/deny/ask,见第 7 节。
- `hooks` — 钩子配置,见第 9 节。
- MCP 配置组:`enableAllProjectMcpServers` 一键放行项目 `.mcp.json` 的所有 server;`enabledMcpjsonServers`(白名单)、`disabledMcpjsonServers`(黑名单)、`deniedMcpServers`(企业级拒绝)。

### 10.2 与 VS Code 扩展的边界

- 模型、API 地址/token、权限、hooks → 放 `.claude/settings.json`(CLI 读)。
- `claudeCode.*` 这类界面开关(model 面板位置、autosave、focusView)→ 放 VS Code 的 `settings.json`,放 `.claude` 会被忽略。
- **不需要两处重复配置**。一条边界详解见 [claude-code-vscode-setup.md](claude-code-vscode-setup.md)。

> ⚠️ 真实 API token 别提交进任何仓库,也别贴到公开场合,一律用 `<YOUR_X_TOKEN>` 占位符。

## 11. CLAUDE.md 最佳实践

`CLAUDE.md` 是会话开始时 Claude 读取的 Markdown,提供持久指令(编码规范、架构、工作流)。

### 11.1 位置

- `~/.claude/CLAUDE.md` — 对所有项目全局生效
- `.claude/CLAUDE.md` — 项目级指令

### 11.2 与 settings.json 的分工

- `settings.json` = **硬约束**(权限、env、MCP、hooks),强制的技术规则。
- `CLAUDE.md` = **软指引**,行为指导(技术栈标识、工作流期望、约定),而非死板的风格规则。
- 建议 **≤300 行**,避免指令被稀释。

> **影响**:规则越精确、越贴近当前项目,Claude 越少跑偏。

## 12. 踩坑与常见问题

**① 接非 Anthropic 后端(如 DeepSeek)** — 用 `ANTHROPIC_BASE_URL` + `ANTHROPIC_AUTH_TOKEN` 绕到 Anthropic 兼容接口,配在 `.claude/settings.json` 的 `env`。完整示例见 vscode-setup 笔记。

**② `code.claude.com` / `docs.claude.com` 直接抓取被网络策略拦截** — 本机无法 WebFetch 官方域。临时替代:用 WebSearch 检索官方文档摘要、官方 mirror、权威第三方整理(如 Cranot/claude-code-guide 等),或通过允许的镜像域名访问。

**③ 出现两个 "Claude Code" 标签页** — 不是装了两个插件,是同一扩展注册的多个视图容器。把 `claudeCode.preferredLocation` 设 `"sidebar"` 即可根治,详见 vscode-setup 笔记。

**④ 扩展不生效** — 检查配置是否放对作用域:CLI/后端相关放 `.claude/settings.json`,界面开关放 VS Code `settings.json`。

> 文档版本会随官方持续更新;本笔记基于 2026-08-29 的官方 overview 及相关参考页整理,建议以官网为准。
