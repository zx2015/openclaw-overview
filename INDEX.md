# OpenClaw (龙虾) 知识库

欢迎来到 **OpenClaw (龙虾)** 知识库。这是一个专注于 **本地优先 AI 代理运行时 (AI Agent Runtime)** 的技术文档库。

---

## 📖 核心文档
*   **[项目概览](./docs/overview.md)**：什么是 OpenClaw、设计哲学、应用场景与核心优势。
*   **[核心特性](./docs/features.md)**：Markdown 存储、多模态交互、全渠道集成及实战能力。
*   **[如何新建智能体](./docs/create_agent.md)**：通过 CLI、配置或克隆创建新 Agent 的详细步骤。
*   **[智能体选择与切换](./docs/agent_selection.md)**：如何在不同场景下切换 Agent 及多会话管理。
*   **[System Prompt 模板参考](./docs/reference/templates/overview.md)**：`AGENTS.md`、`SOUL.md` 等核心 Prompt 模板的功能说明。

## 🛠️ 技术深度
*   **[架构高层级视图](./technical/architecture.md)**：OpenClaw 整体架构图景。
    *   **[CLI 生命周期](./technical/cli_lifecycle.md)**：启动流程、Respawn 机制。
    *   **[工具管理机制](./technical/tool_management.md)**：Profile 与 Policy 的工具分配逻辑。
    *   **[Gateway 核心模块](./technical/gateway_core.md)**：`src/gateway` 的主要组成部分。
    *   **[Gateway 服务编排](./technical/gateway_orchestration.md)**：运行时、自愈机制与基础设施协调。
    *   **[MCP 集成机制](./technical/mcp_integration.md)**：通过 `mcporter` 支持外部 MCP 服务。
    *   **[心跳机制](./technical/heartbeat_mechanism.md)**：保持连接活跃与 AI 主动巡检逻辑。
    *   **[记忆管理机制](./technical/memory_management.md)**：多层级存储、动态压缩与语义检索。
    *   **[记忆归属与层级](./technical/memory_hierarchy.md)**：多代理环境下的物理隔离、逻辑共享及权属。
    *   **[多代理系统 (MAS)](./technical/multi_agent_system.md)**：子代理注册、异步协作与自动汇报逻辑。
    *   **[Agent 实战与安全](./technical/agent_capabilities.md)**：Bash 执行、子代理注册表。
    *   **[交互与上下文引擎](./technical/interaction_engine.md)**：消息流转、Prompt 构建。
    *   **[System Prompt 构建机制](./technical/system_prompt_construction.md)**：解析协议、工作区文件与动态环境参数的缝合逻辑。
    *   **[构建、测试与 QA](./technical/build_qa.md)**：自动化流水线与多端分发。

## 📚 外部资源
*   **[搜索摘要](./resources/search_summary.md)**：2026 年初关于 OpenClaw 的搜索结果。

---

*最后更新时间：2026-03-10*
