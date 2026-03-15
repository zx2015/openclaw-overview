# OpenClaw (龙虾) 知识库

欢迎来到 **OpenClaw (龙虾)** 知识库。这是一个专注于 **本地优先 AI 代理运行时 (AI Agent Runtime)** 的技术文档库。

---

## 📖 核心文档
*   **[项目概览](./docs/overview.md)**：什么是 OpenClaw、设计哲学、应用场景与核心优势。
*   **[核心特性](./docs/features.md)**：Markdown 存储、多模态交互、全渠道集成及实战能力。
*   **[如何新建智能体](./docs/create_agent.md)**：通过 CLI、配置或克隆创建新 Agent 的详细步骤。
*   **[如何配置智能体 (Agents)](./docs/configure_agent.md)**：使用 `openclaw agents add` 进行交互式或非交互式配置。
*   **[插件管理 (Plugin Management)](./docs/plugin_management.md)**：查看、启用、禁用及卸载 OpenClaw 插件的指南。
*   **[智能体选择与切换](./docs/agent_selection.md)**：如何在不同场景下切换 Agent 及多会话管理。
*   **[模型提供商配置](./docs/model_configuration.md)**：配置魔搭 (ModelScope)、OpenAI 等模型厂商的详细指南。
*   **[System Prompt 模板参考](./docs/reference/templates/overview.md)**：`AGENTS.md`、`SOUL.md` 等核心 Prompt 模板的功能说明。

## 🛠️ 技术深度
*   **[架构高层级视图](./technical/architecture.md)**：OpenClaw 整体架构图景。
    *   **[CLI 生命周期](./technical/cli_lifecycle.md)**：启动流程、Respawn 机制。
    *   **[工具管理机制](./technical/tool_management.md)**：Profile 与 Policy 的工具分配逻辑。
    *   **[技能加载与执行机制](./technical/skill_management.md)**：解析内置、已管理与工作区技能在沙箱环境下的执行逻辑。
    *   **[Gateway 核心模块](./technical/gateway_core.md)**：`src/gateway` 的主要组成部分。
    *   **[Gateway 服务编排](./technical/gateway_orchestration.md)**：运行时、自愈机制与基础设施协调。
    *   **[沙箱机制](./technical/sandbox_mechanism.md)**：隔离执行、浏览器沙箱与安全加固逻辑。
    *   **[ACP 通讯协议](./technical/acp_protocol.md)**：标准化的 AI 交互协议实现。
    *   **[MCP 集成机制](./technical/mcp_integration.md)**：通过 `mcporter` 支持外部 MCP 服务。
    *   **[Docker 环境与 MCP 交互](./technical/mcp_docker_interaction.md)**：解析沙箱内 Agent 调用全局 MCP 工具的 Gateway 代行机制。
    *   **[心跳机制](./technical/heartbeat_mechanism.md)**：保持连接活跃与 AI 主动巡检逻辑。
    *   **[记忆管理机制](./technical/memory_management.md)**：多层级存储、动态压缩与语义检索。
    *   **[记忆归属与层级](./technical/memory_hierarchy.md)**：多代理环境下的物理隔离、逻辑共享及权属。
    *   **[多代理系统 (MAS)](./technical/multi_agent_system.md)**：子代理注册、异步协作与自动汇报逻辑。
    *   **[Agent 实战与安全](./technical/agent_capabilities.md)**：Bash 执行、子代理注册表。
    *   **[交互与上下文引擎](./technical/interaction_engine.md)**：消息流转、Prompt 构建。
    *   **[System Prompt 构建机制](./technical/system_prompt_construction.md)**：解析协议、工作区文件与动态环境参数的缝合逻辑。
    *   **[构建、测试与 QA](./technical/build_qa.md)**：自动化流水线与多端分发。
    *   **[自定义 Provider 开发指南](./technical/custom_provider_guide.md)**：扩展网关以支持新的模型提供商。

## 🧠 环境记忆与实战沉淀
*   **[环境记忆索引](../memory/index.md)**：记录特定环境的主机、URL 及配置信息。
*   **[排障经验集 (Troubleshooting)](./troubleshooting/overview.md)**：实战中的“现象-根因-解决方案”沉淀。

## 📚 外部资源
*   **[官方在线文档](https://docs.openclaw.ai)**
*   **[GitHub 代码仓库](https://github.com/openclaw/openclaw)**
*   **[搜索摘要](./resources/search_summary.md)**：2026 年初关于 OpenClaw 的搜索结果。

---

*最后更新时间：2026-03-15*
