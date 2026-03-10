# OpenClaw 智能体选择与动态切换

在 OpenClaw 中，用户可以灵活地切换当前正在对话的 Agent，或在不同的会话（Session）中分配专属的 Agent。这使得 OpenClaw 能够像“多面手”一样，在不同场景下扮演不同的专业角色。

## 1. 切换与选择方式

### 命令行界面 (CLI)
在终端运行 OpenClaw 时，可以直接指定 ID：
*   **启动指定**：`openclaw run --agent <agentId>`。
    *   *示例*：`openclaw run --agent coder` 或 `openclaw run --agent personal`。
*   **列表查看**：执行 `openclaw agents list` 可查看所有已配置的 Agent 及其 ID。

### 对话中动态切换 (Slash Commands)
在 WhatsApp、Telegram、Slack 或 Web 聊天窗口中，可以使用斜杠命令：
*   **切换指令**：发送 `/agent <agentId>`。
    *   **效果**：网关 (Gateway) 会实时更新当前 Session 绑定的 `agentId`。后续对话将立即由新 Agent 接手，加载其专属的 `SOUL.md` 和 `AGENTS.md`。

### 多会话并行管理 (Multi-Session)
你可以在不同的通讯渠道绑定完全不同的 Agent，互不干扰：
*   **Telegram**：可以为不同的 Bot 或同一个 Bot 的不同 Topic 分配不同的 Agent。
*   **WhatsApp**：不同的手机号码（若配置了多个）可以绑定专属 Agent。
*   **分身并发**：这意味着你可以让“程序员 Agent”在 Slack 帮你改 Bug，同时“生活助理 Agent”在 WhatsApp 帮你订餐。

### 可视化控制台 (Dashboard)
在 Web UI 界面中：
*   **设置菜单**：通过 Session 设置中的下拉菜单一键切换。
*   **实时反馈**：切换后，UI 上的头像 (Emoji) 和身份说明（来自 `IDENTITY.md`）会立即同步更新。

## 2. 自动路由逻辑 (Auto-Routing)
这是面向高级用户的配置功能，允许系统根据消息来源自动选择 Agent：
*   **群组路由**：来自特定工作群的消息自动由 `group-moderator` 处理。
*   **私聊路由**：默认私聊消息由 `personal-assistant` 处理。

## 3. 技术底层：会话绑定机制

系统的核心映射逻辑位于 `src/config/sessions.ts`：
1.  **持久化存储**：系统维护一个 `sessionKey` (如手机号或 UUID) -> `agentId` 的持久化映射。
2.  **动态加载**：每当消息进入，网关会查询该映射，并从磁盘加载对应的 **工作区 (Workspace)** 文件。
3.  **指令响应**：`/agent` 命令的本质是原子化地更新数据库中该 Session 对应的 `agentId` 记录。

## 4. 总结
*   **变身**：在聊天框输入 `/agent <名称>` 即可完成“灵魂”替换。
*   **分身**：在不同 App 或群组配置不同 Agent，实现并行服务。

你可以通过 `openclaw agents list` 随时查看你当前拥有的“人格”库。如果你想创建一个全新的 Agent，请参考 **[如何新建智能体](./create_agent.md)**。
