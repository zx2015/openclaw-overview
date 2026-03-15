# OpenClaw 沙箱：生命周期与激活逻辑

OpenClaw 采用“配置开启开关，命令按需触发”的模式来管理沙箱容器的生命周期。

## 1. 延迟加载 (Lazy Loading) 策略
沙箱容器并非在 OpenClaw Gateway 启动时立即开启，而是在 AI 真正需要执行受限操作时才进行初始化。这种设计显著降低了系统的基础内存占用。

## 2. 核心激活流程 (`ensureSandboxContainer`)
当 Agent 决定调用 `exec` (Bash) 或 `browser` 工具时，系统自动触发以下逻辑：
1.  **模式预检与决策**：根据配置优先级（全局默认 vs 单独覆盖）及 `sandbox.mode`（`off`, `non-main`, `all`）判断当前会话是否需要隔离。
2.  **作用域匹配与状态检查**：根据 `Scope` 策略（`agent`, `session`, `shared`）锁定对应的物理容器名，并检查该容器是否已创建或当前是否存活。
3.  **自动执行链**：若容器不就绪，网关在后台按顺序执行以下指令：
    *   **`docker create`**：基于镜像名、挂载路径及资源限制等最新配置创建容器。
    *   **`docker start`**：激活容器进程（主进程通常为 `sleep infinity` 以保持驻留）。
    *   **`docker exec`**：待容器完全就绪后，将 AI 生成的任务指令发送至容器内部执行。

## 3. 配置一致性校验 (`configHash`)
...
## 5. 配置优先级与差异化管控 (Granular Control)

OpenClaw 采用 **“全局默认 + 局部覆盖”** 的继承逻辑，允许用户精细化地决定每个智能体的执行环境。

### 策略继承逻辑
沙箱策略遵循从全局到特定 Agent 的逐级覆盖原则。根据 `openclaw sandbox explain` 的诊断逻辑，优先级如下：

1.  **Agent 级覆盖 (最高优先级)**：路径为 `agents.list[].tools.sandbox.enabled`。
2.  **全局级默认**：路径为 `tools.sandbox.enabled`。

> **注**：在部分版本中，`sandbox.mode`（`off`, `non-main`, `all`）作为行为模式开关，与 `tools.sandbox.enabled` 共同决定最终执行策略。

### 差异化配置示例
你可以在 `openclaw.json` (或通过 `config` 命令) 为不同 Agent 定义不同的沙箱策略：

*   **特定 Agent 开启沙箱**（即使全局关闭）：
    ```json
    {
      "id": "my-secure-agent",
      "tools": { "sandbox": { "enabled": true } }
    }
    ```
*   **特定 Agent 禁用沙箱**（即使全局开启）：
    ```json
    {
      "id": "my-local-agent",
      "tools": { "sandbox": { "enabled": false } }
    }
    ```

### 典型应用场景
*   **场景 A：极致安全 (Default-in-Sandbox)**：全局设为 `true`（或 `all`），仅对受信任的本地运维或开发 Agent（如 `trusted-dev`）覆盖设为 `false`（或 `off`）。
*   **场景 B：按需隔离 (Direct-by-Default)**：主要在本地开发，全局设为 `false`（或 `off`），仅对处理外部不可信代码、执行高风险搜索或审计类 Agent（如 `code-reviewer`）开启沙箱。
*   **场景 C：按“会话类型”自动防御 (`non-main`)**：
    *   **私聊 (Main Session)**：默认宿主机运行，方便操作本地硬件（如 GPU/摄像头）。
    *   **群聊/频道 (Non-main Session)**：默认沙箱运行，自动隔离来自社交渠道不信任成员的潜在风险指令。

### 配置权衡依据 (Rationales)
1.  **性能考量**：直接模式响应最快；沙箱启动与 `docker exec` 会带来约数百毫秒的额外延迟。
2.  **权限需求**：若 Agent 需要直接访问宿主机资源、特定硬件（GPU/传感器）或敏感系统路径，直接模式更为便捷。
3.  **安全性**：具有“全网搜索”、“运行第三方脚本”或“自动审查 PR”能力的 Agent 强制要求进入沙箱（Docker Sandbox），确保宿主机绝对安全。

## 6. 环境真实性验证
用户可以通过询问 Agent 如下问题来验证隔离是否生效：
> “请运行 `hostname` 和 `id` 命令，告诉我你在哪？”

*   **沙箱状态**：返回 `openclaw-sbx-xxx` 之类的主机名，UID 通常为 `sandbox` 或 `1000`。
*   **宿主机状态**：返回用户真实的机器名与系统用户名。
系统通过对镜像名、内存限制、挂载路径等参数计算哈希值：
*   **检测变更**：每次尝试调用沙箱时，都会对比现有容器的 `configHash`。
*   **自愈重建**：若检测到配置不匹配且容器当前不处于任务活跃期，系统会自动执行 `recreate` 逻辑（销毁并重建），确保执行环境始终符合最新的安全策略。

## 4. 任务结束与回收
任务完成或超时后，容器根据其 `Scope` 策略进入保留或销毁状态。主机会自动清理相关的临时挂载点，只保留工作区中明确要求持久化的产出。
