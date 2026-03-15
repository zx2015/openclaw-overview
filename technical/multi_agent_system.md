# OpenClaw 多代理系统 (Multi-Agent System)

OpenClaw 的多代理系统（MAS）并非简单的进程启动器，而是一个具备状态持久化、层级化任务管理和异步通知能力的复杂协同引擎。其核心调度指挥部位于 `src/agents/subagent-registry.ts`。

## 1. 代理的“出生”与注册 (`registerSubagentRun`)

当主代理（Master Agent）识别到复杂任务需要“分身”协作时，会调用 `sessions_spawn` 工具：
*   **注册信息**：系统精确记录 `runId`（运行实例 ID）、`childSessionKey`（子代理专属会话）、`requesterSessionKey`（发起者会话）及具体任务描述（Task）。
*   **持久化追踪**：通过 `persistSubagentRunsToDisk` 将所有运行记录实时写入磁盘。
*   **故障恢复**：若 Gateway 重启，系统会在启动时通过 `restoreSubagentRunsOnce` 自动恢复对未完成子代理任务的追踪，确保任务不因系统中断而丢失。

## 2. 异步执行与生命周期监控

子代理通常以异步模式运行，主代理通过以下机制进行管理：
*   **异步等待**：主代理通过 RPC 调用 `agent.wait` 挂起当前逻辑，等待子代理反馈。
*   **事件监听 (`ensureListener`)**：注册表订阅全局 `lifecycle` 事件。无论是子代理开始执行、抛出错误还是正常结束，调度中心都能毫秒级感知并更新任务状态。
*   **超时安全阀**：支持 `runTimeoutSeconds` 配置，防止子代理因陷入死循环而无限期占用系统资源。

## 3. 结果捕获与自动通报 (`runSubagentAnnounceFlow`)

这是 OpenClaw 实现高效协作的灵魂设计：
*   **结果冻结**：任务结束时，系统通过 `freezeRunResultAtCompletion` 捕获其最终生成的有效文本（`frozenResultText`）。
*   **主动通报**：主代理无需实时轮询。子代理结束后，系统自动启动“通报流”，将成果、错误日志或超时报告，以主代理的口吻异步反馈给原始请求者。
*   **韧性重试**：若通报过程中发生网络波动，系统内置了**指数退避 (Exponential Backoff)** 的自动重试机制。

## 4. 层级化管理与清理

OpenClaw 支持深度的代理嵌套（如 Agent A -> B -> C）：
*   **级联统计**：提供 `countActiveDescendantRuns` 等函数，实时计算整个任务树中活动的后代代理数量。
*   **灵活清理策略**：
    *   **`delete` 模式**：任务完成后立即销毁子会话，释放资源。
    *   **`keep` 模式**：保留会话历史，供用户或主代理后续审计、查阅。

## 5. 舵向控制与重启 (`Steer & Restart`)

*   **动态修正**：通过 `markSubagentRunForSteerRestart`，主代理可以在任务执行中途对子代理进行“航向修正”。若需求发生变更，可标记子代理重启并应用新的指令集（Steer Instructions）。

## 6. 核心协作闭环

1.  **分身 (Spawning)**：将任务拆解给具有独立工作区和 Prompt 的专项子代理。
2.  **隔离 (Sandbox)**：子代理通常在受限沙箱中运行，保护宿主机安全。
3.  **追踪 (Tracking)**：注册表实时监控分身状态，负责崩溃后的状态恢复。
4.  **汇报 (Announcing)**：任务完成后自动汇总并反馈，实现 **“触发 -> 离线执行 -> 自动反馈”** 的完整自动化闭环。

## 7. 会话 (Session) 与 智能体 (Agent) 的关系

在 OpenClaw 的架构设计中，彻底理解 Session 与 Agent 的绑定关系是掌握多代理协作的关键。

### 核心模型：1 Session = 1 活跃 Agent
每一个会话（无论是 WhatsApp 窗口、Telegram 私聊还是 Web Tab）在同一时间都唯一绑定一个 `agentId`：
*   **一致性保障**：由绑定的 Agent（携带专属 `AGENTS.md`、模型配置等）负责响应，确保对话逻辑不发生突变。
*   **隔离设计**：防止多个 AI 在同一上下文内“打群架”，避免 Thinking 过程与工具结果相互污染导致 Prompt 爆炸。

### 核心协作模式

#### 模式 A：父子协作 (Parent-Child Spawn) —— 最常用
*   **触发**：主 Agent A 通过 `sessions_spawn` 工具将复杂任务（如分析海量文件）外包。
*   **逻辑**：系统创建一个全新的**子会话 (Sub-Session)**，并启动专门的 **Agent B**。
*   **并行**：Agent B 在隔离空间内执行，主 Agent A 可继续处理其他事务或在原会话等待。
*   **汇总**：子任务完成后，通过 `subagent-registry` 将结果自动通报回父会话。

#### 模式 B：跨会话通信 (Cross-Session Messaging)
*   **工具**：`sessions_send`。
*   **场景**：不同会话间的 Agent 互发消息。例如，“个人助理”将会话请求发送给正在另一 Session 运行的“服务器监控专家”，获取数据后再转达给用户。

#### 模式 C：动态身份切换 (Agent Swapping)
*   **操作**：通过管理指令动态替换当前 Session 的 `agentId`。这相当于在不改变聊天窗口的前提下，更换了 AI 的“灵魂”与能力集。具体操作方式请参考 **[智能体选择与切换](../docs/agent_selection.md)**。

### 设计哲学：分布式协作网络
OpenClaw 故意放弃了“单会话多代理”的混战模式，转而构建了一个分布式网络：
1.  **上下文隔离**：每个代理拥有纯净的思维空间。
2.  **权责清晰**：便于进行 Token 计费、权限校验与安全审计。
3.  **并发能力**：分身到不同 Session 的子代理可以真正实现任务的并行处理。

## 9. 典型应用场景与实战示例

### 场景 A：并行深度调研 (Parallel Research)
*   **需求**：需要同时分析多个竞争对手或技术方案。
*   **实现**：主代理通过 `sessions_spawn` 派生出多个子代理，每个代理独立负责一个搜索主题。
*   **优势**：利用多模型并发能力大幅缩短调研时间，且各主题上下文不互扰。

### 场景 B：长时执行与“离线”通知 (Long-running Tasks)
*   **需求**：执行一个需要 10 分钟以上的代码编译或数据抓取任务。
*   **实现**：主代理将任务交给子代理后，可以告诉用户“稍后我会通知你”，然后继续处理其他对话。
*   **优势**：避免主会话因超时或上下文过长而崩溃，提升用户体验。

### 场景 C：安全沙箱审计 (Security Sandboxing)
*   **需求**：运行一段来自互联网的、可能具有破坏性的 Python 脚本。
*   **实现**：启动一个配置了严苛沙箱策略（隔离网络和敏感文件）的子代理来执行。
*   **优势**：即使脚本尝试恶意操作，也仅限于该子代理的隔离环境，主代理及其所在宿主机安全无虞。

### 场景 D：专家评审团 (Expert Panel)
*   **需求**：对一份复杂的架构设计进行多维度（性能、安全、成本）评审。
*   **实现**：主代理分别派生出“性能专家”、“安全专家”和“成本控制专家”三个具有不同 Prompt 的子代理进行交叉评审。
*   **优势**：通过 Persona 隔离确保评审深度，避免单个模型在处理多目标优化时的疲软。

## 10. 故障排查建议 (Troubleshooting Tips)
*   **子代理不回复**：检查 `subagent-registry` 中的 `runTimeoutSeconds` 设置。
*   **上下文污染**：确保 `sessions_spawn` 时使用了独立的 `sessionKey`。
*   **资源溢出**：在大规模并行时，注意监控网关的并发限制配置。

## Related
- [Session 与 Agent 关系](../technical/multi_agent_system.md#7-会话-session-与-智能体-agent-的关系)
- [Gateway 编排逻辑](../technical/gateway_orchestration.md)
- [交互与上下文引擎](../technical/interaction_engine.md)
