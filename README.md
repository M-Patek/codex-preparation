# Codex 安装与配置指南

> 适用对象：从零安装 Codex，并完成基础配置、MCP、Skill、Plugin 等扩展配置的人。  
> 文档范围：**介绍安装与配置**；不介绍具体如何用 Codex 写代码、改代码或做项目开发。  

## 目录

- [1. 文档目标](#1-文档目标)
- [2. 安装 Codex](#2-安装-codex)
- [3. 认证配置](#3-认证配置)
- [4. 配置文件位置](#4-配置文件位置)
- [5. 基础配置项](#5-基础配置项)
- [6. 项目级配置](#6-项目级配置)
- [7. MCP 简介与配置入口](#7-mcp-简介与配置入口)
- [8. Skill 简介与配置入口](#8-skill-简介与配置入口)
- [9. Plugin 简介与配置入口](#9-plugin-简介与配置入口)
- [10. 配置校验](#10-配置校验)
- [11. 命令速查](#11-命令速查)
- [12. 参考资料](#12-参考资料)

## 1. 文档目标

这份指南解决安装和配置问题：

1. 如何从零安装 Codex；
2. 如何完成登录或 API Key 认证；
3. Codex 的配置文件在哪里；
4. 常见基础配置项怎么写；
5. MCP、Skill、Plugin 是什么，以及如何让已安装的 Codex 继续代劳配置。

不展开介绍：

- 如何写 prompt；
- 如何让 Codex 开发功能；
- 如何让 Codex 修改代码；
- 具体项目工作流。

## 2. 安装 Codex

### 2.1 安装前检查

先确认本机有 Node.js 和 npm：

```bash
node -v
npm -v
```

如果没有，先安装 Node.js LTS。安装完成后，关闭并重新打开终端，再执行上面的检查命令。

### 2.2 安装 Codex CLI

推荐使用 npm 安装：

```bash
npm i -g @openai/codex
```

验证：

```bash
codex --version
codex --help
```

看到版本号和帮助信息即可。

### 2.3 macOS Homebrew 安装

macOS 也可以使用 Homebrew：

```bash
brew install --cask codex
```

验证：

```bash
codex --version
```

### 2.4 二进制安装

如果不能使用 npm 或 Homebrew，可以从 OpenAI Codex GitHub Release 下载对应平台的二进制包，然后放入 PATH。

### 2.5 桌面 App 安装

如果不想只使用命令行，也可以安装 Codex 桌面 App。

常见方式：

- 运行：

```bash
codex app
```

- Windows 用户也可以在 **Microsoft Store** 搜索并安装 Codex 桌面 App；
- 也可以从官方 Codex App 页面进入安装入口。

桌面 App 和 CLI 可以并存。团队做统一配置时，仍建议先保证 CLI 可用，因为 MCP、Plugin、配置排查等通常更容易通过 CLI 验证。

### 2.6 Windows 注意事项

如果 PowerShell 提示：

```text
codex.ps1 cannot be loaded because running scripts is disabled on this system
```

执行：

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

如果提示找不到 `codex`，检查 npm 全局目录：

```powershell
npm config get prefix
```

通常需要把类似下面的目录加入 PATH：

```text
C:\Users\<用户名>\AppData\Roaming\npm
```

## 3. 认证配置

Codex 常见认证方式有两种。

| 方式 | 适合场景 |
| --- | --- |
| ChatGPT 登录 | 个人开发、团队成员日常使用 |
| API Key 登录 | CI/CD、服务器、自动化脚本 |

### 3.1 ChatGPT 登录

```bash
codex login
```

查看状态：

```bash
codex login status
```

退出登录：

```bash
codex logout
```

### 3.2 API Key 登录

macOS / Linux：

```bash
export OPENAI_API_KEY="sk-..."
printf "%s" "$OPENAI_API_KEY" | codex login --with-api-key
```

Windows PowerShell：

```powershell
$env:OPENAI_API_KEY = "sk-..."
$env:OPENAI_API_KEY | codex login --with-api-key
```

注意：API Key 不要写入 README、仓库、截图、日志或共享聊天记录。

### 3.3 无浏览器环境

远程服务器或容器环境可尝试：

```bash
codex login --device-auth
```

如果不适用，就使用 API Key 登录。

## 4. 配置文件位置

Codex 的用户级配置通常在：

```text
~/.codex/config.toml
```

Windows 示例：

```text
C:\Users\<用户名>\.codex\config.toml
```

这个目录可能包含：

- Codex 全局配置；
- 登录状态或认证相关文件；
- MCP 配置；
- Plugin 配置；
- Skill 或缓存。

因此不要把整个 `~/.codex/` 提交到仓库。

## 5. 基础配置项

Codex CLI 支持两种配置方式：

| 方式 | 作用 |
| --- | --- |
| `~/.codex/config.toml` | 持久配置 |
| `codex -c key=value` | 本次运行临时覆盖配置 |

### 5.1 临时覆盖配置

示例：

```bash
codex -c model="gpt-5.4"
```

嵌套字段可使用点号：

```bash
codex -c shell_environment_policy.inherit=all
```

### 5.2 全局配置示例

`~/.codex/config.toml` 示例：

```toml
model = "gpt-5.4"
model_provider = "OpenAI"
model_reasoning_effort = "medium"
```

团队可以提供一份 `config.example.toml` 作为参考，但不要直接共享个人完整配置文件。

### 5.3 Provider 配置示例

如果使用 OpenAI 官方服务，通常不需要手写 provider。  
如果组织有代理、网关或兼容服务，可在配置中声明 provider。

```toml
model_provider = "OpenAI"

[model_providers.OpenAI]
name = "OpenAI"
base_url = "https://api.openai.com/v1"
wire_api = "responses"
requires_openai_auth = true
```

如果团队使用内部网关，把 `base_url` 改成团队提供的地址。

### 5.4 沙箱和审批配置

Codex 的命令执行通常涉及两个概念：

| 概念 | 含义 |
| --- | --- |
| sandbox | 限制 Codex 可读写和可执行的范围 |
| approval | 运行敏感命令前是否需要用户确认 |

常见命令行参数：

```bash
codex --sandbox read-only
codex --sandbox workspace-write
codex --sandbox danger-full-access
codex --ask-for-approval on-request
codex --ask-for-approval never
```

建议：

- 只做配置检查：`read-only`；
- 日常配置仓库：`workspace-write`；
- 谨慎使用：`danger-full-access`；
- 新手不要使用：`--dangerously-bypass-approvals-and-sandbox`。

### 5.5 Feature flags

查看 feature 相关命令：

```bash
codex features --help
```

常见操作：

```bash
codex features list
codex features enable <feature-name>
codex features disable <feature-name>
```

## 6. 项目级配置

Codex 会记录项目的信任状态。配置中可能出现类似：

```toml
[projects.'C:\Users\you\project']
trust_level = "trusted"
```

建议：

- 只信任你确认安全的项目；
- 不要随意信任陌生仓库；
- 不要把个人项目路径配置作为团队配置提交。

团队项目约束更适合写在仓库文档中，例如：

```text
README.md
AGENTS.md
CONTRIBUTING.md
```

## 7. MCP 简介与配置入口

### 7.1 MCP 是什么

MCP（Model Context Protocol）用于让 Codex 连接外部工具和上下文，例如：

- 文档查询；
- 浏览器自动化；
- 数据库；
- Figma；
- 内部系统。

### 7.2 让 Codex 代劳配置

安装好 Codex 后，可以直接让 Codex 帮你配置 MCP。示例请求：

```text
请帮我为当前 Codex 配置 Context7 MCP，并说明你会修改哪些配置。
```

```text
请检查我当前已配置的 MCP server，并指出哪些可能不可用。
```

### 7.3 常用 MCP 命令

```bash
codex mcp --help
codex mcp list
codex mcp get <server-name>
codex mcp add <server-name> -- <command> [args...]
codex mcp add <server-name> --url <mcp-url>
codex mcp remove <server-name>
```

示例：

```bash
codex mcp add context7 -- npx -y @upstash/context7-mcp
```

## 8. Skill 简介与配置入口

### 8.1 Skill 是什么

Skill 是 Codex 的可复用工作流配置包。它通常包含一个 `SKILL.md`，用于告诉 Codex：遇到某类任务时应该遵循什么流程。

最小结构：

```text
example-skill/
  SKILL.md
```

### 8.2 让 Codex 代劳配置

安装好 Codex 后，可以让 Codex 帮你创建或安装 Skill。示例请求：

```text
请帮我创建一个用于维护本仓库 Codex 安装与配置文档的 Skill。
```

```text
请检查当前 Codex 可用的 Skills，并说明哪些和文档维护相关。
```

如果当前环境包含内置 Skill，也可以尝试：

```text
$skill-creator
$skill-installer
```

### 8.3 Skill 配置建议

`SKILL.md` 的 `description` 要写清楚触发条件。

示例：

```markdown
---
name: codex-config-docs
description: 当用户要求维护 Codex 安装、配置、MCP、Skill 或 Plugin 文档时使用。
---

请保持文档聚焦安装与配置，不展开具体代码开发用法。
```

## 9. Plugin 简介与配置入口

### 9.1 Plugin 是什么

Plugin 是更适合分发的扩展包。一个 Plugin 可以包含：

- Skills；
- MCP 配置；
- App 集成；
- 模板、图标或其他资源。

简单区分：

| 名称 | 作用 |
| --- | --- |
| MCP | 连接外部工具 |
| Skill | 沉淀工作流程 |
| Plugin | 打包和分发扩展能力 |

### 9.2 让 Codex 代劳配置

安装好 Codex 后，可以让 Codex 帮你配置 Plugin。示例请求：

```text
请检查当前 Codex 已安装和启用的 Plugins。
```

```text
请帮我添加一个团队 Codex plugin marketplace，并说明配置会写到哪里。
```

### 9.3 常用 Plugin 命令

```bash
codex plugin --help
codex plugin marketplace --help
codex plugin marketplace add <source>
```

`<source>` 可以是：

- `owner/repo[@ref]`
- HTTP(S) Git URL
- SSH URL
- 本地 marketplace 目录

## 10. 配置校验

完成安装和配置后，按顺序检查：

```bash
codex --version
codex login status
codex --help
codex mcp list
codex plugin --help
```

如果使用 feature：

```bash
codex features --help
```

如果配置了 MCP：

```bash
codex mcp get <server-name>
```

如果配置了 Skill 或 Plugin，建议重启 Codex 后再检查：

```text
/skills
/plugins
```

## 11. 命令速查

```bash
# 安装 / 升级
npm i -g @openai/codex
npm i -g @openai/codex@latest

# 桌面 App
codex app

# 基础信息
codex --version
codex --help

# 登录
codex login
codex login status
codex login --device-auth
codex logout

# API Key 登录
printf "%s" "$OPENAI_API_KEY" | codex login --with-api-key

# 临时配置覆盖
codex -c model="gpt-5.4"
codex -c shell_environment_policy.inherit=all

# 沙箱和审批
codex --sandbox read-only
codex --sandbox workspace-write
codex --ask-for-approval on-request

# MCP
codex mcp --help
codex mcp list
codex mcp get <server-name>
codex mcp add context7 -- npx -y @upstash/context7-mcp
codex mcp remove <server-name>

# Plugin
codex plugin --help
codex plugin marketplace --help
codex plugin marketplace add <source>

# Feature flags
codex features --help
codex features list
```

## 12. 参考资料

- Codex 官方首页：https://developers.openai.com/codex
- Codex Quickstart：https://developers.openai.com/codex/quickstart
- Codex CLI：https://developers.openai.com/codex/cli
- Codex Authentication：https://developers.openai.com/codex/auth
- Codex Config Reference：https://developers.openai.com/codex/config-reference
- Codex MCP：https://developers.openai.com/codex/mcp
- Codex Skills：https://developers.openai.com/codex/skills
- Codex Plugins：https://developers.openai.com/codex/plugins
- Codex 桌面 App：https://chatgpt.com/codex?app-landing-page=true
- Microsoft Store Codex App：https://apps.microsoft.com/detail/9plm9xgg6vks
- OpenAI Codex GitHub：https://github.com/openai/codex
