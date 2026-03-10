# OpenClaw 构建、发布与质量保证

OpenClaw 拥有一个高度自动化且严谨的工程体系，确保了其在多端环境下的高可靠性。

## 1. 测试体系 (Testing Pipeline)
项目通过全方位的自动化测试来覆盖不同的使用场景：
*   **单元测试 (Unit Tests)**：对核心逻辑（如 ACP 协议解析、模型 Provider 适配）进行颗粒度极细的逻辑校验。
*   **端到端测试 (E2E Tests)**：模拟真实的用户输入，验证从网关到 Agent 再到工具调用的完整闭环。
*   **Live 测试**：通过真实的 WhatsApp、Telegram、Discord 账户进行实机通信测试，确保各渠道适配器在真实网络环境下的稳定性。
*   **Docker 回归测试**：利用容器化环境模拟纯净的操作系统，自动验证各种 Bash 脚本在受控沙箱中的执行结果。

## 2. 构建与分发 (Build & Distribution)
`scripts/` 目录中集成了大量复杂的跨平台构建逻辑：
*   **Node.js 编译**：将 TypeScript 源代码编译为高性能的 ESM (type: module) 代码，并进行混淆与压缩。
*   **多端原生构建**：
    *   **Android**：支持通过脚本自动生成、签名并优化 APK 安装包。
    *   **iOS**：自动处理 Xcode 工程的配置、签名并导出对应的 IPA 包。
    *   **macOS**：支持将网关与 UI 封装为标准的 macOS App 应用包。
*   **Docker 镜像**：提供官方维护的容器镜像，便于服务器或私有云环境的一键部署。

## 3. 部署架构支持
OpenClaw 内置了灵活的部署选项：
*   **本地运行**：直接在个人电脑（macOS/Linux/Windows WSL2）上运行。
*   **移动端运行**：通过 iOS 和 Android 原生应用内嵌的 Node.js 环境实现真正的离线运行。
*   **云端与私有隧道**：原生集成 Tailscale，支持将本地网关以零配置的方式安全映射到公网。
