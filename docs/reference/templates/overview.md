# OpenClaw 智能体工作区 (Agent Workspace) 模板参考

在 `docs/reference/templates/` 目录下（或运行时的 `~/.openclaw/workspace/`），OpenClaw 定义了构成 AI “长期记忆”和“行为准则”的五个核心文件。这些文件在系统启动或 Agent 运行时被注入到 System Prompt 中。

## 1. 核心文件总结对照表

| 文件 | 核心维度 | 核心触发场景 | Agent 思考逻辑 |
| :--- | :--- | :--- | :--- |
| **[`AGENTS.md`](./agents.md)** | **How** | 学习新技巧、避坑记录、工作流变更 | “下次遇到这类事，我得这么办。” |
| **[`MEMORY.md`](./memory_root.md)** | **What** | 识别长期事实、项目决策、提炼每日日志 | “这些重要事实需要被永久记录。” |
| **[`SOUL.md`](./soul.md)** | **Who** | 性格设定、语气偏好、道德边界变更 | “我是谁，我说话该是什么范儿。” |
| **[`TOOLS.md`](./tools.md)** | **How with What** | 发现本地硬件/服务、解决工具报错 | “在这个环境下，这个工具得这么用。” |
| **[`IDENTITY.md`](./identity.md)** | **Name/Icon** | 入职设置、名字/头像变更、UI 同步 | “用户希望我叫什么，用什么形象。” |
| **[`USER.md`](./user.md)** | **For Whom** | 记录用户名字、偏好、习惯、安全授权 | “我的主人是谁，他有什么特别要求。” |

## 2. 写入与更新触发场景 (Update Triggers)
...
### 4. 核心 Prompt 执念 (Persistence Instruction)
在 OpenClaw 的 System Prompt 中，有一段关键指令确保了知识的自动沉淀：
> "When you learn something significant about the user, the environment, or yourself, update the corresponding workspace file (.md) to persist this knowledge."

## 3. 注入与更新机制
... Applied fuzzy match at line 1-3.

### 自动化“学习与记忆” (Learned Lessons)
*   **触发条件**：Agent 在执行任务中“学到了教训”（Learn a lesson），例如发现原有的命令报错、找到了更高效的替代方案或避开了某个系统坑点。
*   **核心行为**：Agent 主动调用 `write_file` 工具更新 **`AGENTS.md`**，实现跨 Session 的行为进化。

### 用户明确指令 (Direct User Request)
*   **触发条件**：用户直接要求 Agent 记住新规则或更改性格。
    *   *示例*：“以后所有代码必须使用 TypeScript，请记入 `AGENTS.md`。”
    *   *示例*：“你的说话风格要更像 C-3PO，更新你的 `SOUL.md`。”
*   **核心行为**：Agent 定位到工作区对应文件并执行修改。

### 系统初始化与引导 (Onboarding & Setup)
*   **触发条件**：首次运行、执行 `openclaw reset` 或清空工作区后。
*   **核心行为**：系统从 `src/agents/workspace-templates/` 读取默认模板，并根据用户的初始回答填充 **`USER.md`** 和 **`SOUL.md`**。

### 子代理 (Subagent) 与沙箱协作
*   **触发条件**：在多代理协作模式下，主代理为子代理准备特定的指南，甚至可能生成临时的 **`AGENTS.md`** 传递给沙箱内的子代理。

## 3. 注入与更新机制
*   **注入机制**：系统读取工作区文件并根据 `bootstrapMaxChars`（默认 20,000）进行截断，最后拼接到 System Prompt 开头。
*   **动态进化**：Agent 被赋予了修改工作区文件的权限，可以根据交互学习自动更新 `AGENTS.md` 或根据指令调整 `SOUL.md`。
*   **版本控制**：在 Git 模式下，工作区文件的变更会被自动追踪，确保 AI 记忆演化的透明性。

## 3. 模板渲染与开发模式
*   **开发模式模板**：系统提供 `.dev.md` 后缀的预设模板（如 `SOUL.dev.md`），用于快速初始化特定性格的 Agent。
*   **引导生成**：在 Onboarding 阶段，系统会引导用户完成初始身份与偏好设定。
