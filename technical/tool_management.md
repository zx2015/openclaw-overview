# OpenClaw 工具列表来源与管理机制

OpenClaw 的工具系统采用了“目录化定义、策略化过滤、动态化注入”的管理模式。AI 并非拥有全部权限，其能力边界由当前的 Session 环境和配置策略实时决定。

## 1. 工具列表的来源 (Tool Sources)

OpenClaw 的工具集由以下三大部分构成：

### 核心内置工具 (Core Tools)
*   **定义位置**：`src/agents/tool-catalog.ts` 中的 `CORE_TOOL_DEFINITIONS` 常量。
*   **内容分类**：
    *   **fs (文件系统)**：`read`, `write`, `edit`, `process` 等。
    *   **runtime (运行时)**：`exec` (执行 Shell 命令)。
    *   **web**：`web_search` (网页搜索)。
    *   **memory**：`memory_search` (长期记忆检索)。
    *   **sessions**：`sessions_spawn`, `sessions_list` (多会话管理)。

### 插件与技能工具 (Plugin/Skill Tools)
*   **加载机制**：由 `src/gateway/server-plugins.ts` 在网关启动时动态加载。更多关于技能 (Skills) 的加载逻辑请参考 **[技能加载与执行机制](./skill_management.md)**。
*   **内容示例**：
    *   通讯插件：`whatsapp`, `telegram`, `slack` 专用接口。
    *   三方服务：`github`, `jira`, `notion`, `weather` 等 Skills 提供的工具。

### 子代理与 MCP 工具 (Subagent/MCP Tools)
*   **桥接模式**：通过子代理协议或 **Model Context Protocol (MCP)** 接入的外部工具集，实现跨进程或跨服务的工具调用。

## 2. 工具的过滤与分配机制 (Filtering & Policy)

系统并不会将所有工具都暴露给 AI，而是通过以下两层过滤机制进行动态分配：

### Profile (配置方案)
OpenClaw 预设了四种典型的能力方案，每个核心工具在 `tool-catalog.ts` 中都标记了其归属：
*   **minimal**：仅包含最基础的交互工具。
*   **coding**：侧重文件操作与命令执行（包含 `read`, `write`, `exec` 等）。
*   **messaging**：侧重通讯与会话管理（包含 `sessions_list`, `whatsapp` 等）。
*   **full**：开启全量工具权限。

### Policy (用户策略)
用户可以通过配置文件（`~/.openclaw/config.json` 或 `.env`）进行精细化控制：
*   **Allow/Deny 列表**：显式允许或禁止特定的工具名称（如禁止 `exec` 以增强安全性）。
*   **Session 感知**：系统会根据访问来源（如 Web 端的权限通常高于 WhatsApp 端）自动调整可用工具列表。

## 3. 工具列表的记录与注入

*   **配置层**：存储在用户的 `~/.openclaw/config.json` 中（如 `agents.defaults.tools.profile` 或 `agents.defaults.tools.allow`）。
*   **运行时计算**：在 `src/agents/cli-runner/helpers.ts` 中，系统会通过 `params.tools.map((tool) => tool.name)` 计算出当前会话的“最终可用工具列表”。
*   **Prompt 注入**：计算出的 `toolNames` 数组会被拼接到 System Prompt 的工具定义区。

## 4. 总结：消息处理流中的工具分配

当一条消息进入 OpenClaw 时，工具的分配遵循以下路径：
1.  **识别 Profile**：查看当前 Agent 配置的工具方案（如 `coding`）。
2.  **筛选 Catalog**：从核心目录中筛选出该方案对应的工具子集。
3.  **应用 Policy**：根据黑白名单进行增删。
4.  **动态注入**：将最终确定的工具名称（如 `read`, `exec`, `web_search`）作为 `Tool Names` 写入 System Prompt。

**核心哲学**：AI 知道自己能用什么，是因为 Prompt 中明确授权了这些工具。如果一个工具未出现在 Prompt 中，AI 就不会尝试调用它，这构成了系统的第一道安全防线。
