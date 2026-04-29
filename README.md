# Codex 从零安装与扩展指南

> 适用目标：从一台没有 Codex 的新机器开始，完成 Codex CLI 安装、登录、验证，并了解如何继续安装 MCP、Skill、Plugin 等扩展能力。  
> 最后更新：2026-04-29。Codex 发布较快，执行前建议先看官方文档和 `codex --help`。

## 目录

- [0. 新手最快路径](#0-新手最快路径)
- [1. Codex 是什么](#1-codex-是什么)
- [2. 安装路线选择](#2-安装路线选择)
- [3. 安装前准备](#3-安装前准备)
- [4. 从零安装 Codex CLI](#4-从零安装-codex-cli)
- [5. 登录与认证](#5-登录与认证)
- [6. 验证安装成功](#6-验证安装成功)
- [7. 第一次使用建议](#7-第一次使用建议)
- [8. 新手常见错误与排查](#8-新手常见错误与排查)
- [9. 升级与卸载](#9-升级与卸载)
- [10. MCP 简介与安装](#10-mcp-简介与安装)
- [11. Skill 简介与安装](#11-skill-简介与安装)
- [12. Plugin 简介与安装](#12-plugin-简介与安装)
- [13. 高频 QA](#13-高频-qa)
- [14. 常用命令速查](#14-常用命令速查)
- [15. 参考资料](#15-参考资料)

## 0. 新手最快路径

如果你只想先把 Codex 跑起来，按这个顺序做：

### Windows PowerShell

```powershell
node -v
npm -v
npm i -g @openai/codex
codex --version
codex login
codex login status
codex
```

### macOS / Linux

```bash
node -v
npm -v
npm i -g @openai/codex
codex --version
codex login
codex login status
codex
```

如果其中任何一步报错，先不要跳步骤，直接去看 [8. 新手常见错误与排查](#8-新手常见错误与排查)。

## 1. Codex 是什么

Codex 是 OpenAI 的软件开发智能体，可以在本地或云端帮助你：

- 阅读、解释和理解代码库；
- 按需求生成或修改代码；
- 运行命令、测试、构建并根据结果继续修复；
- 做代码审查、调试、迁移、重构等工程任务；
- 通过 MCP、Skill、Plugin 扩展到更多工具和工作流。

本指南重点介绍 **Codex CLI**，也就是在终端里运行的 Codex。

## 2. 安装路线选择

Codex 有多种入口：

| 入口 | 适合人群 | 说明 |
| --- | --- | --- |
| Codex CLI | 开发者、终端用户、自动化任务 | 本指南重点介绍。适合在本地仓库中直接运行。 |
| Codex App | 偏图形化操作的用户 | 桌面应用体验。通常可以通过 `codex app` 进入。 |
| IDE Extension | VS Code、Cursor、Windsurf 等 IDE 用户 | 在编辑器侧边栏中使用 Codex。 |
| Codex Cloud / Web | 想把任务交给云端跑的用户 | 需要配置云端环境、仓库权限。 |

如果团队要统一落地，推荐先装 **Codex CLI**：它最容易验证、排错，也便于后续配置 MCP、Skill、Plugin。

## 3. 安装前准备

### 3.1 系统要求

Codex CLI 支持 macOS、Windows 和 Linux。Windows 用户可以直接在 PowerShell 中运行；如果项目强依赖 Linux 工具链，也可以用 WSL2。

建议准备：

- 一个可用终端：PowerShell、Windows Terminal、Terminal、iTerm2、bash、zsh 等；
- Git：用于代码版本管理和回滚；
- Node.js / npm：使用 npm 安装 Codex CLI 时需要；
- OpenAI / ChatGPT 账号，或 OpenAI API Key；
- 能访问 npm registry、OpenAI 登录页面和相关 API 的网络环境。

### 3.2 安装前自检清单

新手最容易在环境阶段出错。安装前建议逐项检查：

| 检查项 | 命令 | 正常结果 | 如果异常 |
| --- | --- | --- | --- |
| Node.js 是否安装 | `node -v` | 输出版本号，如 `v20.x.x` | 安装 Node.js LTS 后重新打开终端 |
| npm 是否安装 | `npm -v` | 输出版本号 | 重新安装 Node.js，确认勾选 npm |
| Git 是否安装 | `git --version` | 输出版本号 | 安装 Git；不是安装 Codex 的硬性要求，但强烈建议 |
| 终端是否能联网 | `npm view @openai/codex version` | 输出可安装版本 | 检查代理、公司网络、DNS、证书 |
| 是否在正确目录 | `pwd` 或 `Get-Location` | 显示你的项目目录 | 先 `cd` 到项目目录再运行 Codex |

Windows PowerShell 示例：

```powershell
node -v
npm -v
git --version
npm view @openai/codex version
Get-Location
```

macOS / Linux 示例：

```bash
node -v
npm -v
git --version
npm view @openai/codex version
pwd
```

### 3.3 检查 Node.js 与 npm

```bash
node -v
npm -v
```

如果命令不存在，先安装 Node.js LTS：

- Windows / macOS：从 Node.js 官网下载安装包，或使用系统包管理器；
- macOS：也可使用 Homebrew；
- Linux：使用发行版包管理器、nvm、fnm 等方式安装。

安装后如果仍然找不到 `node` 或 `npm`，通常是 PATH 没刷新：关闭终端，重新打开，再执行一次检查。

### 3.4 建议先准备 Git 检查点

Codex 可以修改文件、运行命令。进入项目后，建议先初始化或确认 Git 状态：

```bash
git status
```

如果是新项目：

```bash
git init
git add .
git commit -m "initial commit"
```

如果暂时不想提交，也至少先确认当前改动：

```bash
git diff
git status
```

之后每次让 Codex 做较大改动前后，都建议提交或至少确认 diff，方便回滚。

### 3.5 Windows、WSL、macOS 环境不要混用

如果你在 Windows PowerShell 安装了 Codex，只代表 Windows 环境有 `codex`；WSL 里不会自动有。反过来也一样。

判断自己在哪个环境：

```bash
codex --version
node -v
npm -v
```

如果你准备在 WSL 里开发，就在 WSL 里安装 Node.js 和 Codex；如果准备在 Windows PowerShell 里开发，就在 Windows 里安装。

## 4. 从零安装 Codex CLI

### 4.1 方式一：npm 安装（推荐通用方式）

适用于 Windows、macOS、Linux。

```bash
npm i -g @openai/codex
```

安装完成后检查：

```bash
codex --version
codex --help
```

如果能看到版本号和帮助信息，说明 Codex CLI 已经进入 PATH。

### 4.2 方式二：Homebrew 安装（macOS 常用）

官方仓库也提供 Homebrew 安装方式。常见命令如下：

```bash
brew install --cask codex
```

如果你的 Homebrew 提示没有 cask，可先查看：

```bash
brew search codex
```

然后验证：

```bash
codex --version
```

### 4.3 方式三：下载二进制包

如果机器不能使用 npm 或 Homebrew，可以从 OpenAI Codex GitHub Release 下载对应平台的二进制包。

一般流程：

1. 下载对应系统和 CPU 架构的压缩包；
2. 解压得到可执行文件；
3. 将文件重命名为 `codex` 或 `codex.exe`；
4. 放到 PATH 中的目录；
5. 执行 `codex --version` 验证。

### 4.4 Windows 注意事项

PowerShell 中推荐这样验证：

```powershell
codex --version
codex --help
```

如果提示找不到 `codex`：

1. 关闭并重新打开终端；
2. 检查 npm 全局目录是否在 PATH 中：

```powershell
npm config get prefix
```

3. 常见 npm 全局目录包括：
   - `C:\Users\<用户名>\AppData\Roaming\npm`
   - 或你安装 Node.js / Anaconda / nvm 时配置的全局目录。

如果 PowerShell 提示 `codex.ps1 cannot be loaded because running scripts is disabled`，这是执行策略问题，可选择其一：

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

然后重新打开 PowerShell。或者临时使用 `cmd.exe` 验证：

```bat
codex --version
```

如果安装时出现权限问题，不建议一开始就用管理员权限暴力安装；优先检查 Node/npm 安装位置和用户目录权限。

### 4.5 macOS / Linux 权限注意事项

如果 `npm i -g` 报 `EACCES`、`permission denied`：

- 不要马上 `sudo npm i -g ...`，除非你清楚这会把全局包装到系统目录；
- 推荐使用 nvm / fnm 管理 Node.js，让全局 npm 包安装到用户目录；
- 或修复 npm 全局目录权限。

常见检查：

```bash
npm config get prefix
which node
which npm
```

如果你用 Homebrew 安装 Node.js，通常不需要 `sudo`。

## 5. 登录与认证

Codex 使用 OpenAI 模型时支持两类登录：

| 方式 | 适合场景 | 说明 |
| --- | --- | --- |
| ChatGPT 登录 | 个人开发、日常交互、团队订阅 | Codex 会打开浏览器完成登录。 |
| API Key 登录 | CI/CD、脚本化、用量计费工作流 | 使用 OpenAI Platform API key，按 API 账户计费。 |

### 5.1 默认登录：ChatGPT

直接运行：

```bash
codex
```

首次运行会提示登录。按提示选择 ChatGPT 登录，浏览器完成认证后回到终端。

也可以显式执行：

```bash
codex login
```

查看登录状态：

```bash
codex login status
```

退出登录：

```bash
codex logout
```

### 5.2 API Key 登录

适合自动化环境或不方便浏览器登录的机器。

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

注意：不要把 API Key 写入仓库、截图、日志或公开聊天记录。

### 5.3 无浏览器 / 远程机器登录

如果运行在远程服务器、容器或无图形界面的机器上，可以尝试：

```bash
codex login --device-auth
```

如果组织未启用 device code 登录，则可改用 API Key，或在可信机器登录后迁移认证缓存。认证缓存一般在 `~/.codex/` 下，属于敏感文件，不要提交到 Git。

### 5.4 登录后还是不可用怎么办

先查状态：

```bash
codex login status
```

再确认你登录的是正确账号：

- ChatGPT 登录：确认账号订阅、组织权限、企业策略允许使用 Codex；
- API Key 登录：确认 key 没有过期、没有复制空格、所属 project 有可用额度；
- 公司网络：确认没有拦截 OpenAI 登录回调、WebSocket 或 API 请求。

如果一直卡在浏览器回调，可以：

1. 关闭 Codex；
2. 重新执行 `codex login`；
3. 换默认浏览器；
4. 尝试 `codex login --device-auth`；
5. 最后改用 API Key 登录。

## 6. 验证安装成功

按顺序执行：

```bash
codex --version
codex login status
codex --help
```

进入一个项目目录：

```bash
cd /path/to/your/project
codex
```

在 Codex 交互界面中输入：

```text
请用中文概述这个仓库的结构，不要修改文件。
```

如果 Codex 能读取目录并返回说明，说明基本可用。

也可以做一次非交互测试：

```bash
codex exec "列出当前仓库的主要文件，并说明你不会修改任何文件"
```

### 6.1 验证结果怎么判断

| 现象 | 说明 | 下一步 |
| --- | --- | --- |
| `codex --version` 有输出 | CLI 安装成功 | 继续登录 |
| `codex login status` 显示已登录 | 认证成功 | 进入项目运行 `codex` |
| `codex` 打开交互界面 | 基础功能可用 | 先做只读任务 |
| 能总结仓库结构 | 能读取本地上下文 | 可以开始小范围改动 |
| 能执行 `codex exec` | 非交互模式可用 | 可用于脚本或 CI 场景 |

## 7. 第一次使用建议

### 7.1 从只读任务开始

新用户不要一上来就让 Codex 大范围改代码。建议先问：

```text
请分析这个项目的目录结构和技术栈，不要修改任何文件。
```

```text
请找出这个仓库可能的测试命令和构建命令，不要运行，只给出依据。
```

### 7.2 再让 Codex 做小改动

例如：

```text
请在 README 中补充本地启动步骤。改动前先说明计划，改完后总结变更。
```

### 7.3 明确边界，减少误改

给 Codex 的任务最好包含范围、限制和验收标准：

```text
只允许修改 docs/ 目录，不要改业务代码。改动前先列计划，改完后说明改了哪些文件。
```

```text
请修复这个测试失败。只修改 src/foo.ts 和对应测试文件，不要改 package.json。
```

### 7.4 使用 Git 审查结果

Codex 修改后：

```bash
git diff
git status
```

确认无误再提交：

```bash
git add .
git commit -m "update docs with Codex"
```

### 7.5 理解 approval、sandbox 和危险模式

Codex 可能会在运行命令前请求确认，这是正常现象。

常见概念：

- `read-only`：只读，适合分析代码；
- `workspace-write`：允许修改当前工作区，适合日常开发；
- `danger-full-access`：权限很高，只适合隔离环境；
- `--dangerously-bypass-approvals-and-sandbox`：跳过确认和沙箱，新手不要使用。

新手建议：默认设置即可；如果 Codex 要运行删除、重置、安装依赖、改系统配置等命令，先看清楚再确认。

## 8. 新手常见错误与排查

### 8.1 一张表快速定位

| 报错 / 现象 | 最可能原因 | 处理方式 |
| --- | --- | --- |
| `node` 不是内部或外部命令 | 没装 Node.js 或 PATH 未刷新 | 安装 Node.js LTS，重新打开终端 |
| `npm` 不是内部或外部命令 | Node.js 安装不完整 | 重新安装 Node.js，确认包含 npm |
| `codex` 不是内部或外部命令 | Codex 没装成功或 npm 全局目录不在 PATH | 重新打开终端，检查 `npm config get prefix` |
| `codex.ps1 cannot be loaded` | PowerShell 执行策略限制 | `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` |
| `EACCES` / `permission denied` | npm 全局目录权限问题 | 用 nvm/fnm 或修复 npm prefix，不优先 sudo |
| npm 下载很慢或失败 | 网络、代理、DNS、证书问题 | 检查 `npm view @openai/codex version`、代理配置 |
| 登录后回不到终端 | 浏览器回调被拦截或默认浏览器异常 | 重试 `codex login`，换浏览器，或用 `--device-auth` |
| API Key 登录失败 | key 错误、复制了空格、额度或 project 权限问题 | 重新生成 key，确认 billing/project |
| Codex 说读不到文件 | 不在项目目录，或沙箱权限限制 | `cd` 到项目目录，检查 `pwd`/`Get-Location` |
| Codex 修改了不该改的文件 | 任务边界不清晰 | 用 Git 回滚，重新给出明确范围 |
| MCP 安装后不可见 | 没重启 Codex 或 MCP server 启动失败 | `codex mcp list`，重启 Codex，看 server 命令是否可运行 |
| Skill 没触发 | description 不清楚或目录放错 | 检查 `SKILL.md` 元数据和安装位置，重启 Codex |

### 8.2 `codex` 命令不存在

处理顺序：

1. 重新打开终端；
2. 再执行：

```bash
codex --version
```

3. 检查 npm 全局目录：

```bash
npm config get prefix
```

4. 重新安装：

```bash
npm i -g @openai/codex
```

5. 如果仍然不行，把 npm 全局 bin 目录加入 PATH。

Windows 常见目录：

```text
C:\Users\<用户名>\AppData\Roaming\npm
```

macOS / Linux 常见检查：

```bash
npm config get prefix
which codex
```

### 8.3 PowerShell 脚本执行被禁用

如果看到类似：

```text
codex.ps1 cannot be loaded because running scripts is disabled on this system
```

执行：

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

然后关闭并重新打开 PowerShell。

如果你没有权限修改执行策略，可以临时使用 `cmd.exe` 或 Windows Terminal 的其他 shell。

### 8.4 npm 全局安装权限错误

如果看到：

```text
EACCES: permission denied
```

macOS / Linux 推荐：

1. 安装 nvm 或 fnm；
2. 用它安装 Node.js LTS；
3. 重新执行：

```bash
npm i -g @openai/codex
```

不建议新手直接：

```bash
sudo npm i -g @openai/codex
```

因为后续升级、卸载、全局包权限可能更混乱。

### 8.5 网络、代理和证书问题

先判断是 npm 下载问题，还是 Codex 登录 / API 问题。

npm 下载测试：

```bash
npm view @openai/codex version
```

如果这一步失败，优先排查：

- npm registry 是否可访问；
- 公司代理是否需要配置；
- HTTPS 证书是否被公司网关替换；
- DNS 是否正常。

常见代理配置示例：

```bash
npm config set proxy http://127.0.0.1:7890
npm config set https-proxy http://127.0.0.1:7890
```

如果是 Codex 登录或 API 请求失败，重点检查：

- 是否能打开 OpenAI / ChatGPT 登录页面；
- 公司网络是否拦截登录回调；
- 是否需要系统代理，而不仅仅是 npm 代理；
- 是否有自定义 CA 证书要求。

### 8.6 登录成功但 Codex 无法使用模型

可能原因：

- 登录了错误的 ChatGPT 账号；
- 当前账号或组织没有对应权限；
- API Key 所属 project 没有额度；
- 企业网络阻止 API 请求；
- 模型或区域策略限制。

排查：

```bash
codex login status
codex --help
```

然后换账号或换 API Key 再登录：

```bash
codex logout
codex login
```

### 8.7 Codex 在空目录里运行

新手经常在错误目录启动 Codex，导致它“看不到项目”。

运行前先看当前目录：

Windows PowerShell：

```powershell
Get-Location
Get-ChildItem
```

macOS / Linux：

```bash
pwd
ls
```

如果不对，先切到项目目录：

```bash
cd /path/to/your/project
codex
```

也可以直接指定目录：

```bash
codex --cd /path/to/your/project
```

### 8.8 Codex 改坏了文件怎么办

先不要继续让 Codex 大量修改。查看变化：

```bash
git status
git diff
```

回滚单个文件：

```bash
git restore <file>
```

如果没有 Git，只能手动恢复，所以强烈建议在使用 Codex 前初始化 Git。

### 8.9 Codex 一直要求确认命令

这是 approval 机制，不是错误。新手可以先保持默认。

如果你只是想让 Codex 分析代码，可以明确说：

```text
不要运行命令，不要修改文件，只阅读并总结。
```

如果你要让它跑测试，可以明确允许：

```text
可以运行测试命令，但不要安装新依赖，不要删除文件。
```

## 9. 升级与卸载

### 9.1 升级 Codex CLI

npm：

```bash
npm i -g @openai/codex@latest
```

Homebrew：

```bash
brew upgrade --cask codex
```

如果你不是用 cask 安装，则按实际安装方式升级。升级后确认：

```bash
codex --version
```

### 9.2 卸载

npm：

```bash
npm uninstall -g @openai/codex
```

Homebrew：

```bash
brew uninstall --cask codex
```

如果你不是用 cask 安装，则按实际安装方式卸载。

### 9.3 清理配置

Codex 配置和认证通常在：

```text
~/.codex/
```

这里可能包含登录信息、MCP 配置、Skills、缓存等。除非你确定不再需要，否则不要直接删除整个目录。

## 10. MCP 简介与安装

### 10.1 MCP 是什么

MCP（Model Context Protocol）用于把模型连接到外部工具和上下文。对 Codex 来说，它可以让 Codex 使用额外工具，例如文档查询、浏览器自动化、Figma、数据库、内部系统等。

### 10.2 Codex 中查看 MCP 命令

```bash
codex mcp --help
```

常用子命令：

```bash
codex mcp list
codex mcp get <server-name>
codex mcp add <server-name> -- <stdio-server-command>
codex mcp add <server-name> --url <http-server-url>
codex mcp login <server-name>
codex mcp remove <server-name>
```

### 10.3 添加一个 STDIO MCP Server

示例：添加 Context7 文档 MCP：

```bash
codex mcp add context7 -- npx -y @upstash/context7-mcp
```

检查：

```bash
codex mcp list
```

进入 Codex TUI 后也可以输入：

```text
/mcp
```

新手注意：

- `npx` 来自 npm，如果 `npx` 不存在，先修复 Node.js/npm；
- 第一次运行 MCP server 可能需要下载包，所以网络要可用；
- 添加后如果 Codex 已经打开，建议重启 Codex；
- 如果 MCP server 需要 token，不要把 token 写进仓库。

### 10.4 添加一个 HTTP MCP Server

```bash
codex mcp add my-docs --url https://example.com/mcp
```

如果需要 bearer token：

```bash
codex mcp add my-docs --url https://example.com/mcp --bearer-token-env-var MY_DOCS_TOKEN
```

然后在环境变量里设置 token，而不是写进命令历史或配置文件。

### 10.5 MCP 配置文件位置

默认配置通常在：

```text
~/.codex/config.toml
```

如果团队要共享 MCP 配置，先确认其中没有个人 token、内网地址或敏感信息，再考虑提交到仓库。

## 11. Skill 简介与安装

### 11.1 Skill 是什么

Skill 是 Codex 的可复用工作流包。一个 Skill 通常是一个目录，包含必需的 `SKILL.md`，以及可选的脚本、参考资料和模板资源。

最小结构：

```text
my-skill/
  SKILL.md
```

复杂结构：

```text
my-skill/
  SKILL.md
  scripts/
  references/
  assets/
```

`SKILL.md` 需要包含 `name` 和 `description` 元数据。

### 11.2 Skill 保存位置

常见位置：

| 范围 | 位置 | 用途 |
| --- | --- | --- |
| 用户级 | `~/.codex/skills/` | 当前用户所有项目可用。 |
| 仓库级 | `.codex/skills/` 或团队约定目录 | 团队共享、随仓库提交。提交前确认当前 Codex 版本支持该位置。 |
| 插件级 | 插件目录内 | 随 Plugin 安装和启用。 |
| 系统级 | Codex 内置 | OpenAI 随 Codex 提供。 |

如果不确定当前版本读取哪些位置，进入 Codex 后查看 `/skills`，或直接问 Codex：

```text
请列出当前可用 skills，并说明它们来自哪里。
```

### 11.3 使用内置 Skill Creator

在 Codex 交互界面中可以输入：

```text
$skill-creator
```

Codex 会引导你创建一个 Skill：说明用途、触发条件、是否需要脚本等。

### 11.4 安装 curated skills

如果当前 Codex 环境包含 `$skill-installer`，可以在 Codex 中输入：

```text
$skill-installer linear
```

也可以让它从其他仓库下载 Skill。安装后 Codex 通常会自动检测；如果没有出现，重启 Codex。

### 11.5 手动创建用户级 Skill

在用户级目录创建：

```text
~/.codex/skills/example-skill/SKILL.md
```

示例内容：

```markdown
---
name: example-skill
description: 当用户要求整理本仓库文档结构时使用；不用于业务代码修改。
---

请先扫描 README 和 docs/，再给出文档结构建议。修改前先列计划。
```

然后重启 Codex 或在新会话中测试：

```text
$example-skill 请帮我检查文档结构
```

### 11.6 Skill 新手常见问题

| 现象 | 原因 | 处理 |
| --- | --- | --- |
| Skill 不出现 | 放错目录或 Codex 未重启 | 确认目录和 `SKILL.md`，重启 Codex |
| Skill 不自动触发 | `description` 太模糊 | 把触发场景写具体 |
| Skill 执行脚本失败 | 脚本依赖不存在或无权限 | 在 `scripts/` 中写清依赖和运行方式 |
| 团队成员不可用 | 只安装到了个人目录 | 改为仓库级或 Plugin 分发 |

## 12. Plugin 简介与安装

### 12.1 Plugin 是什么

Plugin 是 Codex 中更适合分发的扩展单位。一个 Plugin 可以打包：

- 一个或多个 Skills；
- App 集成；
- MCP Server 配置；
- 图标、展示信息等资源。

简单理解：

- **Skill**：定义“怎么做某类工作”；
- **MCP**：提供“能连接哪些工具”；
- **Plugin**：把 Skill、App、MCP 等打包成可安装能力。

### 12.2 在 Codex CLI 中打开插件列表

先启动 Codex：

```bash
codex
```

在交互界面输入：

```text
/plugins
```

然后可以浏览、搜索、安装、启用或禁用插件。

### 12.3 管理 Plugin Marketplace

查看帮助：

```bash
codex plugin --help
codex plugin marketplace --help
```

添加 marketplace：

```bash
codex plugin marketplace add <owner/repo>
```

也支持 HTTP(S) Git URL、SSH URL 或本地 marketplace 目录。具体格式以：

```bash
codex plugin marketplace add --help
```

输出为准。

### 12.4 Plugin 新手注意事项

- 不要安装来源不明的 Plugin；
- Plugin 可能带 MCP、脚本或外部集成，安装前先看说明；
- 如果启用后 Codex 行为异常，先禁用最近安装的 Plugin；
- 团队统一分发时，建议固定 marketplace 版本或 commit，避免每个人装到不同版本。

## 13. 高频 QA

本节整理新手、团队管理员和日常开发中最常被问到的问题。遇到问题时，建议先看对应分类，再回到前面的详细章节排查。

### 13.1 安装类 QA

#### Q1：我应该安装 Codex CLI、Codex App，还是 IDE 插件？

如果你是第一次接触，优先安装 **Codex CLI**。原因是：

- 安装和验证最直接；
- 报错信息更容易复制和排查；
- 后续配置 MCP、Skill、Plugin 都绕不开 CLI；
- 团队写文档、做培训时也最容易统一。

等 CLI 跑通后，再按个人习惯安装 Codex App 或 IDE 插件。

#### Q2：我已经安装了 Node.js，为什么 `npm` 还是不能用？

常见原因：

- 安装 Node.js 时没有包含 npm；
- 终端没有重新打开，PATH 还没刷新；
- Windows 上安装到了一个用户目录，但当前终端使用的是另一个环境；
- WSL 和 Windows 环境混用了。

先执行：

```bash
node -v
npm -v
```

如果 `node` 有、`npm` 没有，建议重新安装 Node.js LTS，并确认 npm 被安装。

#### Q3：能不能不用 npm 安装 Codex？

可以。常见替代方式：

- macOS 使用 Homebrew；
- 从 OpenAI Codex GitHub Release 下载二进制包；
- 使用团队预置的开发机、镜像或内部安装包。

但新手最推荐 npm，因为后续升级也简单：

```bash
npm i -g @openai/codex@latest
```

#### Q4：Windows 上需要管理员权限安装吗？

通常不需要。推荐让 npm 全局包安装到当前用户目录。

只有当你的 Node.js/npm 装在系统目录、且当前用户没有写权限时，才可能遇到权限问题。不要一上来就用管理员 PowerShell，优先检查：

```powershell
npm config get prefix
```

#### Q5：公司电脑无法下载 `@openai/codex` 怎么办？

先判断是 npm 网络问题还是 OpenAI 登录/API 问题。

```bash
npm view @openai/codex version
```

如果这一步失败，通常是 npm registry、代理、DNS 或公司证书问题。需要配置 npm 代理或联系 IT。  
如果这一步成功，但 Codex 登录失败，则重点排查浏览器登录、系统代理、OpenAI 域名访问和企业网络策略。

### 13.2 登录与账号 QA

#### Q6：应该用 ChatGPT 登录还是 API Key 登录？

一般建议：

| 场景 | 推荐方式 |
| --- | --- |
| 个人本地开发 | ChatGPT 登录 |
| 团队成员日常使用 | ChatGPT 登录或组织统一策略 |
| CI/CD、自动化脚本 | API Key 登录 |
| 远程无浏览器服务器 | Device auth 或 API Key |
| 不希望绑定个人 ChatGPT | API Key 或企业统一账号策略 |

#### Q7：登录成功了，但 Codex 还是不能回答，为什么？

可能原因：

- 登录的不是你有权限的账号；
- 当前 ChatGPT 计划或组织策略不允许使用 Codex；
- API Key 没有额度或 project 权限不对；
- 公司网络拦截了 API 请求；
- 本地代理只配置给浏览器，没有配置给终端。

先看：

```bash
codex login status
```

必要时退出重登：

```bash
codex logout
codex login
```

#### Q8：API Key 要放在哪里？可以写进 README 吗？

不可以。API Key 是敏感凭据，不能写入：

- README；
- 代码仓库；
- issue / PR；
- 截图；
- 共享聊天记录；
- shell history 中长期保留的命令。

推荐用环境变量或密钥管理系统。PowerShell 临时设置示例：

```powershell
$env:OPENAI_API_KEY = "sk-..."
$env:OPENAI_API_KEY | codex login --with-api-key
```

#### Q9：远程服务器没有浏览器怎么登录？

优先尝试：

```bash
codex login --device-auth
```

如果不支持或组织策略限制，就使用 API Key：

```bash
printf "%s" "$OPENAI_API_KEY" | codex login --with-api-key
```

### 13.3 使用方式 QA

#### Q10：Codex 会不会自动改我的文件？

取决于你给的任务和当前权限。为了安全，新手一开始可以明确要求：

```text
只分析，不要修改文件，不要运行命令。
```

等确认 Codex 理解项目后，再给小范围改动任务。

#### Q11：怎样让 Codex 少犯错？

给任务时写清楚四件事：

1. 目标：要完成什么；
2. 范围：允许改哪些文件或目录；
3. 禁止项：不能改什么、不能运行什么；
4. 验收：怎样判断完成。

示例：

```text
请补充 docs/install.md 中的 Windows 安装说明。只允许修改 docs/install.md，不要改代码。完成后列出改动摘要和验证方式。
```

#### Q12：Codex 运行命令前总问我确认，正常吗？

正常。这是 approval 机制，用来防止模型未经确认执行敏感命令。

新手建议保持默认。看到以下命令要特别谨慎：

- 删除文件；
- `git reset --hard`；
- `git clean`；
- 修改系统配置；
- 安装大量依赖；
- 上传或发送本地文件；
- 访问需要凭据的内部系统。

#### Q13：Codex 改坏了文件，第一步该做什么？

第一步不是继续让 Codex 修，而是先看 diff：

```bash
git status
git diff
```

如果只想回滚单个文件：

```bash
git restore <file>
```

如果没有 Git，只能手动恢复。因此强烈建议使用 Codex 前先初始化 Git。

#### Q14：能不能让 Codex 直接跑完整项目构建和测试？

可以，但建议逐步来：

1. 先让 Codex 找出可能的构建/测试命令；
2. 让它解释这些命令会做什么；
3. 再允许它运行；
4. 如果失败，再让它根据日志修复。

示例提示：

```text
请先找出本项目的测试命令，不要运行。说明依据后等待我确认。
```

### 13.4 MCP / Skill / Plugin QA

#### Q15：MCP、Skill、Plugin 三者有什么区别？

简单理解：

| 名称 | 解决的问题 | 类比 |
| --- | --- | --- |
| MCP | 让 Codex 连接外部工具和数据 | 插上工具接口 |
| Skill | 告诉 Codex 某类任务应该怎么做 | 工作手册 |
| Plugin | 把 Skills、MCP、App 等打包分发 | 扩展包 |

新手顺序建议：先会用 Codex CLI，再加 MCP；有重复工作流后再沉淀 Skill；需要团队分发时再做 Plugin。

#### Q16：安装 MCP 后，为什么 Codex 里看不到？

常见原因：

- Codex 会话没有重启；
- MCP server 命令本身启动失败；
- 需要的环境变量没有设置；
- `npx`、`node` 或相关依赖不存在；
- HTTP MCP 地址不可访问。

先检查：

```bash
codex mcp list
codex mcp get <server-name>
```

然后重启 Codex。

#### Q17：Skill 写好了，为什么没有触发？

常见原因：

- `SKILL.md` 位置不对；
- frontmatter 中 `name` 或 `description` 写错；
- `description` 太泛，Codex 不知道什么时候该用；
- 当前会话没有重新加载。

改进 `description` 时要写清楚触发场景，例如：

```markdown
description: 当用户要求为本仓库补充安装指南、排错指南或 README 文档结构时使用。
```

#### Q18：团队应该直接共享 Skill 目录，还是做 Plugin？

如果只是少量内部工作流，可以先共享 Skill。  
如果需要版本管理、统一启用、包含 MCP 或多项能力，建议做 Plugin。

经验判断：

- 1-3 个简单工作流：Skill 足够；
- 多个团队复用：Plugin 更合适；
- 需要绑定外部工具：Plugin + MCP 更清晰。

### 13.5 团队落地 QA

#### Q19：团队新成员入职应该怎么培训 Codex？

建议分三步：

1. 按本指南安装并登录 Codex；
2. 在一个测试仓库里练习只读分析、小改动、回滚；
3. 再进入真实业务仓库，并要求所有改动通过 Git diff / PR 审查。

不要让新成员第一次就用 Codex 修改核心业务代码。

#### Q20：团队是否应该统一 Codex 版本？

建议统一大版本或至少记录版本。不同 Codex CLI 版本可能在命令、MCP、Plugin、Skill 支持上有差异。

查看版本：

```bash
codex --version
```

团队文档中可以写明“推荐版本不低于 x.y.z”。

#### Q21：哪些文件不应该提交到仓库？

通常不要提交：

- API Key；
- `.env` 中的真实密钥；
- `~/.codex/` 下的个人认证文件；
- 带个人 token 的 MCP 配置；
- 本地缓存和日志；
- Codex 生成但未审查的大量临时代码。

建议在 `.gitignore` 中排除敏感和临时文件。

#### Q22：怎么让 Codex 更符合团队规范？

可以通过这些方式逐步增强：

- 在 README 或 docs 中写清项目规范；
- 增加 AGENTS.md / CONTRIBUTING.md；
- 沉淀团队 Skill；
- 配置 formatter、linter、test；
- 要求 Codex 修改后必须运行或说明验证方式；
- 通过 PR review 审查 Codex 产物。

### 13.6 什么时候该升级或重装

#### Q23：遇到问题应该先升级 Codex 吗？

不一定。先判断问题类型：

- 命令不存在、PATH 问题：升级通常没用；
- 登录失败：优先排查账号和网络；
- MCP/Plugin 命令缺失：可能需要升级；
- 官方文档示例和本机命令不一致：建议升级。

升级命令：

```bash
npm i -g @openai/codex@latest
codex --version
```

#### Q24：什么时候需要删除 `~/.codex/`？

很少需要。`~/.codex/` 里可能包含认证、配置、Skill、MCP 信息。删除前先备份。

更推荐的顺序：

1. `codex logout`；
2. 重新 `codex login`；
3. 检查 `~/.codex/config.toml`；
4. 必要时只移动或重命名有问题的配置文件；
5. 最后才考虑清空整个目录。

## 14. 常用命令速查

```bash
# 安装 / 升级
npm i -g @openai/codex
npm i -g @openai/codex@latest

# 基础信息
codex --version
codex --help

# 登录
codex login
codex login status
codex login --device-auth
codex logout

# 启动交互模式
codex

# 非交互执行
codex exec "总结这个仓库的结构，不要修改文件"

# 代码审查
codex review

# 指定目录启动
codex --cd /path/to/project

# MCP
codex mcp --help
codex mcp list
codex mcp add context7 -- npx -y @upstash/context7-mcp

# Plugin
codex plugin --help
codex plugin marketplace --help

# 常用交互命令（在 Codex TUI 内输入）
/help
/mcp
/plugins
/skills
```

## 15. 参考资料

- Codex 官方首页：https://developers.openai.com/codex
- Codex Quickstart：https://developers.openai.com/codex/quickstart
- Codex CLI：https://developers.openai.com/codex/cli
- Codex Authentication：https://developers.openai.com/codex/auth
- Codex MCP：https://developers.openai.com/codex/mcp
- Codex Skills：https://developers.openai.com/codex/skills
- Codex Plugins：https://developers.openai.com/codex/plugins
- OpenAI Codex GitHub：https://github.com/openai/codex

