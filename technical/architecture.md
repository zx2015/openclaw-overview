# OpenClaw 架构高层级视图 (High-Level View)

OpenClaw 采用高度模块化的 **Monorepo** 结构，旨在构建一个本地优先、隐私受控的个人 AI 编排平台。

---

## 1. 核心技术栈 (Technical Stack)
*   **运行时**：Node.js >= 22.12.0 (ESM)。
*   **开发语言**：TypeScript。
*   **核心库**：ACP SDK, Playwright, sqlite-vec, Pi Agent Core。
*   **接入模型**：Anthropic, OpenAI, Gemini, Ollama, 豆包, Kimi 等。

## 2. 三层逻辑模型 (Logical Layers)
OpenClaw 的架构由三个关键层次组成：
*   **网关层 (Gateway Layer)**：连接用户与渠道（如 WhatsApp, Slack）。
*   **推理层 (Reasoning Layer)**：AI 的决策大脑，采用 ReAct 循环。
*   **执行层 (Execution Layer)**：安全调用系统工具、Shell 或浏览器。

## 3. 存储模型：Markdown-as-Database
OpenClaw 将人类可读性置于首位：
*   **`~/clawd/`**：所有的会话记录、记忆、技能配置均以 **Markdown** 文件形式存储。
*   **版本控制**：原生支持通过 Git 备份 AI 的所有状态。

---

## 📂 详细模块解析
为了更深入地理解系统，请参阅以下专项架构文档：

### ⚙️ 启动与执行控制
*   **[CLI 生命周期与执行逻辑](./cli_lifecycle.md)**：解析从 `entry.ts` 到 `run-main.ts` 的启动流程。
*   **[工具列表来源与管理机制](./tool_management.md)**：解析 Profile 与 Policy 如何动态分配 AI 的工具权限。

### 🏗️ 系统核心实现
*   **[Gateway 核心模块](./gateway_core.md)**：深入了解 `src/gateway` 的各子模块组成。
*   **[Gateway 服务编排](./gateway_orchestration.md)**：解析 `server.impl.ts` 如何协调运行时、基础设施、插件渠道及动态自愈。
*   **[记忆管理机制](./memory_management.md)**：解析多层级存储、动态压缩与 RAG 语义检索逻辑。
*   **[记忆归属与层级](./memory_hierarchy.md)**：解析多代理环境下的物理隔离、逻辑共享及权属关系。
*   **[多代理系统 (MAS)](./multi_agent_system.md)**：解析子代理的注册、异步协作、持久化追踪与结果自动通报机制。
*   **[Agent 实战能力与安全](./agent_capabilities.md)**：解析 `src/agents` 的 Bash 执行、多代理协作与沙箱机制。

### 🔄 交互与上下文引擎
*   **[消息交互与上下文处理](./interaction_engine.md)**：解析对话流状态管理、增量推送频率限制及上下文序列化引擎。
*   **[System Prompt 构建机制](./system_prompt_construction.md)**：解析 OpenClaw 如何动态拼接核心协议、工作区文件与实时环境参数。

### 🚀 工程化与分发
*   **[构建、测试与质量保证](./build_qa.md)**：了解详尽的自动化测试体系、多平台构建脚本以及部署架构。

---

## 4. 目录结构概览 (`src/`)
*   `acp/`: 通信协议实现。
*   `agents/`: 核心 AI 逻辑（Bash, 记忆, 压缩）。
*   `channels/`: 各渠道适配器。
*   `gateway/`: 控制平面、身份验证、服务器实现。
*   `cli/`: 命令行界面具体逻辑。
*   `plugin-sdk/`: 第三方扩展 SDK。
*   `providers/`: 模型厂商适配器。
*   `wizard/`: 入职引导逻辑。
