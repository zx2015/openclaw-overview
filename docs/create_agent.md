# 如何新建 OpenClaw 智能体 (Agent)

新建 Agent 是扩展 OpenClaw 能力边界的核心操作。你可以通过 CLI 交互式命令、手动修改配置文件或克隆现有工作区三种方式来实现。

## 方法一：使用命令行工具 (最推荐)

OpenClaw 提供了直观的子命令来管理 Agent 生命周期。

1.  **查看现有列表**：确认 ID 避免冲突。
    ```bash
    openclaw agents list
    ```
2.  **新建 Agent**：
    ```bash
    openclaw agents add my-new-agent
    ```
    对于 **Docker 环境** 用户，应通过 `openclaw-cli` 服务执行命令：
    ```bash
    docker compose run --rm openclaw-cli agents add --name "my-agent" --workspace "/home/node/.openclaw/workspace/my-agent"
    ```
    关于详细的 CLI 参数、交互向导以及 Docker 专用配置，请参考 **[如何配置智能体 (Agents)](./configure_agent.md)**。

    *注：若 `add` 命令在当前版本不可用，可运行 `openclaw onboard` 重新进入引导流程。*
3.  **初始化工作区**：首次运行该 Agent 时，系统会自动在 `~/.openclaw/workspace/my-new-agent` 目录下生成 `AGENTS.md`、`SOUL.md` 等核心文件模板。

---

## 方法二：手动修改 `config.json` (开发者首选)

若需精准控制模型选型、工作区物理路径及工具权限，建议直接编辑配置文件（通常位于 `~/.openclaw/config.json`）。

1.  **编辑配置**：在 `agents.definitions` 下添加新条目。
    ```json
    {
      "agents": {
        "definitions": {
          "researcher": {
            "model": "anthropic/claude-3-5-sonnet",
            "workspace": "~/.openclaw/workspaces/researcher",
            "tools": {
              "profile": "coding"
            }
          }
        }
      }
    }
    ```
    *   **`researcher`**：自定义的 `agentId`。
    *   **`model`**：指定该 Agent 调用的后端模型。
    *   **`workspace`**：指定其记忆与教训存放的独立物理目录。
    *   **`tools.profile`**：设定工具集方案（如 `minimal`, `coding`, `messaging`, `full`）。

2.  **应用变更**：保存后，若 Gateway 正在运行，通常会自动触发热重载。若未生效，请手动重启：
    ```bash
    openclaw gateway restart
    ```

---

## 方法三：通过工作区克隆 (快速创建)

适用于需要快速创建一个“孪生兄弟”并微调性格的场景。

1.  **复制工作区目录**：
    ```bash
    cp -r ~/.openclaw/workspace/default ~/.openclaw/workspace/my-clone
    ```
2.  **微调灵魂**：直接编辑新目录下的 `SOUL.md` (修改语气) 和 `IDENTITY.md` (修改头像/Emoji)。
3.  **注册 ID**：参考“方法二”，在 `config.json` 中将新 ID 指向该克隆路径。

---

## 新建完成后的测试

新建完成后，你可以立即通过以下方式激活你的 Agent：

*   **终端直接测试**：
    ```bash
    openclaw run --agent researcher "你好，请介绍一下你的专业领域。"
    ```
*   **在社交应用中切换**：
    在 WhatsApp、Telegram 或 Slack 中发送：
    `/agent researcher`

## 总结：新建 Agent 的三要素
1.  **ID**：唯一的绰号（用于检索与映射）。
2.  **Model**：执行推理的“大脑”。
3.  **Workspace**：存放记忆与教训的独立“房间”。

一旦三要素确定，OpenClaw 就会自动完成剩余的初始化工作，你的新 Agent 即可上线服务。
