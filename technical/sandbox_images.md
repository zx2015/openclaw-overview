# OpenClaw 沙箱：镜像解耦与专用环境

OpenClaw 将其“大脑”（网关主程序）与“手脚”（工具执行环境）进行了彻底的物理与镜像解耦。

## 1. 核心架构分离
*   **Gateway 主镜像/环境**：包含全量 TypeScript 源码、数据库、API 接口和 Agent 编排逻辑。它是决策中枢。
*   **Sandbox 执行镜像**：仅包含最小化的 Linux 指令集（如 `bash`, `curl`, `python`）。它是受控的“隔离实验室”。

## 2. 默认基础镜像
在代码 `src/agents/sandbox/constants.ts` 中，系统预设了默认镜像：
*   **镜像标识**：`openclaw-sandbox:bookworm-slim`。
*   **自动准备**：若本地缺失该镜像，OpenClaw 会自动执行 `docker pull debian:bookworm-slim` 并重命名为上述标识。

## 3. 定制化 Dockerfile 变体
项目源码提供了多套 `Dockerfile.*` 以应对不同的执行需求：
*   **`Dockerfile.sandbox`**：通用环境。创建一个名为 `sandbox` 的低权限用户，预装 Git, Python, Node.js 等常用开发工具，防止 AI 以 root 权限对容器进行全局破坏。
*   **`Dockerfile.sandbox-browser`**：浏览器专项环境。内置 Chromium 浏览器、系统字体及 **noVNC** 组件，专门用于执行 `browser` 相关的自动化任务。
*   **`Dockerfile.sandbox-common`**：共享基础层，定义了各沙箱环境通用的系统库与环境变量。

## 4. 位置关系：宿主机网关 vs 容器化工具

这是一个至关重要的架构设计：**Gateway (主程序) 始终运行在宿主机上，而沙箱内仅承载“工具”执行。**

### 实验室类比
*   **Gateway 是“控制室”**：位于实验室外，透过“防弹玻璃”（Docker 隔离层）进行观察。它掌握着所有实验手册（配置）、门禁卡（API 密钥）及实验记录（记忆）。
*   **Sandbox 是“隔离实验柜”**：即 Docker 容器内部。起初是空的，仅预装了基础实验器材（Bash, Python）。

### 为什么 Gateway 不进入沙箱？
1.  **极端安全性 (Security)**：若 Gateway 运行在沙箱内，一旦 AI 执行恶意代码，即可直接读取内存并窃取 OpenAI/Anthropic 的 API Keys，或篡改核心认知文件。将 Gateway 留在宿主机，确保了即使沙箱被摧毁，控制室内的机密依然绝对安全。
2.  **持久性 (Persistence)**：沙箱容器具有生命周期（可能因超时、更新而销毁）。Gateway 运行在宿主机上，确保持久化数据库、配置连接及长连接渠道（WhatsApp/Slack）的稳定性。
3.  **资源开销优化**：避免为每个隔离环境重复运行冗余的网关逻辑，显著降低内存开销。

## 6. 指令通信流 (Command Flow)
当用户发送消息时，系统按以下闭环逻辑运作：
1.  **Gateway (宿主机)**：接收消息并投递给 LLM 推理。
2.  **LLM (决策)**：回复“我想运行一个 Python 脚本处理数据”。
3.  **Gateway (宿主机)**：解析到工具请求，调用宿主机的 Docker CLI。
4.  **指令投递**：Gateway 执行 `docker exec [容器名] python script.py`。
5.  **结果返回**：容器将执行结果实时吐回给宿主机网关。
6.  **感知闭环**：Gateway 将结果转发给 LLM，完成一次思考-执行循环。

## 7. 构建沙箱镜像 (Build Sandbox Images)

在启用沙箱功能前，需要在 **宿主机**（而非容器内）手动构建 Agent 任务运行所需的 Docker 镜像。

### 1. 构建基础沙箱镜像
该脚本将构建 `openclaw-sandbox:bookworm-slim` 基础镜像：
```bash
./scripts/sandbox-setup.sh
```

### 2. 构建增强型工具链（推荐）
如果你需要更完整的开发工具链（如 Node.js, Go, Rust 等），建议运行：
```bash
./scripts/sandbox-common-setup.sh
```

## 8. 镜像安全哲学
通过这种解耦设计，如果 AI 生成了具有破坏性的代码（如 `rm -rf /`），其破坏范围被严格限制在轻量级的 Debian 沙箱镜像内，而无法波及宿主机或 Gateway 核心程序。任务结束后，容器可被随时丢弃并从干净的镜像重新创建。

