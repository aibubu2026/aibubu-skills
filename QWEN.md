# Aibubu 图片和视频生成

当用户要求用 Aibubu 生成图片或视频、列出 Aibubu 模型、查询模型参数或价格、查询 Aibubu
余额、上传参考文件或查询生成任务时，使用 Aibubu。

本扩展连接远程 MCP 服务 `https://aibubu.cn/mcp`（Streamable HTTP）。如果服务要求登录，
让用户在交互会话中运行 `/mcp`，在对话框里完成浏览器授权；不要让用户把 API Key 贴进聊天。
工具列表能显示不代表已授权，用一次真实的 `get_balance` 调用确认。

## 规则

1. 模型 ID 只能从 `list_models` 获取，不要凭记忆写模型名。
2. 只传 `get_model` 返回的参数 schema 中声明过的字段，不要把一个模型的参数照搬到另一个模型。
3. 调用会消耗积分的工具（`generate_image`、`generate_video`、`generate_batch`、`upscale_image`）
   之前，先调用 `quote`（或 `quote_batch`、`quote_upscale`）和 `get_balance`，把模型、关键参数和
   预计积分告诉用户；批量或高额估价需用户确认；余额不足时停止并说明。
4. 生成是异步的：只提交一次，记下 `task_id`，用 `wait_task` 或 `get_task` 轮询，不要重复提交来查进度。
   提交时带 `idempotency_key`，网络重试时沿用；丢失 `task_id` 时用 `list_tasks` 找回。
5. 参考图、视频、音频要先通过 `open_uploader`、`upload_image`、`upload_file` 登记，再用 `asset_id` 引用。
6. 结果链接会过期，任务成功后及时下载。
7. 工具调用被拒绝后，遵循 Qwen Code 正常的权限流程，不要切换传输方式或扩大权限来绕过。

使用 API Key 而不是登录授权时，在 `~/.qwen/settings.json` 中定义同名服务器 `aibubu`，设置
`"httpUrl": "https://aibubu.cn/mcp"` 和 `"headers": {"Authorization": "Bearer <key>"}`。

文档：https://docs.aibubu.cn/cn/agent-tools
