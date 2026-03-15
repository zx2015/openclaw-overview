# 模型提供商配置 (Model Provider Configuration)

OpenClaw 支持通过 CLI 或直接编辑配置文件来接入各种 AI 模型提供商。本文档介绍如何配置常见的模型提供商。

## 1. 通过 CLI 配置模型提供商

`openclaw models configure` 是配置模型厂商适配器的推荐方式。

### 1.1 配置魔搭 (ModelScope) 提供商

魔搭 (ModelScope) 提供符合 OpenAI 兼容接口的推理服务。你可以使用以下命令进行配置：

```bash
openclaw models configure --provider modelscope \
  --base-url https://api-inference.modelscope.cn/v1 \
  --api-key 你的魔搭Token \
  --compatible openai
```

**参数说明：**
*   `--provider`: 指定提供商名称（如 `modelscope`）。
*   `--base-url`: 提供商的 API 基础地址。
*   `--api-key`: 你的访问令牌 (API Key)。
*   `--compatible`: 指定接口兼容类型（对于魔搭，请设置为 `openai`）。

## 2. 手动编辑配置文件

除了使用 CLI，你也可以直接编辑 `~/.openclaw/config.json` 文件来管理模型提供商。

在该文件的 `providers` 部分，你可以手动添加或修改配置：

```json
{
  "providers": {
    "modelscope": {
      "baseUrl": "https://api-inference.modelscope.cn/v1",
      "apiKey": "你的魔搭Token",
      "type": "openai"
    }
  }
}
```

## 3. 验证配置

配置完成后，你可以尝试通过 `openclaw models list` 查看已启用的模型列表，或直接启动智能体进行测试。
