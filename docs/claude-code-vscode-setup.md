# Claude Code 插件 + VS Code 设置笔记

> 记录日期:2026-08-29 · 环境:Windows 11 / VS Code

## 1. 搞清楚几个设置文件:别混用

Claude Code 有两套**独立**的设置,作用域完全不同:

### `C:\Users\<user>\.claude\settings.json` — Claude Code CLI 设置

由 **Claude Code CLI 进程**读取。VS Code 扩展底层跑的就是这个 CLI,所以这里的配置**对扩展同样生效**。

管:
- `env`(环境变量:`ANTHROPIC_BASE_URL` API 地址、`ANTHROPIC_AUTH_TOKEN` token)
- `model`、`modelPicker`(模型与下拉选项)
- `permissions`(权限)、`hooks` 等

### `C:\Users\<user>\AppData\Roaming\Code\User\settings.json` — VS Code 扩展设置

由 **VS Code 扩展本身**读取,控制**界面表现**。key 统一带 `claudeCode.` 前缀:

- `claudeCode.preferredLocation`(`panel` / `sidebar`,Claude 开在哪里)
- `claudeCode.useTerminal`、`claudeCode.autosave`、`claudeCode.focusView` ……

### 边界结论

- 模型、API 地址/token → **只能放 `.claude/settings.json`**(CLI 读)。扩展官方明确建议"优先在 Claude 的 settings.json 里配环境变量"。
- `claudeCode.*` 这类界面开关 → **只能放 VS Code 的 `settings.json`**,放 `.claude/settings.json` 会被忽略。
- **不需要两处重复配置**。DeepSeek 的 env/model 在 `.claude/settings.json` 里配好,扩展就会跟着生效。

## 2. "两个 Claude Code 插件"的误解(重要踩坑)

**现象**:VS Code 里出现两个叫 "Claude Code" 的标签页/图标,容易误以为装了两个插件。

**真相**:`~/.vscode/extensions` 里其实**只有一个**扩展 `anthropic.claude-code`。重复来自这**同一个扩展注册了多个都叫 "Claude Code" 的视图容器**:

- 聊天面板(`claudeVSCodeSidebar`)— 显示 "Auto mode is enabled" 等
- 会话列表面板(`claudeVSCodeSessionsList`)— 显示 "New session / No sessions yet"
- 次级侧边栏(`claude-sidebar-secondary`)— 同一扩展的另一形态

这些由 `package.json` 的 `contributes.viewsContainers` / `views` 注册,通过上下文开关控制显示。

### 解决办法

**① 立即**:把多余那个 "Claude Code" 标签页的 **×** 关掉;活动栏里多余图标右键 → 移除。它只是视图,不影响功能。

**② 根治(推荐)**:把 VS Code 设置里的 `claudeCode.preferredLocation` 改为 **`sidebar`**:

```json
"claudeCode.preferredLocation": "sidebar"
```

这样 Claude 固定显示在右侧边栏,不再额外开一个 "Claude Code" 标签页。

> 注意:`preferredLocation` 的**默认值就是 `panel`**。就算把这行注释掉,效果仍是"每次开新标签页",重复照样出现。**必须写成生效的 `"sidebar"`**。

改完后 `Ctrl+Shift+P` → `Developer: Reload Window` 重载生效,再手动关掉残留标签页。

## 3. DeepSeek 后端配置(示例,不含真实 token)

`.claude/settings.json` 用于把 Claude Code 接到 DeepSeek 的 Anthropic 兼容接口:

```json
{
  "env": {
    "ANTHROPIC_BASE_URL": "https://api.deepseek.com/anthropic",
    "ANTHROPIC_AUTH_TOKEN": "<YOUR_DEEPSEEK_API_KEY>"
  },
  "model": "deepseek-v4-flash-vision-exp",
  "modelPicker": {
    "options": [
      { "model": "deepseek-v4-flash", "label": "DeepSeek V4 Flash" },
      { "model": "deepseek-v4-pro",   "label": "DeepSeek V4 Pro" },
      { "model": "deepseek-v4-flash-vision-exp", "label": "DeepSeek V4 Flash Vision Exp", "description": "Vision-capable experimental" }
    ],
    "replaceBuiltInOptions": true
  }
}
```

> ⚠️ 真实 token 别提交进任何仓库,也别贴到公开场合。占位符 `<YOUR_DEEPSEEK_API_KEY>` 即为安全写法。
