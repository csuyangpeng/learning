# DeepSeek dsh CLI 用法笔记

> 记录日期:2026-08-29 · Node v24 / npm 11.17

## 1. 是什么

`@deepseek-ai/dsh` 是 DeepSeek 官方的 CLI(DeepSeek Harness),用于 boot 各种 profile:

> "dsh: boot a DeepSeek Harness profile — an ordered stack of plugin-bundle patch layers under your own overrides."

- 来源:GitHub `deepseek-ai/deepseek-harness`
- 发布者:deepseek 官方维护者,tianyi@deepseek.com
- 提供 `profile boot`、`plugin 管理`、`web`(浏览器 UI 别名)等功能

## 2. 安装

使用 npmmirror 镜像,安装更稳:

```bash
npm install -g @deepseek-ai/dsh --registry=https://registry.npmmirror.com
```

> 首次安装会拉 60+ 依赖,耗时可能超过 1 分钟,属正常。装完全局命令在:
> `C:\Users\<user>\AppData\Roaming\npm\dsh`
>
> **注意**:当前终端可能没刷新 PATH。新开一个终端即可直接敲 `dsh`;或 `export PATH="$PATH:/c/Users/<user>/AppData/Roaming/npm"`。

## 3. 基本用法

```bash
dsh --help                          # 总帮助
dsh web                             # 启动 web profile(浏览器 UI),自动开浏览器
dsh web --no-open                   # 启动但不弹浏览器
dsh web --port 8080                 # 指定端口
dsh web --port 0                    # 让系统挑空闲端口
dsh --profile web --help            # web app 自己的参数帮助
dsh --profile headless "run the tests"   # 无头模式:完成一件事就退出
dsh --profile tui                   # tui profile
dsh plugin --profile tui add <pkg>  # 往 profile 里装插件
```

### dsh web 常用参数

| 参数 | 说明 |
|------|------|
| `--host <host>` | 绑定主机 |
| `--no-open` | 不自动在浏览器打开 |
| `--port <port>` | 监听端口,`0` 表示由系统挑一个空闲端口 |
| `--trusted-host <authority...>` | 额外允许的浏览器信任域名(可多个) |

## 4. 验证是否可用

```bash
dsh -V                 # 打印版本,如 0.1.1-rc.2
dsh web --no-open --port 0   # 启动后输出如: dsh web: http://127.0.0.1:56266
curl -s -o /dev/null -w "HTTP %{http_code}\n" http://127.0.0.1:<port>/   # 期望 200
```

## 5. 踩坑:原生模块脚本被 npm 跳过

npm 11 默认启用 `allow-scripts` 安全机制,`npm install -g` 时**跳过了这些包的 install 脚本**:

- `node-pty`(pseudo-terminal,终端要用)
- `koffi`(原生 FFI 绑定)
- `@deepseek-ai/dsh-subprocess-local`(spawn helper)
- `@google/genai`、`protobufjs`

**影响**:web UI 的表层 HTTP 服务能正常跑(实测 HTTP 200),但如果界面里**终端/子进程**功能报错,需要补装并允许脚本:

```bash
# 方式一:一次性允许
npm install -g --allow-scripts=@deepseek-ai/dsh-subprocess-local,koffi,node-pty,@google/genai,protobufjs

# 方式二:写进 npm 全局配置后重装(对全局安装生效)
npm config set allow-scripts=@deepseek-ai/dsh-subprocess-local,koffi,node-pty,@google/genai,protobufjs --location=user
npm install -g @deepseek-ai/dsh
```

> 方式一在 npm 11 某些版本会误当成"本地包安装"而失败(报 ENOENT 找不到当前目录 package.json),此时优先用**方式二**。
