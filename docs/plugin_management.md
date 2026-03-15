# OpenClaw 插件管理 (Plugin Management)

OpenClaw 允许通过插件 (Plugins) 扩展其核心功能，如接入新的通讯渠道、集成外部服务或添加自定义 AI 技能。插件管理主要通过 `openclaw plugins` 命令组完成。

## 1. 查看插件列表 (List)

你可以使用以下命令查看系统中已安装或已发现的插件：

```bash
openclaw plugins list
```

*   **状态说明**：该命令会显示每个插件的 ID、版本以及当前状态（`enabled` 或 `disabled`）。
*   **JSON 输出**：如果需要将结果用于自动化脚本，可以使用 `--json` 参数。

---

## 2. 启用与禁用插件 (Enable/Disable)

新安装的插件通常默认处于禁用状态，需要手动启用后才能生效。

### 启用插件
```bash
openclaw plugins enable <plugin-id>
```

### 禁用插件
```bash
openclaw plugins disable <plugin-id>
```

> **注意**：启用或禁用操作通常修改的是配置文件。为了使变更生效，你可能需要重启网关服务：
> ```bash
> openclaw gateway restart
> ```

---

## 3. 卸载插件 (Uninstall)

如果你不再需要某个插件，可以使用以下命令将其从系统中彻底移除：

```bash
openclaw plugins uninstall <plugin-id>
```

### 常用可选项：
*   **`--keep-files`**：仅从配置中移除插件记录，但保留磁盘上的插件源码/文件。
*   **`--dry-run`**：模拟卸载过程，显示将要执行的操作但不实际删除任何文件。

---

## 4. 其它常用插件命令

*   **安装插件**：`openclaw plugins install <npm-package|path>`
*   **查看详情**：`openclaw plugins info <plugin-id>`
*   **故障排查**：`openclaw plugins doctor` (用于检测插件冲突或加载失败的原因)
*   **更新插件**：`openclaw plugins update <plugin-id>`

---

## 总结

插件是 OpenClaw 保持灵活性的核心。通过 `openclaw plugins` 命令组，你可以轻松地构建适合自己工作流的定制化 AI 运行时环境。
