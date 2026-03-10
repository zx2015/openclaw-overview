# IDENTITY.md: 身份元数据与 UI 标识 (Name/Icon)

`IDENTITY.md` 是 OpenClaw 智能体工作区的“名片”，主要用于系统展示。

## 1. 核心定位
*   **维度**：**Name/Icon** (我的名片)
*   **角色**：Agent 的身份、名称及 UI 渲染元数据。

## 2. 具体用途
*   **基础标识**：定义 Agent 的显示名称、头像标识 (Emoji) 以及主色调。
*   **视觉风格**：控制 Agent 在 Control UI（控制中心）中的呈现方式。
*   **极简化身份**：相比 `SOUL.md` 的深度性格定义，`IDENTITY.md` 主要承担对外展示任务。

## 3. 修改触发场景 (Update Triggers)
*   **首次入职触发 (Initial Onboarding)**：在 `onboarding` 过程中设定的名称和 Emoji 会被直接写入。
*   **品牌重塑触发 (Rebranding Request)**：用户直接指令更改身份：*“从现在起，你的名字叫‘管家阿尔弗雷德’，头像换成 🎩。”*
*   **控制中心同步触发 (Control UI Sync)**：在可视化界面修改后，系统后端会将变更写回此文件实现持久化。

## 4. Agent 思考逻辑
> “用户希望我叫什么，用什么形象出现。”

## 5. 技术逻辑
在入职引导 (Onboarding) 阶段，系统会自动生成此文件，并将其用于多渠道（如 WhatsApp 账号名称、Discord 机器人昵称）的展示。
