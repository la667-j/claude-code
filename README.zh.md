# Claude Code 源码快照（安全研究）

> 本仓库镜像了在 **2026年3月31日** 通过 npm 分发包中的 source map 泄露而公开暴露的 **Claude Code 源码快照**。它仅用于**教育、防御性安全研究和软件供应链分析**。

---

## 研究背景
翻译来自https://github.com/instructkr/claude-code
本仓库由一名**大学生**维护，用于研究以下内容：

- 软件供应链暴露与构建产物泄露
- 安全软件工程实践
- 智能体开发工具架构
- 对真实世界 CLI 系统的防御性分析

本存档旨在支持：

- 教育学习
- 安全研究实践
- 架构审查
- 关于打包与发布流程失败的讨论

它**不**声称拥有原始代码的所有权，也不应被解读为 Anthropic 的官方仓库。

---

## 公开快照是如何被获取的

[Chaofan Shou (@Fried_rice)](https://x.com/Fried_rice) 公开指出，Claude Code 的源码材料可以通过 npm 包中暴露的 `.map` 文件访问：

> **"Claude Code 的源码通过 npm 注册表中的 map 文件泄露了！"**
>
> — [@Fried_rice, 2026年3月31日](https://x.com/Fried_rice/status/2038894956459290963)

公开发布的 source map 引用了托管在 Anthropic R2 存储桶中的未混淆 TypeScript 源码，这使得 `src/` 快照可以被公开下载。

---

## 仓库范围

Claude Code 是 Anthropic 推出的命令行工具（CLI），用于在终端中与 Claude 交互，执行软件工程任务，例如编辑文件、运行命令、搜索代码库和协调工作流。

本仓库包含一个用于研究和分析的 `src/` 镜像快照。

- **公开暴露发现日期**：2026-03-31
- **编程语言**：TypeScript
- **运行时**：Bun
- **终端 UI**：React + [Ink](https://github.com/vadimdemedes/ink)
- **规模**：约 1,900 个文件，512,000+ 行代码

---

## 目录结构

```text
src/
├── main.tsx                 # 入口编排（基于 Commander.js 的 CLI 路径）
├── commands.ts              # 命令注册表
├── tools.ts                 # 工具注册表
├── Tool.ts                  # 工具类型定义
├── QueryEngine.ts           # LLM 查询引擎
├── context.ts               # 系统/用户上下文收集
├── cost-tracker.ts          # Token 成本追踪
│
├── commands/                # 斜杠命令实现（约 50 个）
├── tools/                   # 智能体工具实现（约 40 个）
├── components/              # Ink UI 组件（约 140 个）
├── hooks/                   # React hooks
├── services/                # 外部服务集成
├── screens/                 # 全屏 UI（Doctor、REPL、Resume）
├── types/                   # TypeScript 类型定义
├── utils/                   # 工具函数
│
├── bridge/                  # IDE 和远程控制桥接
├── coordinator/             # 多智能体协调器
├── plugins/                 # 插件系统
├── skills/                  # 技能系统
├── keybindings/             # 快捷键配置
├── vim/                     # Vim 模式
├── voice/                   # 语音输入
├── remote/                  # 远程会话
├── server/                  # 服务器模式
├── memdir/                  # 持久化记忆目录
├── tasks/                   # 任务管理
├── state/                   # 状态管理
├── migrations/              # 配置迁移
├── schemas/                 # 配置 schema（Zod）
├── entrypoints/             # 初始化逻辑
├── ink/                     # Ink 渲染器包装器
├── buddy/                   # 伴侣精灵
├── native-ts/               # 原生 TypeScript 工具
├── outputStyles/            # 输出样式
├── query/                   # 查询管道
└── upstreamproxy/           # 代理配置
```

---

## 架构概述

### 1. 工具系统（`src/tools/`）

Claude Code 可以调用的每个工具都实现为一个独立的模块。每个工具都定义了自己的输入 schema、权限模型和执行逻辑。

| 工具 | 描述 |
|---|---|
| `BashTool` | 执行 shell 命令 |
| `FileReadTool` | 读取文件（支持图片、PDF、notebook） |
| `FileWriteTool` | 创建/覆盖文件 |
| `FileEditTool` | 部分修改文件（字符串替换） |
| `GlobTool` | 文件模式匹配搜索 |
| `GrepTool` | 基于 ripgrep 的内容搜索 |
| `WebFetchTool` | 获取 URL 内容 |
| `WebSearchTool` | 网络搜索 |
| `AgentTool` | 生成子智能体 |
| `SkillTool` | 执行技能 |
| `MCPTool` | MCP 服务器工具调用 |
| `LSPTool` | 语言服务器协议集成 |
| `NotebookEditTool` | 编辑 Jupyter notebook |
| `TaskCreateTool` / `TaskUpdateTool` | 任务创建与管理 |
| `SendMessageTool` | 智能体间消息传递 |
| `TeamCreateTool` / `TeamDeleteTool` | 团队智能体管理 |
| `EnterPlanModeTool` / `ExitPlanModeTool` | 切换计划模式 |
| `EnterWorktreeTool` / `ExitWorktreeTool` | Git 工作树隔离 |
| `ToolSearchTool` | 延迟工具发现 |
| `CronCreateTool` | 创建定时触发器 |
| `RemoteTriggerTool` | 远程触发 |
| `SleepTool` | 主动模式等待 |
| `SyntheticOutputTool` | 生成结构化输出 |

### 2. 命令系统（`src/commands/`）

用户通过 `/` 前缀调用的斜杠命令。

| 命令 | 描述 |
|---|---|
| `/commit` | 创建 git 提交 |
| `/review` | 代码审查 |
| `/compact` | 上下文压缩 |
| `/mcp` | MCP 服务器管理 |
| `/config` | 设置管理 |
| `/doctor` | 环境诊断 |
| `/login` / `/logout` | 身份验证 |
| `/memory` | 持久化记忆管理 |
| `/skills` | 技能管理 |
| `/tasks` | 任务管理 |
| `/vim` | 切换 Vim 模式 |
| `/diff` | 查看变更 |
| `/cost` | 查看使用成本 |
| `/theme` | 更改主题 |
| `/context` | 上下文可视化 |
| `/pr_comments` | 查看 PR 评论 |
| `/resume` | 恢复上一次会话 |
| `/share` | 分享会话 |
| `/desktop` | 切换到桌面应用 |
| `/mobile` | 切换到移动应用 |

### 3. 服务层（`src/services/`）

| 服务 | 描述 |
|---|---|
| `api/` | Anthropic API 客户端、文件 API、引导程序 |
| `mcp/` | Model Context Protocol 服务器连接与管理 |
| `oauth/` | OAuth 2.0 认证流程 |
| `lsp/` | 语言服务器协议管理器 |
| `analytics/` | 基于 GrowthBook 的功能标志与分析 |
| `plugins/` | 插件加载器 |
| `compact/` | 对话上下文压缩 |
| `policyLimits/` | 组织策略限制 |
| `remoteManagedSettings/` | 远程托管设置 |
| `extractMemories/` | 自动记忆提取 |
| `tokenEstimation.ts` | Token 数量估算 |
| `teamMemorySync/` | 团队记忆同步 |

### 4. 桥接系统（`src/bridge/`）

一个双向通信层，连接 IDE 扩展（VS Code、JetBrains）与 Claude Code CLI。

- `bridgeMain.ts` — 桥接主循环
- `bridgeMessaging.ts` — 消息协议
- `bridgePermissionCallbacks.ts` — 权限回调
- `replBridge.ts` — REPL 会话桥接
- `jwtUtils.ts` — 基于 JWT 的身份验证
- `sessionRunner.ts` — 会话执行管理

### 5. 权限系统（`src/hooks/toolPermission/`）

在每次工具调用时检查权限。根据用户批准/拒绝，或基于配置的权限模式（`default`、`plan`、`bypassPermissions`、`auto` 等）自动解析。

### 6. 功能标志

通过 Bun 的 `bun:bundle` 功能标志进行死代码消除：

```typescript
import { feature } from 'bun:bundle'

// 未激活的代码会在构建时完全被剥离
const voiceCommand = feature('VOICE_MODE')
  ? require('./commands/voice/index.js').default
  : null
```

主要标志：`PROACTIVE`、`KAIROS`、`BRIDGE_MODE`、`DAEMON`、`VOICE_MODE`、`AGENT_TRIGGERS`、`MONITOR_TOOL`

---

## 关键文件详解

### `QueryEngine.ts`（约 46K 行）

LLM API 调用的核心引擎。处理流式响应、工具调用循环、思考模式、重试逻辑和 Token 计数。

### `Tool.ts`（约 29K 行）

定义所有工具的基础类型和接口 — 输入 schema、权限模型和进度状态类型。

### `commands.ts`（约 25K 行）

管理所有斜杠命令的注册和执行。使用条件导入按环境加载不同的命令集。

### `main.tsx`

基于 Commander.js 的 CLI 解析器和 React/Ink 渲染器初始化。在启动时，它会并行执行 MDM 设置、钥匙串预取和 GrowthBook 初始化，以加快启动速度。

---

## 技术栈

| 类别 | 技术 |
|---|---|
| 运行时 | [Bun](https://bun.sh) |
| 语言 | TypeScript（严格模式） |
| 终端 UI | [React](https://react.dev) + [Ink](https://github.com/vadimdemedes/ink) |
| CLI 解析 | [Commander.js](https://github.com/tj/commander.js)（extra-typings） |
| Schema 验证 | [Zod v4](https://zod.dev) |
| 代码搜索 | [ripgrep](https://github.com/BurntSushi/ripgrep) |
| 协议 | [MCP SDK](https://modelcontextprotocol.io)、LSP |
| API | [Anthropic SDK](https://docs.anthropic.com) |
| 遥测 | OpenTelemetry + gRPC |
| 功能标志 | GrowthBook |
| 认证 | OAuth 2.0、JWT、macOS 钥匙串 |

---

## 值得注意的设计模式

### 并行预取

通过在其他模块求值之前并行预取 MDM 设置、钥匙串读取和 API 预连接，来优化启动时间。

```typescript
// main.tsx — 在其他导入之前作为副作用触发
startMdmRawRead()
startKeychainPrefetch()
```

### 懒加载

重量级模块（OpenTelemetry、gRPC、分析系统以及部分功能标志子系统）通过动态 `import()` 延迟加载，直到真正需要时才加载。

### 智能体集群

通过 `AgentTool` 生成子智能体，由 `coordinator/` 处理多智能体编排。`TeamCreateTool` 支持团队级别的并行工作。

### 技能系统

`skills/` 中定义的可复用工作流通过 `SkillTool` 执行。用户可以添加自定义技能。

### 插件架构

内置和第三方插件通过 `plugins/` 子系统加载。

---

## 研究 / 所有权免责声明

- 本仓库是一份由大学生维护的**教育和防御性安全研究存档**。
- 它存在的目的是研究源码暴露、打包失败以及现代智能体 CLI 系统的架构。
- 原始 Claude Code 源码仍归 **Anthropic** 所有。
- 本仓库**不隶属于 Anthropic，未获得其认可，也不由其维护**。
