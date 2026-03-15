# 自定义 Provider 开发指南

本指南详细介绍了如何在 OpenClaw (龙虾) 中新增一个自定义的模型提供商 (Provider)。

## 架构概览

OpenClaw 的 Provider 并非通过传统的插件钩子注入，而是作为 **Gateway (网关)** 的 HTTP 路由处理器实现的。网关负责接收外部请求，将其转换为 OpenClaw 的内部指令 (`agentCommand`)，并在执行完成后将结果格式化回外部协议。

## 核心实现步骤

### 1. 创建处理器
在 `src/gateway/` 目录下创建一个新的 TypeScript 文件（例如 `my-custom-http.ts`）。

**核心职责：**
- **鉴权检查**：验证请求头中的 API Key。
- **协议转换**：将外部请求的 JSON Body 转换为 `agentCommand`。
- **流式响应处理**：如果支持流式输出 (SSE)，需处理 `AssistantStreamDeltaText` 事件。

### 2. 注册路由钩子
在 `src/gateway/server-http.ts` 中，将新的 URL 路径映射到你的处理器。

```typescript
// src/gateway/server-http.ts
import { handleMyCustomHttpRequest } from "./my-custom-http.js";

// 在 handleHttpRequest 逻辑中：
if (req.url?.startsWith("/v1/my-custom-endpoint")) {
  return await handleMyCustomHttpRequest({ ...options });
}
```

### 3. 模型注册 (可选)
如果需要让 OpenClaw 的其他组件（如 WebUI 或管理命令）感知到新 Provider 的模型，请在 `src/agents/model-catalog.ts` 中添加对应的元数据。

## 实战参考

- **OpenAI 兼容实现**：`src/gateway/openai-http.ts`。这是最通用的实现参考。
- **OpenResponses 实现**：`src/gateway/openresponses-http.ts`。展示了如何处理更复杂的交互协议。

## 相关建议

- **优先选择 OpenAI 兼容层**：如果你的自定义 Provider 能够提供 OpenAI 兼容的 API，建议直接通过配置文件修改 `baseUrl` 指向它，而无需修改源码。
- **安全检查**：在处理器中务必调用 `authorizeHttpGatewayConnect` 确保请求来源合法。

## Related
- [架构概览](../technical/architecture.md)
- [网关核心模块](../technical/gateway_core.md)
