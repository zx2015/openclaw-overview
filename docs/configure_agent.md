# 如何配置 OpenClaw 智能体 (Agents)

配置一个新的 Agent 是扩展 OpenClaw 能力的关键。你可以通过交互式向导或非交互式参数来完成。

## 1. 交互式向导（推荐）

只需运行以下简单命令，系统会引导你输入名称、工作空间和模型配置：

```bash
openclaw agents add
```

或者如果你想直接指定名字进入向导：

```bash
openclaw agents add "我的新智能体"
```

执行 `openclaw agents add` 后，你可以按照提示：
1. **输入名字**。
2. **确认工作空间路径**。
3. **选择是否现在配置模型和鉴权**（对应 OpenAI/Claude 等接口）。
4. **选择是否绑定渠道**（如 Telegram/Discord）。

---

## 2. 非交互式（直接通过参数创建）

如果你想在脚本中或一次性完成创建，可以使用以下格式：

```bash
openclaw agents add "my-agent" --workspace "/path/to/workspace" --model "gpt-4o"
```

> **注**：在非交互模式下，**智能体名称**（位置参数）和 `--workspace` 是必填项。

### 参数说明：
*   **`<name>`**: 智能体的显示名称（如 "my-agent"），直接跟在 `add` 后面，不需要 `--name`。系统会自动将其转换为小写、连字符分隔的 `agentId`。
*   **`--workspace`**: 智能体的工作目录（必须使用绝对路径）。
*   **`--model`**: 使用的模型 ID（如 `gpt-4o`, `deepseek-chat`）。
*   **`--bind`**: 绑定渠道（例如 `--bind "telegram:123456"` 将该智能体绑定到特定的 Telegram 账号）。
*   **`--json`**: 以 JSON 格式输出创建结果。

---

## 3. 为 Agent 配置沙箱 (Sandbox)

要为一个新创建的智能体配置沙箱，目前最标准的做法是采用 **“两步走”** 策略：

### 第一步：新建智能体
使用交互式向导或非交互命令创建智能体：
```bash
openclaw agents add "my-agent" --workspace "/root/my-project" --model "gpt-4o"
```

### 第二步：为其配置沙箱
智能体创建后，使用 `config set` 命令为其开启沙箱。OpenClaw 遵循 **“全局默认 -> Agent 覆盖”** 的继承逻辑。

#### 1. 配置路径与优先级
根据 `openclaw sandbox explain` 的诊断提示，沙箱开关配置遵循以下优先级：
1.  **Agent 级覆盖 (最高优先级)**：`agents.list[].tools.sandbox.enabled` 或 `agents.list[my-agent].sandbox.mode`
2.  **全局级默认**：`tools.sandbox.enabled` 或 `agents.defaults.sandbox.mode`

#### 2. 通过命令行开启（示例）
假设新创建的 Agent ID 为 `isolated-bot`：

```bash
# 方式 A：为该特定 Agent 开启沙箱开关
openclaw config set agents.list[id=isolated-bot].tools.sandbox.enabled true --json

# 方式 B：配置沙箱运行模式 (all, non-main 或 off)
openclaw config set agents.list[id=isolated-bot].sandbox.mode all
```

#### 3. (可选) 配置沙箱权限与镜像
```bash
# 配置沙箱工作区访问权限 (rw, ro 或 none)
openclaw config set agents.list[id=isolated-bot].sandbox.workspaceAccess rw

# 指定 Docker 镜像
openclaw config set agents.list[id=isolated-bot].sandbox.docker.image python:3.11-slim
```

### 进阶：如果你想一步到位
由于 OpenClaw 的配置本质上是修改 `openclaw.json`，你可以编写一个复合命令：

```bash
openclaw agents add "my-agent" --workspace "/root/my-project" && \
openclaw config set agents.list[my-agent].tools.sandbox.enabled true
```

> **为什么不支持在 add 时直接配置？**
> 这是因为 OpenClaw 的设计哲学是将 “**逻辑身份（Agent）**” 与 “**执行策略（Sandbox/Security）**” 解耦。`add` 命令负责建立身份，而 `config` 负责精细化的策略调整。

### 总结建议
1. 运行 `openclaw agents add` 完成基础创建。
2. 运行 `openclaw config set agents.list[智能体ID].tools.sandbox.enabled true` 开启沙箱。
3. **全局默认设置**：如果你有大量智能体需要沙箱，建议设置全局默认值：
   ```bash
   openclaw config set tools.sandbox.enabled true
   # 或
   openclaw config set agents.defaults.sandbox.mode all
   ```
   这样以后新建的所有智能体都会自动继承这个沙箱配置。

---

## 4. 删除智能体 (Delete Agent)

要删除一个智能体，你可以使用 `openclaw agents delete` 命令。

### 1. 交互式删除（有二次确认）
只需提供智能体的 ID（即你在创建时给它的名字）：
```bash
openclaw agents delete "x-man"
```
系统会提示你确认是否删除该智能体及其关联的工作区和状态文件。

### 2. 强制删除（跳过确认）
如果你在脚本中使用，或者确定要直接删除，可以使用 `--force` 参数：
```bash
openclaw agents delete "x-man" --force
```

### 注意事项：
*   **不可恢复**：删除操作会将智能体的配置、工作区目录、私有目录（`agentDir`）以及会话记录（`sessionsDir`）移动到回收站或直接清理，请谨慎操作。
*   **主智能体保护**：默认的 `main` 智能体（`DEFAULT_AGENT_ID`）通常受保护，无法被删除。
*   **ID 归一化**：如果你输入的是 `X-Man`，系统会自动将其归一化为 `x-man` 进行匹配。

---

## 5. Docker 环境下的 CLI 操作

在 Docker 部署环境下，所有的 CLI 操作应当通过 `openclaw-cli` 服务容器来执行。

### 1. 使用命令行新建 Agent
运行以下命令新建一个名为 `my-agent` 的 Agent：

```bash
docker compose run --rm openclaw-cli agents add \
  --name "my-agent" \
  --workspace "/home/node/.openclaw/workspace/my-agent" \
  --non-interactive
```
> **注**：`/home/node/.openclaw/workspace` 是 Docker 内部的默认工作区挂载点。

### 2. 配置 Agent 启用 Docker 沙箱
你可以通过命令行开启（假设该 Agent 是列表中的第 2 个，索引为 1）：

```bash
docker compose run --rm openclaw-cli config set "agents.list.1.sandbox.mode" "all"
```
> **注**：`mode` 可选值为 `off`（关闭）、`non-main`（仅非主会话开启）、`all`（全部开启）。

### 3. 绑定频道（可选）
若需该 Agent 接收特定消息（如来自 Telegram），需进行绑定：

```bash
docker compose run --rm openclaw-cli agents bind --agent my-agent --channel telegram
```

### 4. 运行与生效
完成上述配置后，重启 OpenClaw Gateway 容器以应用配置：

```bash
docker compose restart openclaw-gateway
```

---

## 核心命令总结 (Docker)

| 操作 | 命令 |
| :--- | :--- |
| **创建 Agent** | `docker compose run --rm openclaw-cli agents add <name>` |
| **开启沙箱** | `docker compose run --rm openclaw-cli config set "agents.list.<index>.sandbox.mode" "all"` |
| **查看状态** | `docker compose run --rm openclaw-cli agents list --bindings` |
| **重启网关** | `docker compose restart openclaw-gateway` |

## 总结

创建完成后，你可以通过以下命令来查看它的详细配置：

```bash
openclaw config get agents.list[my-agent]
```

你也可以随时前往该 Agent 的工作空间目录，通过编辑 `SOUL.md` 或 `AGENTS.md` 来进一步微调它的行为。
