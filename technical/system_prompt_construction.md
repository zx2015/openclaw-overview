# OpenClaw System Prompt 构建机制

OpenClaw 的 System Prompt 构建是一个多层级、动态化的编排过程。其核心逻辑位于 `src/agents/system-prompt.ts`（通过 `buildAgentSystemPrompt` 实现），并由 `src/agents/cli-runner/helpers.ts` 中的 `buildSystemPrompt` 进行协调。

## 1. 核心构建组件

### 静态与半静态组件 (Core Structure)
系统首先加载确保 Agent 基础运行的指令集：
*   **基础协议 (Base Protocol)**：定义 Agent 的基本行为模式与交互规范。
*   **ACP 协议支持**：若配置启用 ACP，则注入标准化的协议操作指令。
*   **运行环境信息 (Runtime Info)**：由 `buildSystemPromptParams` 实时生成，包含：
    *   **操作系统**：(如 Linux 6.1.0, Darwin 23.4.0)。
    *   **Shell 类型**：(如 zsh, bash)。
    *   **Node.js 版本**与**系统架构** (x64/arm64)。
    *   **模型详情**：当前使用的模型名称及其厂商别名。

### 工作区文件注入 (Workspace Injection) —— **核心动态逻辑**
系统会动态加载用户工作区（`~/.openclaw/workspace/`）中的核心 Markdown 文件：
*   **`AGENTS.md`**：作为操作手册与进化日志。
*   **`SOUL.md`**：作为人格定义与行为边界。
*   **`USER.md`**：作为用户偏好与个人背景。
*   **`IDENTITY.md`**：作为身份标识与 UI 元数据。
*   **`TOOLS.md`**：作为工具使用的本地化备注。
*   **`BOOTSTRAP.md`**：新工作区的引导指令。

**注意**：系统具备 **截断保护机制**。若文件内容过长，会根据 `bootstrapMaxChars` 进行截断，并在 Prompt 中注入 `bootstrapTruncationWarningLines` 以提醒 AI 信息不完整。

### 时间与本地化感知 (Temporal Context)
*   **动态时间**：实时生成 `userTime`。
*   **时区与格式**：注入 `userTimezone` 和 `userTimeFormat`。
这确保了 Agent 能够准确理解“今天”、“下周五”或“凌晨三点”等相对时间概念。

### 工具与技能定义 (Tool Definitions)
*   **可用工具列表**：将当前会话（Run）允许调用的所有 `toolNames` 注入。
*   **TTS 提示词**：通过 `buildTtsSystemPromptHint` 指导 AI 生成适合语音合成的文本（如避免复杂的 Markdown 符号）。
*   **模型映射**：通过 `buildModelAliasLines` 处理不同厂商模型的特定别名。

### 诊断与安全护栏 (Guardrails)
*   **权限校验**：注入 `ownerNumbers`（如管理员 WhatsApp 号码）及显示策略。
*   **推理模式 (Think Level)**：根据配置决定是否启用推理模型特有的 `thinking` 模式。
*   **引用规范**：决定 AI 在检索长期记忆时是否必须提供来源引用（Citations）。

## 2. 底层拼接细节与逻辑结构 (Low-level Implementation)

`buildAgentSystemPrompt` 函数通过将各个模块化的字符串数组按顺序拼接，并最终使用 `.join("\n")` 合并，完成了 System Prompt 的最底层构建。

### 核心架构：基于数组的顺序拼接
系统使用 `lines` 字符串数组顺序收集所有片段，确保了指令的层级优先级：
*   **起始行**：`You are a personal assistant running inside OpenClaw.`
*   **模块化注入**：子模块（如 Tooling, Safety, Workspace）作为独立的数组块，通过 `lines.push(...)` 或展开操作符加入主数组。

### 工具定义模块 (## Tooling)
这是逻辑最复杂的模块之一：
*   **内置工具描述**：通过硬编码的 `coreToolSummaries` 映射为 `read`, `write`, `exec` 等核心工具提供标准化的功能描述。
*   **动态过滤与去重**：根据 `toolNames` 参数只保留当前会话授权的工具，并执行大小写不敏感的去重（Deduplication）。
*   **优先级排序**：通过 `toolOrder` 定义显示顺序，确保 AI 优先看到基础文件工具，最后看到复杂的 Agent 管理工具（格式为 `- 名称: 描述`）。

### 环境与沙箱上下文 (## Workspace & ## Sandbox)
*   **路径注入**：实时注入 `displayWorkspaceDir`。
*   **沙箱隔离指令**：若启用沙箱，会额外拼接关于 **Docker 隔离**、**路径映射**（Host vs Container 路径对比）以及 **提升权限 (Elevated exec)** 的详尽说明，防止 AI 尝试访问不存在的宿主机路径。

### 项目上下文与文件注入 (# Project Context) —— **关键拼接点**
这是工作区核心文件被“吞入”的地方：
*   **文件循环加载**：遍历 `contextFiles` 数组，为每个文件生成 `## 文件路径` 二级标题并紧跟其 `content`。
*   **Persona 强化**：若检测到 `SOUL.md`，会特意注入一句：`If SOUL.md is present, embody its persona and tone.`，以强化 AI 对性格设定的重视程度。

### 交互协议与约束
*   **静默回复 (## Silent Replies)**：注入 `SILENT_REPLY_TOKEN` 指令，规范 AI 在无需回复时的行为。
*   **心跳机制 (## Heartbeats)**：指导 AI 如何识别并响应 `HEARTBEAT_OK`。
*   **渠道标签 (## Reply Tags)**：针对 Telegram, Signal 等不同消息渠道注入 `[[reply_to_current]]` 等交互指令。

### 运行时元数据 (## Runtime)
Prompt 的最后一行由 `buildRuntimeLine` 生成，作为当前环境的极简快照（利用 Recency Bias 效应）：
`1 Runtime: agent=... | host=... | os=... | node=... | model=... | thinking=...`

## 3. 灵活的构建模式：PromptMode
函数支持三种模式以平衡能力与 Token 消耗：
*   **`full`**：全量模式，包含身份、自我更新、静默回复等所有章节。
*   **`minimal`** (子代理常用)：跳过身份设定、自我更新等冗余章节，仅保留核心工具、工作区信息及基础运行环境。这有助于在嵌套调用中大幅节省 Token。
*   **`none`**：仅保留最底层的运行环境快照。

## 4. 底层实现亮点 (Implementation Highlights)
*   **路径脱敏 (Sanitization)**：使用 `sanitizeForPromptLiteral` 过滤路径中的特殊字符，防止破坏 Prompt 结构。
*   **高度灵活性**：通过大量的条件判断（如 `if (isMinimal)`, `if (sandboxedRuntime)`），使同一个构建函数能生成完全不同用途（如主代理 vs 沙箱子代理）的 Prompt。

## 5. 组装顺序 (Assembly Order)

Prompt 的组装遵循严密的逻辑顺序，以确保 AI 的注意力聚焦：
1.  **[身份与本质]**：你是谁（`IDENTITY`, `SOUL`）。
2.  **[用户上下文]**：你为谁服务（`USER`）。
3.  **[操作环境]**：你在哪运行（Runtime Info, Shell, Path）。
4.  **[核心准则]**：你应该怎么做（`AGENTS.md`, Rules）。
5.  **[工具说明]**：你能用什么（Tools, `TOOLS.md`, Skills）。
6.  **[当前状态]**：现在是什么时间，什么上下文（Time, Extra System Prompt）。

## 3. 核心代码逻辑参考 (`src/agents/cli-runner/helpers.ts`)

```typescript
export function buildSystemPrompt(params: { ... }) {
  // 1. 获取动态运行参数（OS, Shell, Time等）
  const { runtimeInfo, userTimezone, userTime, userTimeFormat } = buildSystemPromptParams({ ... });

  // 2. 获取 TTS 指令提示（如果适用）
  const ttsHint = params.config ? buildTtsSystemPromptHint(params.config) : undefined;

  // 3. 调用核心组装函数执行拼接
  return buildAgentSystemPrompt({
    workspaceDir: params.workspaceDir,
    // 注入 AGENTS.md / SOUL.md / USER.md 等上下文文件内容
    contextFiles: params.contextFiles, 
    runtimeInfo,
    toolNames: params.tools.map((tool) => tool.name),
    // ... 其他动态参数
  });
}
```

## 4. 总结
OpenClaw 的 System Prompt 是其 **“意识流”** 的起点。通过将底层协议、用户定义的长期记忆文件以及实时的物理环境参数进行有机缝合，系统成功地将一个通用的 LLM 转化为了一个具备本地执行力、拥有独立人格且深度理解用户的 **“龙虾助手”**。
