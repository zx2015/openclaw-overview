# OpenClaw Gateway 服务编排逻辑

OpenClaw Gateway 是整个系统的“控制中心”，负责协调 AI 智能体、通讯渠道、本地硬件以及自动化任务。其核心编排逻辑位于 `src/gateway/server.impl.ts`，展现了一个高度解耦且具备强韧性的架构设计。

## 1. 运行时状态初始化 (`createGatewayRuntimeState`)

在启动之初，系统首先建立最底层的 **运行时环境快照**，确保所有后续子系统有统一的状态上下文：
*   **服务器集群**：初始化支持 TLS 及多 IP 绑定的 HTTP/HTTPS 和 WebSocket 服务器。
*   **连接追踪**：建立 `clients` (WS 实时连接) 与 `dedupe` (消息去重器) 维护机制。
*   **聊天状态机**：统一管理 `chatRunState`、文本增量缓冲区以及用于任务干预的 `AbortController`。
*   **可视化宿主**：若启用 **Canvas** 功能，将在此阶段完成宿主环境的初始化。

## 2. 核心基础设施编排

状态就绪后，Gateway 开始按序激活核心支撑子系统：
*   **节点注册表 (NodeRegistry)**：管理所有连接的外部节点（如移动设备、摄像头、传感器），支持 **节点订阅模式**，允许将特定 Agent 事件定向推送到目标设备。
*   **任务调度 (CronService)**：加载并激活定时任务引擎，负责执行如“每日简报”或自动化脚本等计划任务。
*   **发现与暴露 (Discovery)**：
    *   **局域网**：通过 Bonjour/mDNS 在本地网络广播服务。
    *   **广域网**：可选地通过 **Tailscale** 隧道实现零配置的安全公网暴露。
*   **心跳机制 (HeartbeatRunner)**：维持各个渠道与节点间的活跃状态，确保长连接的可靠性。

## 3. 插件与渠道生命周期管理

*   **渠道管理器 (ChannelManager)**：动态加载并监控 WhatsApp、Telegram、微信等通讯插件，确保消息能准确路由。
*   **服务侧挂 (GatewaySidecars)**：启动辅助进程，如独立的插件服务进程、浏览器控制后端（Playwright）等。
*   **技能实时同步**：注册技能变更监听器。当本地 `.skill` 文件变化时，系统会触发 **30 秒防抖 (Debounce) 刷新**，自动同步远程节点的资源。

## 4. 消息与事件处理流

*   **Agent 事件处理器 (`createAgentEventHandler`)**：作为核心转换器，将 Agent 的底层行为（Speech, Tool Call, Lifecycle）转化为标准的 WS 广播消息或通过 `nodeSendToSession` 投递给特定渠道。
*   **WS 协议绑定 (`attachGatewayWsHandlers`)**：将 `agents.list`、`sessions.send` 等核心方法以及插件定义的自定义扩展方法挂载到 WebSocket 接口。

## 5. 动态重载与自愈 (Self-healing)

这是 OpenClaw 实现长驻运行的关键：
*   **配置热重载 (Hot Reload)**：监测 `config.json` 变化。若仅涉及 Token 或 UI 偏好等非核心变更，系统会计算 **重载计划** 原地重启相关子系统，无需中断进程。
*   **平滑重启**：若涉及端口绑定等核心变更，系统会优雅地请求完整重启，并确保在重启前完成所有连接的清理。

## 6. 安全与机密管理

*   **机密运行时快照 (`SecretsRuntimeSnapshot`)**：在启动/重载时预备好 API Keys 的预检快照，确保 AI 在推理时密钥“即取即用”。
*   **执行审批 (ExecApprovalManager)**：集中管理需要用户手动确认的高风险操作（如删除、执行高危脚本）。

## 7. 编排架构图简述

```text
[ Config/Secrets ] <--> [ Gateway Server (Express/WS) ] <--> [ Clients (Web/App) ]
                              |          |
      +-----------------------+          +-----------------------+
      |                       |                                  |
[ Channels ]           [ Automation ]                     [ Nodes/Devices ]
- WhatsApp             - Cron Service                     - Remote Cameras
- Telegram             - Heartbeat                        - Mobile Apps
- Slack                - Update Check                     - Browser Control
                              |
                     [ Plugin Registry ]
                     - Custom Tools/Handlers
                     - Global Hooks (Start/Stop)
```

## 8. 设计哲学

*   **防火墙思维**：所有敏感的物理操作（执行、读写）必须通过审批经理或特定的机密处理逻辑。
*   **原子化启停**：子系统具备明确的 `start`/`stop` 方法，由 `createGatewayCloseHandler` 严格协调关闭顺序（先连接，后逻辑，最后句柄）。
*   **高韧性设计**：通过热重载、自动心跳和崩溃任务恢复，确保系统作为“24/7 数字管家”的长期稳定性。
