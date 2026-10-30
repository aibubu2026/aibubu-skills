---
name: aibubu
description: 用 Aibubu（aibubu.cn）生成营销图片和视频：电商主图、商品场景图、广告变体、图生视频。当用户要求用 Aibubu 生成图片或视频、列出模型、查询模型参数或价格、查询余额、上传参考图或查询生成任务时使用。
---

# 用 Aibubu 生成营销图片和视频

本连接器通过 MCP 工具调用 Aibubu。认证由 WorkBuddy 的 OAuth 流程完成：需要授权时会打开浏览器，
不要让用户把 API Key 贴进聊天。工具列表能显示不代表已授权，用一次真实的 `get_balance` 调用确认。

## 规则

1. 模型 ID 只能从 `list_models` 获取，不要凭记忆写模型名；模型会新增、改名和下线。
2. 只传 `get_model` 返回的参数 schema 中声明过的字段，不要把一个模型的参数照搬到另一个模型。
3. 调用会消耗积分的工具（`generate_image`、`generate_video`、`generate_batch`、`upscale_image`）之前，
   先调用 `quote`（批量用 `quote_batch`，放大用 `quote_upscale`）和 `get_balance`。把模型、关键参数和
   预计积分告诉用户；批量生成或预计积分较高时，先得到用户确认；余额不足时停止并说明。
4. 生成是异步的：提交工具会立即返回 `task_id`。只提交一次，记下 `task_id`，再用 `wait_task` 或
   `get_task` 查进度，不要重复提交来查进度。
5. `wait_task` 每次调用请传 `timeout_seconds`（不超过 25），未完成就带同一个 `task_id` 再调用，
   以免单次调用超过连接器 30 秒的超时。
6. 提交时带 `idempotency_key`，网络重试时沿用同一个值；丢失 `task_id` 时用 `list_tasks` 找回。
7. 参考图、视频、音频要先通过 `open_uploader`、`upload_image`、`upload_file` 登记，再用 `asset_id` 引用。
8. 结果链接会过期，任务成功后及时下载或交付给用户。

## 典型流程

1. `list_models` 选模型，`get_model` 看参数。
2. 有参考图时先上传，拿到 `asset_id`。
3. `quote` 报价 + `get_balance` 查余额，告诉用户后得到确认。
4. `generate_image` / `generate_video` / `generate_batch` 提交，拿到 `task_id`。
5. `wait_task`（`timeout_seconds` ≤ 25）循环到任务结束，把结果交给用户。

文档：https://docs.aibubu.cn/cn/agent-tools
