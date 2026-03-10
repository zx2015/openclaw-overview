# OpenClaw Agent 实战能力与安全

OpenClaw 的核心执行层 (Agent Layer) 位于 `src/agents/`，赋予了 AI 真正的生产力。

## 1. 实战执行力 (Action-Oriented)
OpenClaw 并非一个简单的对话模型，它具备深度的系统交互能力：
*   **Bash & Shell 执行**：通过 `bash-tools.ts` 和 `bash-tools.exec.ts` 组件，AI 可以直接在宿主机上运行 Shell 脚本。
*   **Docker 隔离**：支持将高风险的执行操作引导至 Docker 容器内，实现资源隔离与安全保护。
*   **审批流机制 (exec-approval)**：在执行可能引发数据丢失或安全风险的命令前，系统会通过 IM 渠道向用户请求实时确认。

## 2. 多代理协作体系 (Multi-Agent)
*   **子代理注册表 (Subagent Registry)**：系统支持动态加载并编排多个专业化的子代理（如搜索专家、代码专家、系统管理员）。
*   **任务分发**：主代理可以根据需求将复杂目标拆解，并委派给特定的子代理协作完成。

## 3. 智能上下文管理 (Context Management)
*   **消息压缩 (Compaction)**：利用 AI 自主总结历史对话，定期对上下文进行逻辑压缩，平衡记忆力与 Token 成本。
*   **长期记忆搜索 (Memory Search)**：结合 `sqlite-vec` 向量插件，在用户提问时自动检索海量历史记忆。
*   **上下文守卫 (Context Window Guard)**：针对超长对话，系统会智能移除低权重的冗余信息，保护对话的连续性。

## 4. 隔离沙箱 (Sandbox)
*   **代码运行环境**：`sandbox/` 目录通过虚拟化或受控执行路径，为 AI 执行的动态代码片段提供安全围栏，防止恶意注入或误操作影响主系统。
