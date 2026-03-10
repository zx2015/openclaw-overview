# OpenClaw 核心编排：Gateway

`src/gateway` 目录包含了 OpenClaw 系统中最核心的控制平面实现，负责连接用户、通讯渠道与 AI Agent。

## 1. 核心模块组成
网关的功能高度模块化，通过以下关键文件实现：
*   **身份验证 (Auth)**：
    *   `auth.ts` / `connection-auth.ts` / `device-auth.ts`：管理多端连接的安全校验、设备配对及会话令牌。
*   **运行时服务**：
    *   `server-http.ts`：提供 RESTful 管理接口。
    *   `server-ws-runtime.ts`：处理实时的 WebSocket 双向通信，支持流式对话响应。
*   **渠道生命周期**：
    *   `server-channels.ts`：动态管理各社交平台适配器（WhatsApp, Discord 等）的加载与心跳。
*   **Agent 逻辑桥接**：
    *   `server-chat.ts`：处理核心对话流转发。
    *   `agent-prompt.ts`：负责系统级 Prompt 的动态组装。
*   **节点与工具调度**：
    *   `node-registry.ts`：管理本地及远程工作节点的注册状态。
    *   `tools-invoke-http.ts`：实现 AI 对 HTTP API 工具的调用逻辑。

## 2. 系统指挥中心：`server.impl.ts`
`server.impl.ts` 是整个 OpenClaw 运行时的灵魂，负责编排所有底层服务的启动与协同。

### 关键职责分析
1.  **子系统初始化 (Orchestration)**：
    *   依次启动子代理注册表 (Subagent Registry)、技能引擎 (Skills)、插件中心。
    *   初始化健康检查 (Diagnostics) 与心跳监控子系统。
2.  **配置驱动与持久化**：
    *   负责 `config.json` 等配置文件的动态加载与 schema 校验。
    *   实现配置版本迁移逻辑，确保升级时配置的平滑兼容。
3.  **机密与状态管理**：
    *   管理 Auth Token、API Key 等敏感机密。
    *   支持运行时状态快照，确保系统重启后能快速恢复部分上下文。
4.  **启动流程编排 (`startGatewayServer`)**：
    *   **协议层**：建立 HTTP 与 WebSocket 服务器。
    *   **服务发现**：初始化发现服务 (Discovery) 逻辑。
    *   **网络增强**：支持通过 Tailscale 隧道将本地网关安全暴露到外部公网。
    *   **后台任务**：启动 Cron 定时任务调度器，执行主动任务（如每日简报）。
