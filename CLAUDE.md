# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 这是什么仓库

个人**学习笔记仓库**,不是代码项目。全部内容是 Markdown 文档,没有构建、lint、测试、依赖或运行命令 —— 不要尝试 `npm install` / `build` / `test`,这里没有这些。

## 结构

```
learning/
├── docs/          # 所有笔记(Markdown,中文撰写)
├── README.md      # 仓库入口 + 内容索引表
└── .gitignore     # 忽略 node_modules / .env / .vscode 等
```

- `docs/claude-code.md` — Claude Code 本身(命令、权限、MCP/Hooks/Skills、settings.json 配置)
- `docs/claude-code-vscode-setup.md` — VS Code 插件接线 + DeepSeek 后端配置
- `docs/deepseek-dsh-cli.md` — DeepSeek `dsh` CLI 用法

## 约定

- **语言**:笔记正文用中文,仓库文档(README、本文件说明)也是中文。
- **新增文档时,同步更新 README.md 里的「内容索引」表格** —— README 是唯一索引,新笔记不在里面就会被漏掉。
- **每条笔记开头**用 blockquote 记录元信息,遵循现有格式:

  ```markdown
  > 记录日期:YYYY-MM-DD · 环境:Windows 11 / VS Code / Node v24
  ```

- **绝不在仓库里记录真实 token / key**。所有 API 密钥、token 一律用 `<YOUR_X_TOKEN>` 之类占位符。这是硬约束,README 与 .gitignore(忽略 `.env`、`.env.*`)都在强化它。

## 本机环境要点(影响 Claude 实际操作)

- 平台:Windows 11,通过 VS Code 扩展运行,shell 为 Git Bash。Claude Code 后端接的是 **DeepSeek Anthropic 兼容接口**(`ANTHROPIC_BASE_URL` + `ANTHROPIC_AUTH_TOKEN`,配在用户级 `~/.claude/settings.json`,不在本仓库里)。
- **网络限制**:`code.claude.com` / `docs.claude.com` 等官方域无法直接 WebFetch(被网络策略拦截)。需要查官方资料时改用 `WebSearch`,或走官方 mirror / 允许的镜像域名。
- 更新本仓库的 Claude Code / DeepSeek 相关笔记时,以这些文档现有内容为基准,不要凭空新增「官方文档没写的功能」。
