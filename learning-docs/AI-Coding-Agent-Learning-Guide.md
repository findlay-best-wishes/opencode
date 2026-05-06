# AI Coding Agent 完整学习指南

> 从理论到实践，从 OpenCode 到 oh-my-openagent 到 Claude Code，系统性掌握 AI Coding Agent 的设计与实现。

---

## 目录

- [第一部分：AI Coding Agent 通用学习思路](#第一部分ai-coding-agent-通用学习思路)
- [第二部分：OpenCode 源码学习路径](#第二部分opencode-源码学习路径)
- [第三部分：oh-my-openagent 学习路径](#第三部分oh-my-openagent-学习路径)
- [第四部分：Claude Code 学习路径](#第四部分claude-code-学习路径)
- [第五部分：综合实践建议](#第五部分综合实践建议)

---

## 第一部分：AI Coding Agent 通用学习思路

### 1.1 核心概念模型

AI Coding Agent 的本质是：**LLM + 循环 + 工具 + 上下文管理**

```
┌─────────────────────────────────────────────────────┐
│                   Agent Loop                         │
│                                                     │
│  用户输入 → System Prompt + Context 构建             │
│      ↓                                              │
│  LLM 推理（选择下一步行动）                           │
│      ↓                                              │
│  ┌─ 返回文本 → 输出给用户                            │
│  └─ 返回 Tool Call → 执行工具 → 观察结果 → 再推理    │
│                                                     │
│  循环直到任务完成或需要用户输入                        │
└─────────────────────────────────────────────────────┘
```

### 1.2 分层架构

| 层级 | 职责 | 关键技术 |
|------|------|----------|
| **LLM 层** | 推理、决策、代码生成 | Token 管理、Context Window、Streaming |
| **Agent Loop** | 工具选择与执行循环 | ReAct 模式、Function Calling、状态管理 |
| **Tool 层** | 与外部世界交互 | 文件 I/O、Shell、LSP、Git、搜索 |
| **Context 层** | 信息压缩与检索 | 滑动窗口、摘要、RAG、知识图谱 |
| **编排层** | 多 Agent 协作 | 任务委派、并行执行、结果聚合 |
| **安全层** | 权限与沙箱 | 文件权限、命令确认、回滚机制 |
| **UI 层** | 用户交互界面 | TUI、Web、IDE 扩展、Desktop |

### 1.3 核心技术栈

#### 必学基础

| 技术 | 说明 | 学习优先级 |
|------|------|------------|
| **Function Calling / Tool Use** | LLM 如何选择并调用工具 | P0 |
| **Streaming (SSE)** | 逐 token 输出的工程实现 | P0 |
| **Context Management** | 长对话的截断、摘要策略 | P0 |
| **System Prompt 设计** | Agent 行为约束与角色定义 | P0 |
| **Provider 抽象** | 多模型统一接口 | P1 |
| **Sandboxing** | 安全执行用户代码 | P1 |

#### 进阶技术

| 技术 | 说明 | 应用场景 |
|------|------|----------|
| **MCP (Model Context Protocol)** | 标准化工具接入协议 | 扩展 Agent 能力 |
| **LSP (Language Server Protocol)** | 代码智能（跳转、诊断、重命名） | 精准代码操作 |
| **AST 解析** | 结构化代码理解 | 精确代码搜索/重写 |
| **RAG on Code** | 代码库向量化检索 | 大型代码库上下文 |
| **多 Agent 编排** | Agent 间通信与协作 | 复杂任务分解 |

### 1.4 关键设计决策

学习 AI Coding Agent 时需要思考的核心问题：

1. **编辑策略选择**
   - Whole-file rewrite：简单但费 token
   - Search & Replace：精准但依赖匹配
   - Unified Diff：紧凑但模型易出错
   - Hash-anchored Edit：稳定但需额外标注
   - AST Transform：精确但语言相关

2. **上下文工程**
   - 如何在有限 window 里塞入最相关代码？
   - 文件选择：用户指定 vs 自动发现？
   - 对话历史压缩：摘要 vs 截断 vs 分层？

3. **安全边界**
   - 哪些操作需要用户确认？
   - 如何实现文件系统权限？
   - 回滚机制（git 集成）

4. **多 Agent 架构**
   - 何时需要多个 Agent？
   - Agent 间如何通信？
   - 并行 vs 串行策略？

### 1.5 关键论文与资料

| 资料 | 类型 | 核心价值 |
|------|------|----------|
| ReAct (Yao et al. 2022) | 论文 | Agent 推理+行动的理论基础 |
| Toolformer (Schick et al. 2023) | 论文 | LLM 学习使用工具 |
| SWE-bench | 评测 | 代码 Agent 的标准 Benchmark |
| The Harness Problem (Can Bölük) | 博客 | 编辑工具设计的深度分析 |
| Anthropic Tool Use Docs | 文档 | 工业级 function calling 设计 |
| MCP Specification | 规范 | 工具协议标准化 |
| TerminalBench | 评测 | Agent 终端能力评估 |

---

## 第二部分：OpenCode 源码学习路径

> 仓库地址：https://github.com/anomalyco/opencode
> 
> 技术栈：TypeScript + Bun + SolidJS + Turbo Monorepo
> 
> 规模：155k Stars, 880+ Contributors, 12,287 Commits

### 2.1 项目架构总览

```
anomalyco/opencode
├── packages/
│   ├── opencode/          ← 🧠 核心：Agent 逻辑 + Server + CLI + TUI
│   │   └── src/
│   │       ├── index.ts          ← CLI 入口
│   │       ├── cli/cmd/tui/      ← TUI (SolidJS + opentui)
│   │       └── server/server.ts  ← API Server
│   ├── app/               ← Web UI (SolidJS 共享组件)
│   ├── desktop/           ← Electron 桌面端
│   ├── plugin/            ← @opencode-ai/plugin 插件系统
│   ├── sdk/               ← 客户端 SDK
│   ├── console/           ← 控制台管理界面
│   ├── ui/                ← 共享 UI 组件库
│   ├── containers/        ← 沙箱容器
│   ├── docs/              ← 文档站 (Mintlify)
│   └── extensions/zed/    ← Zed 编辑器扩展
├── specs/                 ← 设计文档/规格说明
├── infra/                 ← 基础设施 (SST)
├── sdks/vscode/           ← VS Code 扩展
├── sst.config.ts          ← SST 部署配置
└── turbo.json             ← Turborepo 配置
```

### 2.2 核心设计特点

| 特点 | 说明 |
|------|------|
| **Client/Server 分离** | Server 独立运行，TUI/Web/Desktop/Mobile 均为客户端 |
| **Provider-Agnostic** | 不绑定单一 LLM 供应商（Claude/OpenAI/Google/本地模型） |
| **Built-in Agent 系统** | build（全权限）+ plan（只读分析）+ general（子任务） |
| **可选 LSP 集成** | 内置 LSP 支持，提供代码智能 |
| **SolidJS TUI** | 用 opentui 在终端实现声明式 UI |
| **Plugin 系统** | 第三方可通过 @opencode-ai/plugin 扩展功能 |

### 2.3 Client/Server 架构详解

```
┌──────────┐                    ┌──────────────────┐
│   TUI    │◄──── HTTP/WS ────►│                  │
├──────────┤                    │                  │
│  Web UI  │◄──── HTTP/WS ────►│   OpenCode       │
├──────────┤                    │   Server         │
│ Desktop  │◄──── HTTP/WS ────►│   (default:4096) │
├──────────┤                    │                  │
│  Mobile  │◄──── HTTP/WS ────►│                  │
├──────────┤                    │                  │
│ VS Code  │◄──── HTTP/WS ────►│                  │
└──────────┘                    └──────────────────┘
```

**设计动机**：
- 允许远程驱动（手机控制电脑上的 Agent）
- TUI 只是众多客户端之一
- 服务端可独立部署（headless 模式）

### 2.4 逐步阅读路径

#### Phase 1：理解入口与启动流程（1-2天）

```
目标：理解 CLI 解析、Server 启动、TUI 渲染的完整链路

阅读顺序：
1. packages/opencode/src/index.ts        → CLI 入口
2. packages/opencode/package.json        → 依赖与脚本
3. 找到 server 启动逻辑                   → HTTP/WS 服务初始化
4. packages/opencode/src/cli/cmd/tui/    → TUI 组件树
```

#### Phase 2：理解 Agent 定义与配置（2-3天）

```
目标：理解 build/plan/general agent 的定义、权限差异、System Prompt

关注：
- Agent 配置结构（model、permissions、system prompt）
- build agent 为何能执行任何操作
- plan agent 如何限制为只读
- general subagent 的调用机制（@general 语法）
```

#### Phase 3：跟踪一次完整 Tool Use 循环（3-5天）

```
目标：断点跟踪从用户输入到 LLM 响应到工具执行的完整链路

跟踪路径：
用户输入文本
  → 构建 messages 数组（system + history + user）
  → 调用 LLM API（streaming）
  → 解析响应（text vs tool_use）
  → 如果是 tool_use：
      → 查找 tool 注册表
      → 执行 tool（传入参数）
      → 收集结果
      → 构建 tool_result message
      → 再次调用 LLM
  → 如果是 text：
      → 输出给用户
      → 循环结束
```

#### Phase 4：理解 Tool 注册与实现（3-5天）

```
目标：理解每个 tool 的 schema 定义、权限控制、执行逻辑

重点 Tool：
- file_read / file_write   → 文件操作 + 权限检查
- bash / shell             → 命令执行 + 安全约束
- grep / glob              → 搜索工具
- lsp_*                    → LSP 集成工具
- git_*                    → Git 操作
```

#### Phase 5：理解 Provider 抽象层（2-3天）

```
目标：理解如何统一 Claude/OpenAI/Google 等不同 API

关注：
- Provider 接口定义
- 消息格式转换（各家格式不同）
- Tool Use 协议差异处理
- 流式输出的统一处理
- 模型能力声明（context length、tool support）
- 参考：https://github.com/anomalyco/models.dev
```

#### Phase 6：理解 Context 管理（2-3天）

```
目标：理解对话过长时的处理策略

关注：
- Token 计数逻辑
- 对话截断/摘要策略
- AGENTS.md 自动注入机制
- 文件内容如何按需加载
```

#### Phase 7：理解 Server API 与 SDK（2-3天）

```
目标：理解 API 路由设计、SDK 如何消费

关注：
- packages/opencode/src/server/server.ts → API 路由
- packages/sdk/ → 客户端 SDK 实现
- WebSocket 推送流式响应
- Session 管理（创建/恢复/销毁）
- script/generate.ts → SDK 自动生成
```

#### Phase 8：理解 TUI 渲染层（2-3天）

```
目标：理解 SolidJS + opentui 如何在终端渲染

关注：
- opentui 的组件模型
- 流式数据如何驱动 UI 更新
- 终端 input 处理（键盘、Tab 切换 Agent）
- 响应式更新机制（fine-grained reactivity）
```

### 2.5 本地开发环境

```bash
# 克隆
git clone https://github.com/anomalyco/opencode
cd opencode

# 安装依赖
bun install

# 启动 TUI 开发模式（默认在 packages/opencode 目录运行）
bun dev

# 在指定目录运行
bun dev /path/to/your/project

# 启动 headless server（方便调试）
bun dev serve              # 默认 port 4096
bun dev serve --port 8080  # 自定义端口

# 启动 Web UI 开发
bun run --cwd packages/app dev

# 构建独立二进制
./packages/opencode/script/build.ts --single
```

### 2.6 调试技巧

```bash
# Bun 调试模式
bun run --inspect=ws://localhost:6499/ dev

# 分离调试 server 和 TUI
# 1. 调试 server
bun run --inspect=ws://localhost:6499/ --cwd packages/opencode ./src/index.ts serve --port 4096
# 2. attach TUI
opencode attach http://localhost:4096

# 环境变量方式（省去每次输入）
export BUN_OPTIONS=--inspect=ws://localhost:6499/
```

### 2.7 设计决策深度思考

| 决策 | 原因 |
|------|------|
| 为什么 Client/Server 分离？ | 解耦 UI 与核心逻辑，支持远程驱动 |
| 为什么 Provider-agnostic？ | 模型迭代快，不绑定单一厂商 |
| 为什么 build vs plan？ | 只读分析 vs 全权执行的权限边界清晰 |
| 为什么用 Bun？ | 启动快、原生 TS、单二进制打包 |
| 为什么 TUI 用 SolidJS？ | Fine-grained reactivity 适合终端高效更新 |
| 为什么有 Plugin 系统？ | 允许社区扩展而不 fork 核心 |

---

## 第三部分：oh-my-openagent 学习路径

> 仓库地址：https://github.com/code-yeongyu/oh-my-openagent
> 
> 定位：OpenCode 的 "Agent Harness"（增强层/插件），而非独立产品
> 
> 技术栈：TypeScript + Bun
> 
> 规模：56k Stars, 4.6k Forks, 5,659 Commits

### 3.1 项目定位

oh-my-openagent 不是另一个 AI Coding Agent，而是 **OpenCode 的增强插件**，类似于：
- OpenCode = Debian/Arch（底层系统）
- oh-my-openagent = Ubuntu/Omarchy（开箱即用的增强层）

它在 OpenCode 之上提供：
- 预配置的多 Agent 编排系统
- 优化的 System Prompt（Discipline Agents）
- Hash-anchored 编辑工具
- 内置 MCP 集成
- 一键 `ultrawork` 启动

### 3.2 项目结构

```
code-yeongyu/oh-my-openagent
├── src/                   ← 🧠 核心插件逻辑
├── packages/              ← 子包
├── bin/                   ← CLI 入口
├── docs/
│   ├── guide/
│   │   ├── installation.md   ← 安装指南
│   │   ├── overview.md       ← 功能总览
│   │   └── orchestration.md  ← Agent 编排指南
│   ├── reference/
│   │   ├── features.md       ← 完整功能文档
│   │   └── configuration.md  ← 配置参考
│   └── manifesto.md          ← Ultrawork 宣言
├── signatures/            ← 签名/验证
├── tests/hashline/        ← Hash-anchored edit 测试
├── drafts/gpt-5-5/        ← 模型适配草案
├── AGENTS.md              ← Agent 定义规范
└── .opencode/             ← OpenCode 插件配置
```

### 3.3 核心特性学习

#### 3.3.1 Discipline Agents（纪律 Agent 系统）

```
┌─────────────────────────────────────────────────┐
│               Sisyphus (主编排者)                  │
│  Model: claude-opus-4-7 / kimi-k2.5 / glm-5    │
│  角色: 计划、委派、并行执行、驱动至完成             │
├─────────────────────────────────────────────────┤
│                                                  │
│  ┌─────────────┐  ┌─────────────┐               │
│  │ Hephaestus  │  │ Prometheus  │               │
│  │ (GPT-5.4)   │  │ (Opus/Kimi) │               │
│  │ 自主深度工作  │  │  战略规划    │               │
│  └─────────────┘  └─────────────┘               │
│                                                  │
│  ┌─────────────┐  ┌─────────────┐               │
│  │   Oracle    │  │  Librarian  │               │
│  │ 架构/调试咨询 │  │  文档/代码搜索│               │
│  └─────────────┘  └─────────────┘               │
│                                                  │
│  ┌─────────────┐  ┌─────────────┐               │
│  │   Explore   │  │   Metis     │               │
│  │ 代码库快速搜索│  │  计划评审    │               │
│  └─────────────┘  └─────────────┘               │
└─────────────────────────────────────────────────┘
```

**学习重点**：
- 每个 Agent 的 System Prompt 如何设计
- Agent 之间如何委派任务
- 模型选择策略（不同 Agent 用不同模型的原因）
- Category 系统如何路由到正确模型

#### 3.3.2 Hash-Anchored Edit（核心创新）

传统编辑的问题：模型需要精确复制原始内容来定位修改位置，经常出错。

Hash-anchored 方案：

```
# 读取文件时，每行附带内容哈希
11#VK| function hello() {
22#XJ|   return "world";
33#MB| }

# 编辑时引用哈希标识
编辑指令：将 22#XJ 的内容改为 return "hello world";

# 如果文件已变更，哈希不匹配 → 编辑被拒绝
```

**学习重点**：
- `tests/hashline/` 中的测试用例
- 哈希生成算法
- 冲突检测逻辑
- 与传统 search & replace 的对比（参考 The Harness Problem 博文）

#### 3.3.3 Category 路由系统

```
用户请求 → Sisyphus 判断任务类型 → 选择 Category → 自动路由到最佳模型

Category 映射：
┌────────────────────┬──────────────────────┬─────────────┐
│ Category           │ 适用场景              │ 默认模型     │
├────────────────────┼──────────────────────┼─────────────┤
│ visual-engineering │ 前端/UI/UX/设计       │ 视觉优化模型  │
│ deep               │ 自主研究+端到端执行    │ GPT-5.4     │
│ quick              │ 单文件修改/typo        │ 轻量模型     │
│ ultrabrain         │ 复杂逻辑/架构决策      │ GPT-5.4 xhigh │
│ writing            │ 文档/技术写作          │ 写作优化模型  │
└────────────────────┴──────────────────────┴─────────────┘
```

#### 3.3.4 IntentGate（意图门控）

在执行任何操作之前，先分析用户的真实意图：

```
用户说 "look into X"
  → IntentGate 分析：这是研究请求，不是实现请求
  → 路由到 Explore/Librarian，而不是开始编码

用户说 "add dark mode"
  → IntentGate 分析：这是明确的实现请求
  → 路由到 Prometheus 规划 → Sisyphus 执行
```

#### 3.3.5 内置 MCP 集成

| MCP | 功能 | 用途 |
|-----|------|------|
| Exa (websearch) | 网络搜索 | 查找最新文档、解决方案 |
| Context7 | 官方文档查询 | 获取库/框架的准确 API 文档 |
| Grep.app | GitHub 代码搜索 | 查找真实世界的代码示例 |

#### 3.3.6 Ralph Loop / ulw-loop

自引用循环机制：Agent 不断自检任务完成度，不停直到 100% 完成。

```
启动 ultrawork
  → 执行任务
  → 自检：完成了吗？
  → 没有 → 继续执行
  → 自检：完成了吗？
  → 是 → 结束
```

配合 **Todo Enforcer**：如果 Agent 空闲，系统自动拉回继续工作。

### 3.4 学习路径

#### Phase 1：理解插件加载机制（1天）

```
关注：
- oh-my-openagent 如何被 OpenCode 加载
- opencode.json 中的 plugin 配置
- 插件如何注入/覆盖 Agent 定义
```

#### Phase 2：分析 Agent System Prompts（2-3天）

```
关注：
- AGENTS.md 中的完整 Agent 定义
- Sisyphus 的编排逻辑（最核心）
- 各 Agent 的行为约束（MUST DO / MUST NOT DO）
- 委派 prompt 的结构要求
```

#### Phase 3：理解 Hash-anchored Edit（2天）

```
关注：
- src/ 中 hashline 相关代码
- tests/hashline/ 测试用例
- 哈希生成、验证、冲突处理
```

#### Phase 4：理解编排与路由（2-3天）

```
关注：
- Category 系统实现
- IntentGate 意图分析逻辑
- Background Agent 的并发控制
- 任务委派的 prompt 构建
```

#### Phase 5：理解 Skill 系统（1-2天）

```
关注：
- Skill 定义格式（SKILL.md）
- Skill-Embedded MCP 的加载/卸载
- 内置 Skill（playwright, git-master, frontend-ui-ux）
```

### 3.5 安装与体验

```bash
# 方式1：让 AI Agent 安装（推荐）
# 把以下 prompt 粘贴给你的 Agent：
# Install and configure oh-my-openagent by following the instructions here:
# https://raw.githubusercontent.com/code-yeongyu/oh-my-openagent/refs/heads/dev/docs/guide/installation.md

# 方式2：手动安装
npm install -g oh-my-opencode  # npm 包名仍为 oh-my-opencode

# 验证
opencode  # 启动后输入 ultrawork 或 ulw
```

### 3.6 核心思想总结

| 理念 | 具体体现 |
|------|----------|
| **模型各有所长** | 不同 Agent 用不同模型，发挥各自优势 |
| **纪律胜过自由** | Agent 有严格的行为约束，不随意发挥 |
| **编辑工具决定上限** | Hash-anchored edit 解决 "The Harness Problem" |
| **不停直到完成** | Ralph Loop + Todo Enforcer 确保任务完成 |
| **意图先行** | IntentGate 先理解再行动，避免误解 |
| **并行为王** | Background Agents 并发执行，节省时间 |

---

## 第四部分：Claude Code 学习路径

> 官方文档：https://docs.claude.com / https://code.claude.com/docs
> 
> 定位：Anthropic 官方 AI Coding Agent（闭源）
> 
> 特点：深度绑定 Claude 模型，工业级稳定性

### 4.1 产品架构

```
Claude Code 生态
├── Terminal CLI        ← 核心，命令行工具
├── VS Code Extension  ← IDE 集成
├── JetBrains Plugin   ← IDE 集成
├── Desktop App        ← 独立桌面应用
├── Web (claude.ai/code) ← 浏览器版本
└── iOS App            ← 移动端
```

### 4.2 核心概念

#### 4.2.1 Tool 系统

Claude Code 内置的工具集：

| Tool | 功能 |
|------|------|
| Read | 读取文件内容 |
| Write | 写入/覆盖文件 |
| Edit (Search & Replace) | 精确文本替换 |
| Bash | 执行 shell 命令 |
| Glob | 文件模式匹配搜索 |
| Grep | 内容正则搜索 |
| LSP Tools | 代码智能（定义跳转、引用查找等） |
| MCP Tools | 外部工具集成 |

#### 4.2.2 CLAUDE.md（项目记忆）

```markdown
# CLAUDE.md - 项目根目录

## 项目概述
这是一个 Next.js 电商平台...

## 编码规范
- 使用 TypeScript strict mode
- 组件使用 function 声明而非 arrow function
- 测试文件使用 .test.ts 后缀

## 常用命令
- `npm run dev` - 启动开发服务器
- `npm run test` - 运行测试
- `npm run build` - 构建项目

## 架构决策
- 使用 Server Components 优先
- 数据库使用 Drizzle ORM
```

**学习要点**：
- CLAUDE.md 在每次会话开始时被读取
- 可以在子目录放置局部 CLAUDE.md
- Auto Memory：Claude 自动保存学习到的信息

#### 4.2.3 Permissions 模型

```
权限级别：
┌─────────────────────────────────────────┐
│ Allow All    → 所有操作自动批准          │
├─────────────────────────────────────────┤
│ Ask          → 危险操作需要用户确认      │
├─────────────────────────────────────────┤
│ Deny         → 拒绝特定操作             │
└─────────────────────────────────────────┘

文件级别：
- 读取：通常自动允许
- 写入：首次需要 Trust Folder
- 执行：bash 命令需要确认（可配置自动批准）
```

#### 4.2.4 Hooks 系统

```javascript
// .claude/hooks/pre-edit.sh - 文件编辑前触发
#!/bin/bash
# 自动备份
cp "$CLAUDE_FILE_PATH" "$CLAUDE_FILE_PATH.bak"

// .claude/hooks/post-edit.sh - 文件编辑后触发
#!/bin/bash
# 自动格式化
prettier --write "$CLAUDE_FILE_PATH"
```

#### 4.2.5 Skills（可复用工作流）

```markdown
# /deploy-staging
## Description
Deploy current branch to staging environment

## Steps
1. Run tests: `npm run test`
2. Build: `npm run build`
3. Deploy: `npm run deploy:staging`
4. Verify: Check staging URL responds with 200
```

#### 4.2.6 Sub-Agents（多 Agent 协作）

```
Lead Agent（主 Agent）
├── Sub-Agent 1: "重构 auth 模块"
├── Sub-Agent 2: "更新相关测试"
└── Sub-Agent 3: "更新文档"

各 Sub-Agent 并行工作，主 Agent 协调合并结果
```

#### 4.2.7 MCP (Model Context Protocol)

```bash
# 添加 MCP Server
claude mcp add slack -- npx -y @anthropic-ai/slack-mcp
claude mcp add jira -- npx -y @anthropic-ai/jira-mcp

# 列出已注册的 MCP
claude mcp list
```

### 4.3 学习路径

#### Phase 1：日常使用掌握（1周）

```
目标：熟练使用 Claude Code 完成日常开发任务

练习：
1. 用 claude 阅读并理解一个开源项目
2. 让 claude 添加一个完整功能（含测试）
3. 使用 Plan mode 规划再执行
4. 使用 /undo 和 /redo 管理变更
5. 让 claude 创建 commit 和 PR
```

#### Phase 2：CLAUDE.md 与上下文工程（3天）

```
目标：通过 CLAUDE.md 优化 Agent 行为

练习：
1. 为你的项目编写完善的 CLAUDE.md
2. 在子目录放置局部规则
3. 观察 Auto Memory 的行为
4. 对比有/无 CLAUDE.md 的输出质量
```

#### Phase 3：Hooks 与 Skills（3天）

```
目标：自动化重复工作流

练习：
1. 创建 post-edit hook 自动格式化
2. 创建 pre-commit hook 运行 lint
3. 编写自定义 Skill（如 /review-pr）
4. 创建团队共享的 Skills
```

#### Phase 4：MCP 集成（2-3天）

```
目标：扩展 Agent 能力边界

练习：
1. 集成 Slack MCP（读取频道消息）
2. 集成自定义 MCP Server
3. 理解 MCP 协议（JSON-RPC over stdio）
4. 阅读 MCP Specification
```

#### Phase 5：Sub-Agents 与 Agent SDK（3-5天）

```
目标：理解多 Agent 编排

练习：
1. 使用 Sub-Agents 并行处理任务
2. 阅读 Agent SDK 文档
3. 构建自定义 Agent（使用 SDK）
4. 分析 Agent 间通信机制
```

#### Phase 6：CI/CD 集成（2-3天）

```
目标：在自动化流程中使用 Claude Code

练习：
1. 配置 GitHub Actions + Claude Code
2. 自动 PR Review
3. 自动 Issue Triage
4. Scheduled Routines（定时任务）
```

### 4.4 与 OpenCode 的对比分析

| 维度 | Claude Code | OpenCode |
|------|-------------|----------|
| 开源 | 否（闭源） | 是（MIT） |
| 模型支持 | Claude 为主（支持第三方） | 完全 Provider-agnostic |
| 架构 | 单体 CLI | Client/Server 分离 |
| 扩展性 | Hooks + Skills + MCP | Plugin + 完整 SDK |
| LSP | 支持 | 内置可选 LSP |
| TUI 技术 | 未公开 | SolidJS + opentui |
| 多 Agent | Sub-Agents | build + plan + general |
| 商业模式 | 订阅制（$20-200/月） | 自带 API Key |
| 学习价值 | 产品设计、最佳实践 | 底层实现、架构设计 |

### 4.5 Claude Code 的学习价值

虽然 Claude Code 闭源，但可以从以下角度学习：

1. **产品设计**：如何设计用户友好的 Agent 交互
2. **CLAUDE.md 规范**：如何设计项目级上下文注入
3. **权限模型**：如何平衡自动化与安全
4. **MCP 协议**：标准化工具接入的设计思路
5. **Hooks 系统**：如何让用户自定义 Agent 行为
6. **多 Agent 协作**：Sub-Agents 的设计模式

---

## 第五部分：综合实践建议

### 5.1 推荐学习顺序

```
Week 1-2: 基础体验
├── 安装并日常使用 Claude Code / OpenCode
├── 理解 Tool Use 循环（通过使用感受）
└── 编写 CLAUDE.md / AGENTS.md

Week 3-4: OpenCode 源码
├── Clone opencode，跑起 bun dev
├── 跟踪 Agent Loop 核心流程
├── 理解 Tool 注册与执行
└── 理解 Provider 抽象

Week 5-6: oh-my-openagent 增强层
├── 安装并体验 ultrawork
├── 分析 Discipline Agents 的 System Prompt
├── 理解 Hash-anchored Edit
├── 理解 Category 路由系统
└── 理解多 Agent 编排

Week 7-8: 造轮子
├── 实现最小 Agent Loop（选任意语言）
│   ├── 接入一个 LLM API
│   ├── 实现 3 个 Tool：read_file, write_file, bash
│   └── 实现 ReAct 循环
├── 实现简单的 MCP Client
└── 实现基础的 Context 管理

Week 9-10: 进阶
├── 实现 Provider 抽象（支持 2+ 模型）
├── 实现 Agent 编排（主 Agent + 子 Agent）
├── 实现编辑策略（search & replace 或 hash-anchored）
├── 接入 LSP
└── 用 SWE-bench 或自建 Benchmark 测试
```

### 5.2 关键源码对照表

| 概念 | OpenCode 中找 | oh-my-openagent 中找 |
|------|---------------|---------------------|
| Agent Loop | `packages/opencode/src/` | 插件如何覆盖循环行为 |
| Tool 定义 | tool 注册模块 | Hash-anchored edit 实现 |
| System Prompt | Agent 配置文件 | `AGENTS.md` + src/ 中的 prompt |
| Provider | provider 抽象层 | Category → Model 映射 |
| Context 管理 | 消息构建逻辑 | IntentGate + 上下文裁剪 |
| 多 Agent | general subagent | Sisyphus 编排 + Background Agents |
| Plugin/扩展 | `packages/plugin/` | 整个项目就是一个 Plugin |

### 5.3 推荐对照阅读的开源项目

| 项目 | 语言 | 特点 | 对照学习 |
|------|------|------|----------|
| [Aider](https://github.com/paul-gauthier/aider) | Python | 成熟的 diff 编辑模式 | 编辑策略对比 |
| [Cline](https://github.com/cline/cline) | TypeScript | 完整的 VS Code Agent | IDE 集成模式 |
| [SWE-agent](https://github.com/princeton-nlp/SWE-agent) | Python | 学术研究级 | 评估方法论 |
| [Continue](https://github.com/continuedev/continue) | TypeScript | IDE 插件架构 | 编辑器集成 |
| [Goose](https://github.com/block/goose) | Rust+Python | 插件系统设计 | 扩展性架构 |
| [oh-my-pi](https://github.com/can1357/oh-my-pi) | - | Hash-anchored 原创 | 编辑工具设计 |

### 5.4 常见陷阱与建议

| 陷阱 | 建议 |
|------|------|
| 只看不做 | 尽早动手实现最小版本 |
| 纠结完美架构 | 先跑通再优化 |
| 忽略 Context 管理 | 这是最影响效果的部分 |
| 过度关注 UI | 核心价值在 Agent Loop，不在界面 |
| 忽视安全 | 从一开始就考虑权限边界 |
| 模型依赖 | 保持 Provider-agnostic 思维 |
| 忽略评估 | 没有 benchmark 就无法量化改进 |

### 5.5 进一步探索方向

```
方向1: 垂直场景 Agent
├── 专注某一技术栈（如 React Agent、Rust Agent）
├── 深度集成特定工具链
└── 针对性优化 System Prompt

方向2: Agent 评估与优化
├── 构建自动化测试套件
├── 分析失败案例改进 prompt
├── Token 效率优化
└── 延迟优化

方向3: 多模态扩展
├── 图像理解（UI 截图 → 代码）
├── 语音交互
└── 视频分析（录屏 → Bug 复现）

方向4: 团队协作
├── 多人共享 Agent Session
├── 权限与审计
├── 知识库集成（团队经验沉淀）
└── CI/CD 深度整合
```

---

## 附录

### A. 快速参考链接

| 资源 | 链接 |
|------|------|
| OpenCode 仓库 | https://github.com/anomalyco/opencode |
| OpenCode 文档 | https://opencode.ai/docs |
| oh-my-openagent 仓库 | https://github.com/code-yeongyu/oh-my-openagent |
| Claude Code 文档 | https://docs.claude.com |
| MCP 规范 | https://modelcontextprotocol.io |
| models.dev | https://github.com/anomalyco/models.dev |
| opentui | https://github.com/sst/opentui |
| The Harness Problem | https://blog.can.ac/2026/02/12/the-harness-problem/ |
| SWE-bench | https://swebench.com |
| TerminalBench | https://factory.ai/news/terminal-bench |

### B. 术语表

| 术语 | 含义 |
|------|------|
| Agent Loop | LLM 推理 → 工具调用 → 观察结果的循环 |
| ReAct | Reasoning + Acting，Agent 的核心范式 |
| Tool Use / Function Calling | LLM 调用外部工具的能力 |
| MCP | Model Context Protocol，工具接入标准协议 |
| LSP | Language Server Protocol，代码智能协议 |
| AST | Abstract Syntax Tree，抽象语法树 |
| RAG | Retrieval Augmented Generation，检索增强生成 |
| Context Window | LLM 单次能处理的最大 token 数 |
| System Prompt | 定义 Agent 行为的系统级指令 |
| Harness | Agent 的运行框架/外壳 |
| Hash-anchored Edit | 用内容哈希锚定行位置的编辑方式 |
| Category Routing | 按任务类型路由到最佳模型 |
| IntentGate | 执行前的意图分析门控 |
| Provider | LLM API 供应商（Anthropic/OpenAI/Google 等） |
| TUI | Terminal User Interface，终端用户界面 |

---

*文档生成时间：2026-05-06*
*基于 OpenCode v1.14.39, oh-my-openagent latest, Claude Code v2.1.123*
