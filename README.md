# learning

个人学习笔记仓库,存放学习相关的东西、踩坑记录和环境配置笔记。

## 结构

```
learning/
├── docs/          # 各类笔记(markdown / word)
│   ├── claude-code.md                     # Claude Code 系统性笔记(官方文档整理)
│   ├── claude-code-vscode-setup.md        # Claude Code 插件 + VS Code 设置
│   ├── deepseek-dsh-cli.md                # DeepSeek dsh CLI 与 web UI 用法
│   ├── 融合方案执行计划.md                    # 基站/核心网管融合方案执行计划(方案2)
│   ├── 融合方案执行计划.docx                # 上述计划的 Word 版(全量,含原始需求与 mermaid 源码)
│   └── 基站网管与核心网管(5GC)融合方案执行计划(方案2).docx  # 清理后成品版 Word(图已渲染)
└── README.md
```

## 内容索引

| 文档 | 内容 |
|------|------|
| [docs/claude-code.md](docs/claude-code.md) | Claude Code 系统性笔记:概述与特性、安装认证、Slash 命令、CLI 参数、权限安全、MCP/Hooks/Skills、settings.json 配置引用、踩坑 |
| [docs/claude-code-vscode-setup.md](docs/claude-code-vscode-setup.md) | Claude Code 插件的"两个 Claude Code"问题、`.claude/settings.json` 与 VS Code `settings.json` 的边界、DeepSeek 环境变量配置 |
| [docs/deepseek-dsh-cli.md](docs/deepseek-dsh-cli.md) | `@deepseek-ai/dsh` CLI 的安装、`dsh web` 启动浏览器 UI、常用子命令 |
| [docs/融合方案执行计划.md](docs/融合方案执行计划.md) | 基站/核心网管融合方案执行计划(方案2):方案概述、页面分工、分阶段计划、材料清单、风险与行动项(含 mermaid 架构图源码) |
| [docs/融合方案执行计划.docx](docs/融合方案执行计划.docx) | 上述计划的 Word 版(全量,含原始需求与 mermaid 源码) |
| [docs/基站网管与核心网管(5GC)融合方案执行计划(方案2).docx](docs/基站网管与核心网管(5GC)融合方案执行计划(方案2).docx) | 融合方案执行计划的成品版 Word(图已渲染,内容已清理) |

## 说明

- 创建日期:2026-08-29
- 环境:Windows 11 / VS Code / Node v24
- 涉及敏感信息的文档中,**API Token 一律不记录真实值**,以占位符代替。
