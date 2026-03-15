# OpenClaw 技能 (Skills) 加载与执行机制

技能 (Skills) 是 OpenClaw 扩展能力的核心单元。系统通过动态加载机制，将不同来源的技能注入到 Agent 的工具箱中。

---

## 1. 技能加载机制

根据 `buildWorkspaceSkillStatus` 函数的逻辑，系统会扫描以下三个位置来加载技能：

*   **内置技能 (Bundled Skills)**：OpenClaw 核心代码自带的技能（如 `google-search`, `read-file`）。
*   **已管理技能 (Managed Skills)**：用户安装在 `~/.openclaw/skills` 目录下的全局技能。
*   **工作区技能 (Workspace Skills)**：存放在 Agent 独立工作区（`workspaceDir`）下的特定技能，实现 Agent 能力的个性化定制。

---

## 2. 沙箱 (Docker) 环境下的执行逻辑

当 Agent 运行在 Docker 沙箱中时，技能的执行方式取决于其类型：

### A. API 类技能 (如 `google-search`)
*   **调用方式**：**Gateway 代行**。
*   **逻辑**：这些技能由 Gateway 统一管理。沙箱中的 Agent 请求调用时，Gateway 在其自身环境执行并将结果返回给沙箱。
*   **支持情况**：无缝支持，无需在沙箱容器中做任何配置。

### B. 工具类/脚本技能 (如 `git`, `python-script`)
*   **调用方式**：**容器内执行**。
*   **逻辑**：如果技能需要调用本地二进制文件（如 `python3`, `git`, `ripgrep`），OpenClaw 会检查环境依赖（Requirements）。
*   **依赖要求**：沙箱容器必须预装这些二进制工具。
*   **镜像支持**：`scripts/sandbox-setup.sh` 构建的 `openclaw-sandbox:bookworm-slim` 镜像已预装了 `python3`, `git`, `jq`, `ripgrep` 等常用工具。

### C. MCP 转化为的技能
*   **调用方式**：**Gateway 转发**。
*   **逻辑**：参考 [Docker 环境与 MCP 交互](./mcp_docker_interaction.md)，由 Gateway 转发给 MCP Server 执行。
*   **支持情况**：无缝支持。

---

## 3. 配置与授权

### 全局默认开启
在 `openclaw.json` 的 `agents.defaults.skills` 中配置默认允许的技能列表。

### Agent 特定授权
确保目标技能在 Agent 的 `skills.allow` 列表中：
```bash
docker compose run --rm openclaw-cli config set "agents.list.<index>.skills.allow" "[\"my-skill\"]"
```

---

## 4. 诊断与验证

### 验证就绪状态 (Eligible)
使用以下命令查看特定 Agent 的技能是否可用：
```bash
docker compose run --rm openclaw-cli skills check --eligible
```
*   如果 `eligible: false`，通常是因为沙箱容器缺少该技能声明的 `bin` 依赖（例如容器未安装 `ffmpeg`，但技能需要它）。

### 综合诊断
使用 `sandbox explain` 查看 Agent 在沙箱环境下能看到的完整工具/技能清单：
```bash
docker compose run --rm openclaw-cli sandbox explain --agent <name>
```

---

## 5. 总结对比

| 类别 | 调用方式 | 容器内 Agent 支持情况 |
| :--- | :--- | :--- |
| **API 类 Skill** | Gateway 代行 | 无缝支持 |
| **工具类 Skill** | 容器内执行 | 支持，但需镜像预装对应工具 |
| **MCP 转化类 Skill** | Gateway 转发 | 无缝支持 |

---

## 建议
对于沙箱 Agent，全局技能默认可用。若发现某个技能无法使用，请优先通过 `sandbox explain` 检查容器环境是否满足其二进制依赖要求。
