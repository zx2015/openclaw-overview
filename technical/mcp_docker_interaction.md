# OpenClaw：Docker 容器中 Agent 使用全局 MCP 工具指南

在 OpenClaw 的 Docker 架构中，即使 Agent 运行在隔离的沙箱容器（Sandbox Container）中，它依然能够高效地使用宿主机或网关定义的“全局”MCP 工具。

---

## 1. 核心机制：Gateway “代行制”

OpenClaw 遵循 **“指挥与执行分离”** 的设计哲学。其工具调用逻辑如下：

*   **Gateway 作为中枢**：`openclaw-gateway` 容器负责管理所有的 MCP 连接、鉴权和工具调用逻辑。
*   **请求回传**：当沙箱中的 Agent 决定调用一个工具时，该请求会通过通信协议传回给宿主机的 Gateway。
*   **网关执行**：Gateway 在其自身环境（或其可访问的环境）中运行该工具，并将结果（Observation）返回给沙箱内的 Agent。

**结论**：由于工具是由 Gateway 运行的，Agent 所在的沙箱容器不需要安装该工具的依赖环境，只需拥有调用权限即可。

---

## 2. 两种“全局”工具的调用差异

### A. 配置在 `agents.defaults.tools.mcp` (继承制)
如果你将 MCP 服务器定义在配置文件的 `agents.defaults` 下：
*   **表现**：所有的 Agent（包括容器内的）都会默认继承并拥有调用该工具的权限。
*   **优点**：即插即用，无需为每个新 Agent 重复配置。
*   **注意**：需确保在 `agents.list.[id].tools.allow` 中没有显式禁用这些工具。

### B. 配置在 `tools.mcp.servers` (注册池制)
这种配置通常被视为全局注册池。
*   **调用方式**：只需在特定 Agent 的配置中，将这些已注册的工具 ID 添加到其 `allow` 列表中即可授权。

---

## 3. 关键要求与障碍排除

若要确保容器内的 Agent 顺利使用全局工具，需满足以下三个条件：

### 1. 网络互通 (Network)
如果 MCP 服务器是一个基于网络的服务（通过 URL 访问）：
*   必须确保 `openclaw-sandbox` 容器所属的 Docker 网络能够访问该 URL。

### 2. 文件访问权限 (File Access)
如果 MCP 工具是一个本地脚本（如 `command: "node", args: ["./myscript.js"]`）：
*   **执行方**：该脚本必须存在于 **Gateway 容器** 能访问到的路径下（通常是挂载的 `.openclaw` 目录）。
*   **无需拷贝**：Agent 所在的沙箱容器**不需要**拥有这个脚本文件。

### 3. 环境变量继承 (Env Vars)
*   由于工具由 Gateway 代为执行，必须确保工具所需的 API Key 或 Token 已在 `docker-compose.yml` 的 `openclaw-gateway` 服务中定义，否则执行会因缺少凭证而失败。

---

## 4. 操作建议与验证流程

### 第一步：在主配置中定义全局 MCP
```bash
docker compose run --rm openclaw-cli config set "tools.mcp.servers.global_tool" "{\"command\": \"...\"}"
```

### 第二步：为 Agent 授权
如果 Agent 未能自动继承，可手动开启：
```bash
docker compose run --rm openclaw-cli config set "agents.list.<index>.tools.allow" "[\"global_tool\"]"
```

### 第三步：使用诊断工具验证
运行 `sandbox explain` 是最有效的调试手段，它能展示当前 Agent 在沙箱环境下可见的所有工具清单：
```bash
docker compose run --rm openclaw-cli sandbox explain --agent my-agent
```

---

## 总结

OpenClaw 的 **Gateway 代行制** 极大地简化了 Docker 环境下的工具管理。你只需要在网关层维护一套工具链，即可让所有隔离的沙箱 Agent 共享这些能力，无需在每个容器中重复安装复杂的依赖。
