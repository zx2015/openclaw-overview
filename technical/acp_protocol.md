# OpenClaw ACP 协议详解 (Agent Client Protocol)

ACP 是 OpenClaw 内部通讯的核心标准，是一个基于 **JSON-RPC 2.0** 风格的异步双向协议。它的设计目标是标准化客户端（如 IDE、终端、Raycast 插件）与 AI Agent 之间的交互。在 OpenClaw 实现中，它通常通过 **NDJSON**（换行符分隔的 JSON）在 stdio 或 WebSocket 上传输。

## 1. 核心消息结构

所有通讯均遵循标准的请求/响应/通知模式，确保了交互的可预测性：
*   **Request**: 包含 `id`、`method` 和 `params`，要求服务端必须回复。
*   **Response**: 包含对应请求的 `id` 及执行 `result`。
*   **Notification**: 仅包含 `method` 和 `params`，用于单向的状态更新，无需回复。

## 2. 关键协议方法 (Methods)

| 方法名 | 类型 | 核心描述 |
| :--- | :--- | :--- |
| **`initialize`** | Request | 协商协议版本（`PROTOCOL_VERSION`）及能力集（Capabilities）。 |
| **`newSession`** | Request | 创建新会话，支持指定 `cwd`（当前工作目录）和元数据。 |
| **`loadSession`** | Request | 恢复既有会话，OpenClaw 会自动回放（Replay）历史消息流。 |
| **`prompt`** | Request | 发送用户输入，支持文本与图片的多模态混合内容。 |
| **`setSessionMode`** | Request | 切换会话运行模式（如调整 Thought Level 推理等级）。 |
| **`listSessions`** | Request | 获取 Gateway 上当前所有活跃会话的快照。 |
| **`requestPermission`** | Request | 由客户端发起的权限确认，用于审批高风险工具调用。 |

## 3. 状态与能力协商机制

ACP 通过复杂的初始化握手实现环境对齐：
*   **能力协商 (Capabilities)**：
    *   `agentCapabilities`: 告知客户端是否支持图片/音频输入、嵌入式上下文或会话加载。
    *   `mcpCapabilities`: 明确 Agent 是否具备通过 HTTP/SSE 接入 MCP 服务的能力。
*   **动态配置发现**：OpenClaw 将内部的 `thoughtLevel` (思考深度)、`verboseLevel` (日志详尽度) 等配置映射为 ACP 的 `SessionConfigOption`，允许客户端动态渲染对应的控制 UI。
*   **实时状态反馈**：通过 `SessionModeState` 同步 Agent 的当前繁忙状态（如“正在思考”、“重连中”）。

## 4. 安全与审计特性

OpenClaw 为 ACP 引入了增强的溯源机制：
*   **来源回执 (Source Receipt)**：在每条消息中自动注入发送方的 `Hostname`、`CWD` 及 `SessionID`。
*   **渠道标记**：在内部消息流中明确标记来源渠道为 `acp`，工具来源为 `openclaw_acp`，便于日志审计与权限隔离。

## 5. 工具调用安全流 (Security Flow)

这是 ACP 协议中最具交互性的闭环：
1.  **发起请求**：Agent 在 `prompt` 的流式响应中嵌入工具调用请求。
2.  **权限评估**：客户端收到请求后，根据预设策略决定是自动通过（如读取 `cwd` 内文件）还是挂起并提示用户点击“批准”。
3.  **结果闭环**：权限获准后，Agent 执行工具逻辑并将结果异步传回。

## 6. 总结：通用的适配层

ACP 是 OpenClaw 的 **“万能适配器”**。它使得任何第三方工具（无论是 VS Code 插件还是自定义脚本）都能以类型安全、异步且受控的方式与 OpenClaw Gateway 深度集成，而无需关心后端复杂的社交平台渠道协议。
