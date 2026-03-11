# OpenClaw 沙箱：隔离粒度 (Scope) 与多 Agent 并行

OpenClaw 允许通过 `agents.defaults.sandbox.scope` 参数灵活控制容器的可见性边界，实现资源与安全的平衡。

## 1. 三大隔离模式

### `agent` 模式 (默认)
*   **隔离级别**：按智能体隔离。
*   **逻辑**：系统为每个唯一的 `agentId` 创建专属持久容器。
*   **表现**：同一 Agent 在不同通讯渠道（如 WhatsApp 和 Slack）的对话将共享同一个沙箱空间。

### `session` 模式 (最严格)
*   **隔离级别**：按会话窗口隔离。
*   **逻辑**：每个独立的 `sessionId` 都会获得专属的全新容器。
*   **表现**：即使是同一 Agent，不同窗口间的执行环境也完全独立，提供最高安全保障。

### `shared` 模式 (共享模式)
*   **隔离级别**：全局共享。
*   **配置**：`openclaw config set agents.defaults.sandbox.scope shared`。
*   **表现**：所有 Agent、所有会话挤在同一个名为 `openclaw-sbx-shared` 的容器里。不同 Agent 间的文件和环境变量互相可见。

## 2. 多 Agent 运行的逻辑本质
在 OpenClaw 中，单容器完全可以承载多个 Agent 的指令，其本质是 **Gateway 指令路由**：
*   **Gateway (大脑)**：在宿主机管理全局配置。
*   **Sandbox (黑盒)**：仅作为指令接收并执行的 Linux 环境。
*   如果多个 Agent 的 Scope 配置解析到同一个物理容器 ID，它们就会在同一个沙箱内“并行”工作。

## 3. 共享沙箱的权益对比

| 维度 | 优势 (Pros) | 风险 (Cons) |
| :--- | :--- | :--- |
| **资源** | 节省内存，避免启动海量驻留进程。 | 易发生 OOM 或资源争抢。 |
| **协作** | 方便 Agent A 产出、Agent B 消费。 | 数据隐私无法在 Agent 间隔离。 |
| **安全** | 配置统一，维护成本低。 | 环境污染（如 Agent A 误删系统库）。 |

## 4. 相关指令
*   **查看当前粒度**：`openclaw config get agents.defaults.sandbox.scope`
*   **强制重构环境**：`openclaw sandbox recreate`
