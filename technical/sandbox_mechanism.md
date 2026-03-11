# OpenClaw 沙箱 (Sandbox) 机制总览

OpenClaw 的沙箱机制是其安全架构的核心，旨在为 AI 提供一个“可以随意折腾且不用担心破坏宿主系统”的隔离实验室。它允许 AI 在受控环境中执行运行脚本、安装依赖、操控浏览器等高风险任务。

---

## 1. 技术选型
*   **容器引擎**：支持 Docker (默认) 与 Podman (支持 Rootless 模式)。
*   **基础操作系统**：定制化 Debian Bookworm。
*   **通信协议**：基于标准的 ACP (Agent Client Protocol) 协议桥接。

## 2. 核心架构设计
OpenClaw 遵循 **“指挥与执行分离”** 的原则。Gateway 作为大脑在宿主机运行，而所有涉及物理资源变更的工具调用均按需分发至沙箱执行。

---

## 📂 专项架构文档
为了深入了解沙箱各环节的实现，请参阅以下专项文档：

### ⚙️ 生命周期与激活
*   **[生命周期与激活逻辑](./sandbox_lifecycle.md)**：解析延迟加载 (Lazy Loading)、按需触发流程及配置一致性校验。

### 🏗️ 环境与隔离
*   **[镜像解耦与专用环境](./sandbox_images.md)**：详述宿主机环境与 Sandbox 镜像的解耦、专用 Dockerfile 变体。
*   **[隔离粒度与作用域 (Scope)](./sandbox_scope.md)**：解析 `agent`, `session`, `shared` 三种隔离模式及其多 Agent 并行逻辑。

### 🛠️ 执行与交互
*   **[工具执行流程 (exec)](./sandbox_execution.md)**：解析指令包装、登录 Shell 模拟、环境变量注入及 PATH 优先级。
*   **[浏览器专项隔离](./sandbox_browser.md)**：解析浏览器独立容器策略、Bridge Server 桥接及可视化 noVNC 监控。

### 🛡️ 安全与权限
*   **[文件系统与安全加固](./sandbox_security.md)**：涵盖目录挂载策略、系统级硬化 (Seccomp/AppArmor) 及“提权”审批协作流。

---

## 3. 设计哲学总结
OpenClaw 的沙箱不是为了将 AI “关起来”，而是为了通过精细的权限收敛和物理隔离，赋予 AI 真正的 **“行动自由”**。
