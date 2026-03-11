# OpenClaw MCP 集成机制 (Model Context Protocol)

OpenClaw 采用了一种高度解耦的 **“桥接模式”** 来支持 Model Context Protocol (MCP)。它并不在核心代码中直接实现复杂的协议栈，而是通过专门的 `mcporter` 工具实现与外部 MCP 服务器的交互。

## 1. 核心架构：桥接模式 (The Bridging Strategy)

OpenClaw 将 MCP 的复杂性外包给了独立工具：
*   **McPorter**：由 OpenClaw 同一开发团队打造的独立 CLI 工具 ([mcporter.dev](https://mcporter.dev))，专门用于管理、配置和调用 MCP 服务器。
*   **技能集成**：OpenClaw 通过 `skills/mcporter/` 目录下的专用 Skill，将 `mcporter` 的能力暴露给 AI Agent。
*   **设计优势**：这种设计允许用户独立升级 `mcporter` 以支持最新的 MCP 特性，而无需频繁变动 OpenClaw 的稳定内核，符合“核心轻量化”的设计原则。

## 2. 动态发现与自省 (Discovery & Introspection)

OpenClaw 并不在启动时预知所有 MCP 工具，而是通过实时“自省”机制来获取环境能力：
*   **`mcporter list` 指令**：这是发现能力的关键。当 Agent 需要了解当前环境可用的扩展能力时，会调用此命令获取所有已配置 MCP Server 的工具列表及其参数 Schema。
*   **工具注入方式**：
    *   **动态发现 (Dynamic)**：Agent 在 `AGENTS.md` 的指引下，若发现内置工具无法满足需求，会主动运行 `mcporter list`。通过返回的 JSON/Text 描述，AI 能够即时“学会”如何调用（如 `linear.create_issue`）。
    *   **预定义/代码生成 (Static)**：OpenClaw 支持通过 `mcporter generate-cli` 为特定的 MCP Server 事先生成一套标准的 Agent 工具定义，从而提升调用的稳定性。

## 3. 双层封装策略 (Two-Layer Encapsulation)

为了平衡通用性与执行精度，OpenClaw 采用了独特的双层封装逻辑：

### 第一层：通用封装 (`mcporter` Skill)
*   **定位**：提供统一的外部协议入口。
*   **优势**：AI 通过这一个 Skill 即可访问 MCP 生态中成千上万个工具，极大地扩展了能力边界，无需为每个 Server 编写适配代码。

### 第二层：特定封装 (Generated/Intent-based Skills) —— **最佳实践**
对于高频使用的特定服务（如 Postgres 或 GitHub），推荐将其再次封装为专用的 OpenClaw Skill：
*   **做法**：在 `skills/` 下新建目录（如 `my-db/`），并在其 `SKILL.md` 中明确写出：“本技能通过 `mcporter call postgres.query` 执行 SQL”。
*   **价值**：相比“去 mcporter 仓库里翻找”，直接提供意图导向的上下文（Context）能显著提升 AI 在复杂任务中的执行成功率。

## 4. 未来的演进：原生 MCP 支持 (First-class Citizen)

在 OpenClaw 的长期路线图中，MCP 将被提升为“一等公民”：
*   **自动扫描映射**：Gateway 启动时将自动扫描 mcporter 配置，并将 MCP 工具直接映射为原生的 `AgentTool`。
*   **零转换调用**：AI 将不再需要拼接繁琐的 `mcporter call ...` Shell 命令，而是像调用 `read_file` 一样直接通过标准的 ACP 接口调用 `linear_create_issue`。

## 5. 开发者建议与最佳实践

1.  **全局配置先行**：在宿主机完成 `mcporter config add` 确保基础服务连通。
2.  **引导 AI 意识**：在 `AGENTS.md` 中明确声明：“You have access to extended tools via MCP. Use `mcporter list` to discover them.”
3.  **意图索引优化**：为核心业务工具编写简单的 `SKILL.md` 作为索引，指导 AI 在特定场景下精准匹配 `mcporter` 指令。

## 6. 总结：解耦的合作
... Applied fuzzy match at line 1-3.
*   **OpenClaw 负责 “想”**：逻辑判断、对话管理、决定何时需要调用工具。
*   **McPorter 负责 “做”**：底层协议对接、进程生命周期管理、OAuth 流程处理。

通过这种“想做分离”的架构，OpenClaw 成为了一个极具扩展性的 AI 编排平台，能够无缝接入 MCP 生态中的数千种现有工具。
