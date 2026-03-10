# OpenClaw 核心特性

## 1. Markdown-first Memory (文件首位存储)
*   **本地存储**：弃用传统数据库，所有的短期会话、长期记忆和用户配置都以 Markdown 文件的形式存储（通常在 `~/clawd/` 下）。
*   **透明性**：用户可以随时用文本编辑器查看 AI 的记忆，手动修改配置。
*   **版本控制**：可以通过 Git 对 AI 的状态进行全量备份和历史回溯。

## 2. ReAct Reasoning (推理-执行循环)
*   **思维链条**：AI 内部采用 **Thought (思考) -> Action (行动) -> Observation (观察) -> Thought** 的闭环模型。
*   **自主决策**：能够根据当前任务，自动决定调用哪一个 Skill。

## 3. Proactive Automation (主动执行)
*   **不仅是回复**：集成 Cron 任务引擎，AI 可以按照用户设定的频率自动发起会话。
*   **事件驱动**：支持监听 Webhook 等事件，实现从被动回答到主动通知的跨越。

## 4. 全渠道集成 (Omnichannel Integration)
OpenClaw 支持数十种主流通讯平台接入，允许用户在自己熟悉的社交应用中控制 AI：
*   **WhatsApp / iMessage**
*   **Telegram / Discord**
*   **Slack / 飞书 (Lark Suite)**
*   **微信 (WeChat，通过 WebChat)**
*   **GitHub / Jira** 等。

## 5. 多模态交互与可视化
*   **语音能力**：支持语音唤醒及实时通话模式，实现更自然的交流。
*   **Live Canvas**：由 AI 驱动的可视化工作区，让用户能够直观地看到 AI 的工作过程和产出成果。

## 6. MCP 与标准化
*   **MCP 支持**：通过 `mcporter` 原生支持 **Model Context Protocol (MCP)**。
*   **桥接模式**：通过标准的 ACP 协议接口保持核心逻辑的简洁与稳定性。

## 7. 实战执行力 (Action-Oriented)
*   **Shell & Docker 执行**：通过 `bash-tools` 组件，OpenClaw 可以直接在宿主机或隔离的 Docker 容器中执行 Shell 命令。
*   **执行确认机制 (exec-approval)**：对于高风险操作（如删除文件或运行复杂脚本），具备强制的用户确认机制，确保操作可控。

## 8. 智能上下文管理 (Smart Context)
*   **消息压缩 (Compaction)**：自动对长对话历史进行智能压缩，平衡上下文完整性与响应速度。
*   **长期记忆搜索 (Memory Search)**：结合向量存储，精准提取历史相关信息。
*   **上下文窗口守卫 (Context Window Guard)**：实时监控 Token 消耗，通过智能调度确保长对话的稳定性与成本效益。

## 9. 多代理系统 (Multi-Agent System)
*   **子代理注册表 (Subagent Registry)**：管理多个专业化子代理，支持将复杂任务拆分给擅长不同领域的代理协同工作。

## 10. 广泛的模型生态支持
OpenClaw 适配了几乎所有主流和领先的模型提供商，包括：
*   **国际主流**：Anthropic (Claude), OpenAI (GPT), Gemini (Google), Bedrock (AWS)。
*   **本地模型**：Ollama, LM Studio (本地离线运行)。
*   **国内大模型**：豆包 (Doubao), Kimi (Kimi-coding), Minimax 等。

## 12. 极致的对话体验 (Conversational UX)
OpenClaw 并非简单地转发模型输出，而是通过网关层进行了深度优化：
*   **平滑流式推送**：内置毫秒级频率控制器 (150ms Delta Frequency)，确保即使在网络波动时，用户在 WhatsApp 或 Web 端看到的文字输出也依然丝滑平稳。
*   **隐私清洗 (Scrubbing)**：在消息推送给用户之前，系统会自动剥离 AI 内部思考过程中的私有标签（如 `<thought>`），确保界面简洁且保护内部逻辑。
*   **多模态降级处理**：无论用户发送的是复杂的图片数组、混合附件还是纯文本，系统都能智能转化为 AI 最易理解的结构化输入，实现无损的跨模态交互体验。
*   **无卡顿工具切换**：在 AI 决定执行工具（如查询天气或运行脚本）的一瞬间，系统会立即冲刷缓冲区，确保用户能即刻看到 AI 之前的思考，而非在等待工具执行时面对“空白期”。

