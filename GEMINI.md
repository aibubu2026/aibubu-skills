# Aibubu image and video generation

Use Aibubu when the user asks to generate an image or video with Aibubu, or to
list Aibubu models, check a model's parameters or price, check their Aibubu
balance, upload a reference file, or look up a generation task.

This extension connects to the remote MCP server `https://aibubu.ai/mcp`
(Streamable HTTP). Let Gemini CLI perform OAuth discovery: if the server needs
sign-in, run `/mcp auth aibubu` and let the user authorize in the browser. Do not
ask the user to paste an API key into the chat. Use `/mcp` to inspect the server.
Listing tools is not proof of authorization; confirm with a real `get_balance`
call.

## Rules

1. Get model IDs from `list_models` only. Never write a model name from memory;
   models are added, renamed and retired.
2. Send only parameters declared in the model's schema from `get_model`. Do not
   copy one model's parameters to another.
3. Call `quote` (or `quote_batch`, `quote_upscale`) and `get_balance` before any
   tool that spends credits (`generate_image`, `generate_video`,
   `generate_batch`, `upscale_image`). Tell the user the model, key parameters
   and estimated credits first; get confirmation for batches or large estimates.
   If the balance is short, stop and say so.
4. Generation is asynchronous. Submit once, keep the `task_id`, then poll with
   `wait_task` or `get_task`. Never resubmit to check progress. Pass an
   `idempotency_key` on submissions and reuse it when retrying after a network
   error; to recover a lost `task_id`, use `list_tasks`.
5. Reference images, video and audio must be registered first (`open_uploader`,
   `upload_image`, `upload_file`) and used by `asset_id`.
6. Result links expire. Download outputs promptly after a task succeeds.
7. Follow Gemini CLI's normal permission flow after a denied tool call. Do not
   switch transports or broaden permissions to bypass a denial.

## China region (aibubu.cn)

This extension targets aibubu.ai. For aibubu.cn, define a server named `aibubu`
in `~/.gemini/settings.json` with `"httpUrl": "https://aibubu.cn/mcp"`; a
settings entry of the same name takes precedence over the extension's.

Docs: https://docs.aibubu.ai/en/agent-tools
