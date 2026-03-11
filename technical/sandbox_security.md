# OpenClaw 沙箱：文件系统挂载与安全加固

安全加固是沙箱机制的生命线，OpenClaw 通过多层级的权限收敛确保宿主系统万无一失。

## 1. 目录挂载策略与文件可见性 (Visibility Logic)

网关根据任务需求动态映射主机目录，并严格遵循最小可见性原则：

### `/workspace`：当前任务主工作区
*   **映射逻辑**：若 `agents.defaults.sandbox.workspaceAccess` 设置为 `rw` 或 `ro`，Gateway 会将宿主机的工作区目录挂载至此。
*   **可见性**：AI 在沙箱内执行 `ls /workspace` 即可看到该项目下的所有工程文件。

### `/agent`：智能体私有目录
*   **映射逻辑**：每个 Agent 拥有的私有配置目录（存放 `SKILL.md` 等）会被挂载至此路径。
*   **典型内容**：AI 可以在 `/agent/SKILL.md` 中读取该技能的特定指令。

### 敏感文件隔离：AGENTS.md 与 SOUL.md
这是系统安全设计的关键细节：
*   **注入而非拷贝**：`AGENTS.md` 和 `SOUL.md` 由 Gateway 在宿主机读取，并转化为 LLM 的 System Prompt。它们**默认不会**被拷贝或挂载到沙箱环境中。
*   **隔离目的**：防止沙箱内的第三方脚本通过符号链接 (Symlink) 等手段篡改 AI 的“核心大脑”或窃取人格定义。
*   **分工哲学**：大脑（LLM）需要知道“我是谁”和“我的规矩”，而手脚（沙箱工具）只需执行具体任务，无需感知“灵魂”元数据。

## 2. 系统级硬化 (Hardening)
... Applied fuzzy match at line 1-3.
*   **网络隔离**：默认为 `network: "none"`，彻底切断 AI 向外发送数据的物理路径。
*   **权限防御**：启用 `no-new-privileges`，防止 AI 通过漏洞进行 SUID 提权。
*   **能力收敛 (`cap-drop`)**：丢弃所有非必要的 Linux 内核 Capability（如内核模块加载、网络管理等）。
*   **内核过滤**：应用 Seccomp 与 AppArmor 配置文件，拦截危险的系统调用。

## 3. “提权”协作流 (Elevated Exec)
当 AI 发现任务超出沙箱权限（如需访问宿主机硬件）时：
1.  **发送请求**：AI 通过特定的 `/elevated ask` 指令发起申请。
2.  **审批经理 (`ExecApprovalManager`)**：网关层拦截该请求，并在用户端展示完整的命令。
3.  **用户裁决**：只有在用户输入 `/approve` 或手动点击批准后，指令才会在宿主机环境执行。

## 4. 路径转换器 (Path Sanitizer)
为了防止 AI 利用路径字符串绕过挂载点，系统内置了 Sanitizer 逻辑：
*   **规范化**：自动将 `../` 等相对路径展开并校验是否超出 `/workspace` 边界。
*   **翻译**：负责在宿主机路径与容器路径之间进行透明转换。
