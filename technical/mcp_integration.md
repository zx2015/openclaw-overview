# OpenClaw MCP 集成机制 (Model Context Protocol)

OpenClaw 采用了一种高度解耦的 **“桥接模式”** 来支持 Model Context Protocol (MCP)。它并不在核心代码中直接实现复杂的协议栈，而是通过专门的 `mcporter` 工具实现与外部 MCP 服务器的交互。

## 1. 核心架构：桥接模式 (The Bridging Strategy)

OpenClaw 将 MCP 的复杂性外包给了独立工具：
*   **McPorter**：由 OpenClaw 同一开发团队打造的独立 CLI 工具 ([mcporter.dev](https://mcporter.dev))，专门用于管理、配置和调用 MCP 服务器。
*   **技能集成**：OpenClaw 通过 `skills/mcporter/` 目录下的专用 Skill，将 `mcporter` 的能力暴露给 AI Agent。
*   **设计优势**：这种设计允许用户独立升级 `mcporter` 以支持最新的 MCP 特性，而无需频繁变动 OpenClaw 的稳定内核，符合“核心轻量化”的设计原则。

## 2. 交互流程：Agent 如何使用 MCP

当 Agent 需要调用一个 MCP 工具（如 `linear.create_issue`）时，其内部执行逻辑如下：

1.  **意图识别**：Agent 根据 `AGENTS.md` 或 `SKILL.md` 的描述，意识到 `mcporter` 可以解决当前问题。
2.  **命令生成**：Agent 调用 `exec` 工具执行 Shell 命令：
    `mcporter call <server.tool> --args '{"key": "value"}'`
3.  **协议转换**：`mcporter` 接收到命令后，负责处理与底层 MCP 服务器的通信（无论是 stdio 还是 HTTP 协议）。
4.  **结果解析**：`mcporter` 将结果以标准化 JSON 格式返回，Agent 解析后继续对话。

## 3. MCP 配置与认证

配置主要在 `mcporter` 层级进行，而非 OpenClaw 的主 `config.json`。

### 方式 A：命令行配置 (Agent 自我配置)
由于 Agent 拥有 `exec` 权限，它实际上具备 **“自我进化”** 的能力：
*   **添加服务器**：`mcporter config add <name> <command_or_url>`
*   **OAuth 认证**：`mcporter auth <server>`。若服务器需要授权（如 Google Drive），Agent 会生成 URL 请用户在浏览器中点击确认。

### 方式 B：手动编辑配置文件
用户可以直接修改 `mcporter` 的配置文件（通常位于 `~/.openclaw/config/mcporter.json`）：

```json
{
  "mcp": {
    "servers": {
      "everything": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-everything"]
      },
      "postgres": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-postgres", "postgresql://localhost/mydb"]
      }
    }
  }
}
```

## 4. OpenClaw 侧的辅助配置

虽然 MCP 服务器由 `mcporter` 管理，但在 OpenClaw 的 `.env` 或 `config.json` 中仍需关注：
*   **路径权限**：确保 Agent 有权访问 `mcporter` 二进制文件。
*   **环境变量**：某些 MCP 服务器依赖的环境变量（如 API Key）应定义在 OpenClaw 的环境上下文中，以便在 `exec` 时传递给 `mcporter`。

## 5. 总结：解耦的合作

*   **OpenClaw 负责 “想”**：逻辑判断、对话管理、决定何时需要调用工具。
*   **McPorter 负责 “做”**：底层协议对接、进程生命周期管理、OAuth 流程处理。

通过这种“想做分离”的架构，OpenClaw 成为了一个极具扩展性的 AI 编排平台，能够无缝接入 MCP 生态中的数千种现有工具。
