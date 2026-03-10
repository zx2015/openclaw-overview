# OpenClaw 消息交互与上下文引擎

OpenClaw 并非简单转发 AI 模型输出，其网关层 (Gateway) 承担了极其复杂的实时交互编排工作。

## 1. 实时流处理：`src/gateway/server-chat.ts`

### 消息状态与 Run 生命周期
每次对话启动时，系统都会通过 `createChatRunState` 创建一个临时的会话上下文：
*   **增量缓存 (Buffer)**：缓存尚未完整合并的增量词片 (Tokens)，确保输出文本的连贯性。
*   **频率平滑 (Throttling)**：内置 150ms 推送频率上限，避免 AI 模型输出过快导致客户端 UI 刷新抖动或掉帧。
*   **精准路由**：通过 `clientRunId` 将响应精准投递到发起请求的原始连接。

### 事件驱动模型 (`createAgentEventHandler`)
网关监听 Agent 核心发出的实时事件并进行深度后处理：
*   **增量推送 (Assistant Delta)**：
    *   **智能去重**：利用 `resolveMergedAssistantText` 消除某些模型流式输出中的冗余字符。
    *   **标签脱敏**：实时调用 `stripInlineDirectiveTagsForDisplay` 剔除 AI 内部推理使用的隐藏指令。
    *   **静默处理**：识别并拦截 AI 的内部自校验或心跳包，不干扰用户体验。
*   **工具联动 (Tool Execution)**：
    *   **详尽度动态切换**：支持对不同工具配置不同的日志详细度。
    *   **冲刷缓冲区 (Flush)**：在调用可能耗时的工具前，强制推空当前文本缓冲区，避免用户因等待工具结果而看不到之前的回复。
*   **最终态确认 (Lifecycle)**：确保在对话 `end` 或 `error` 时，完成最后的 UI 同步并进行资源释放。

## 2. 上下文构建：`src/gateway/agent-prompt.ts`

### 语义标准化与降级
*   **多模态处理 (`safeBody`)**：将用户输入的复杂多模态对象（图片、位置、文件数组）智能转换为 AI 兼容的结构化字符串，防止 Prompt 注入或渲染异常。

### 对话序列化引擎 (`buildAgentMessageFromConversationEntries`)
*   **逆向搜索 (Backwards Scan)**：精准锁定最后一条待处理的用户输入或工具结果。
### 系统级模板注入 (System Prompt Templates)
网关会动态读取并渲染 `~/.openclaw/workspace/` 或 `docs/reference/templates/` 目录下的 Markdown 模板：
*   **注入机制**：
    *   **路径识别**：读取工作区下的 `AGENTS.md` (操作逻辑)、`SOUL.md` (性格)、`TOOLS.md` (工具笔记) 及 `IDENTITY.md` (身份标识)。
    *   **截断策略**：如果文件内容过大，系统会根据 `bootstrapMaxChars`（默认 20,000 字符）进行智能截断，以保护模型上下文窗口。
    *   **拼接方式**：将这些核心上下文拼接到 System Prompt 的开头，作为 AI 的长期记忆背景。
*   **动态更新 (Auto-Update)**：
    *   **Agent 权限**：Agent 被赋予了直接修改工作区文件的权限（通过 `src/agents/pi-tools.ts` 或 `bash-tools.ts` 提供的 `write_file`/`edit_file` 工具）。
    *   **主动学习场景**：当 Agent 识别到任务中的“教训”或收到用户的直接配置指令时，会主动发起对 `AGENTS.md` (How), `SOUL.md` (Who), 或 `USER.md` (For Whom) 的写入。
    *   **引导与重置**：在 Onboarding 过程中，系统后端会自动将用户偏好填充进模板文件。
    *   **闭环进化**：通过将实战心得沉淀入 Markdown，AI 实现了跨 Session 的行为准则自我演化。

*   **版本化管理**：
    *   在 Git 模式下，系统通过对工作区文件进行版本控制，确保 AI 的记忆与逻辑变更可追溯。

*   **语义拼接 (Semantic Stitching)**：
    1.  从数据库加载上下文历史。
    2.  按照标准化的文本模板（如 `${Sender}: ${Content}`）对历史进行高质量序列化。
    3.  将整理后的背景信息与当前请求拼接为最终的 Agent 输入 Prompt。

