# Aibubu Skills

Official Aibubu plugins for Claude Code, Codex, Cursor, Gemini CLI, Qwen Code, Kimi Code, CodeBuddy, WorkBuddy, Cline and other coding agents: generate images and videos with Aibubu models through a remote MCP server and the `aibubu-models` Agent Skill.

[中文说明](#中文说明)

## Claude Code

```text
/plugin marketplace add aibubu2026/aibubu-skills
/plugin install aibubu-global@aibubu
```

Then run `/mcp`, choose `aibubu` and sign in to authorize. No API key needed.

## Codex

```bash
codex plugin marketplace add aibubu2026/aibubu-skills
codex plugin add aibubu-global@aibubu
```

Then sign in with `codex mcp login aibubu` (start a new session afterwards). For aibubu.cn install `aibubu@aibubu` instead. To use an API key rather than sign-in, set `bearer_token_env_var = "AIBUBU_API_KEY"` on `[mcp_servers.aibubu]` in `~/.codex/config.toml`.

## Cursor

This repo ships Cursor plugins (`.cursor-plugin/marketplace.json`) for `aibubu-global` and `aibubu`. Add the repo as a team marketplace in the Cursor dashboard, or test locally by copying `plugins/aibubu-global` to `~/.cursor/plugins/local/aibubu-global`; then install it from **Customize** and authorize `aibubu` in MCP settings.

Without the plugin: [Add to Cursor](https://cursor.com/install-mcp?name=aibubu&config=eyJ1cmwiOiJodHRwczovL2FpYnVidS5haS9tY3AifQ==) (aibubu.ai) or [aibubu.cn](https://cursor.com/install-mcp?name=aibubu&config=eyJ1cmwiOiJodHRwczovL2FpYnVidS5jbi9tY3AifQ==), or add `{"mcpServers": {"aibubu": {"url": "https://aibubu.ai/mcp"}}}` to `~/.cursor/mcp.json`.

## Gemini CLI

```bash
gemini extensions install https://github.com/aibubu2026/aibubu-skills
```

Then run `/mcp auth aibubu` and sign in. The extension targets aibubu.ai. For aibubu.cn, define a server named `aibubu` with `"httpUrl": "https://aibubu.cn/mcp"` in `~/.gemini/settings.json`; it takes precedence over the extension's. For an API key, add `"headers": {"Authorization": "Bearer <key>"}` to that entry.

## CodeBuddy / WorkBuddy (Tencent)

```text
/plugin marketplace add aibubu2026/aibubu-skills
/plugin install aibubu@aibubu
```

Run it in CodeBuddy CLI/IDE or in the WorkBuddy chat box. This repo ships `.codebuddy-plugin/marketplace.json` and `.workbuddy-plugin/marketplace.json` (identical). The `aibubu` plugin targets aibubu.cn; `aibubu-global` targets aibubu.ai. Then sign in to `aibubu` from the MCP panel (`/mcp` in CodeBuddy CLI).

## Qwen Code

```bash
qwen extensions install https://github.com/aibubu2026/aibubu-skills
```

The extension (`qwen-extension.json`) targets aibubu.cn. Run `/mcp` in a session to sign in. For aibubu.ai, define a server named `aibubu` with `"httpUrl": "https://aibubu.ai/mcp"` in `~/.qwen/settings.json`; it takes precedence over the extension's.

## Kimi Code

```text
/plugins install https://github.com/aibubu2026/aibubu-skills
```

The root `kimi.plugin.json` targets aibubu.cn. Then run `/mcp-config login aibubu` and sign in. For aibubu.ai, add `{"mcpServers": {"aibubu": {"url": "https://aibubu.ai/mcp"}}}` to `~/.kimi-code/mcp.json`.

## Cline

Ask Cline to set up the server from [llms-install.md](llms-install.md), or add `aibubu` as a Streamable HTTP remote server with the URL `https://aibubu.ai/mcp` (aibubu.cn: `https://aibubu.cn/mcp`).

## Other coding agents

```bash
npx skills add aibubu2026/aibubu-skills
```

The skill calls the API with `curl` and reads `AIBUBU_API_KEY` from the environment.

## Plugins

| Plugin | Region | MCP server |
| --- | --- | --- |
| `aibubu-global` | aibubu.ai | `https://aibubu.ai/mcp` |
| `aibubu` | aibubu.cn (China) | `https://aibubu.cn/mcp` |

## Repository layout

| Path | Used by |
| --- | --- |
| `.claude-plugin/marketplace.json`, `plugins/*/.claude-plugin/plugin.json`, `plugins/*/.mcp.json` | Claude Code |
| `.agents/plugins/marketplace.json`, `plugins/*/plugin.json`, `plugins/*/mcp.json` | Codex |
| `.cursor-plugin/marketplace.json`, `plugins/*/.cursor-plugin/plugin.json` | Cursor |
| `gemini-extension.json`, `GEMINI.md` | Gemini CLI |
| `.codebuddy-plugin/marketplace.json`, `.workbuddy-plugin/marketplace.json` (plugins reuse `.claude-plugin/plugin.json` and `.mcp.json`) | CodeBuddy, WorkBuddy |
| `qwen-extension.json`, `QWEN.md` | Qwen Code |
| `kimi.plugin.json` | Kimi Code |
| `connectors/workbuddy/aibubu` | WorkBuddy 官方连接器市场提交包（尚未上架） |
| `llms-install.md` | Cline |
| `plugins/*/skills/aibubu-models` | Claude Code, Codex, Cursor, CodeBuddy, WorkBuddy, Qwen Code, Kimi Code, `npx skills add` |

Docs: <https://docs.aibubu.ai/en/agent-tools>

## 中文说明

适用于 Claude Code、Codex、Cursor、Gemini CLI、千问 Qwen Code、Kimi Code、CodeBuddy / WorkBuddy、Cline 等编程 agent 的 Aibubu 官方插件：通过远程 MCP 和 `aibubu-models` Skill，用 Aibubu 的模型生成图片和视频。

### Claude Code

```text
/plugin marketplace add aibubu2026/aibubu-skills
/plugin install aibubu@aibubu
```

安装后输入 `/mcp`，选择 `aibubu`，登录 Aibubu 账号授权即可，不需要 API Key。国际版请安装 `aibubu-global@aibubu`。

### Codex

```bash
codex plugin marketplace add aibubu2026/aibubu-skills
codex plugin add aibubu@aibubu
```

然后运行 `codex mcp login aibubu` 登录授权，并新开会话。国际版请安装 `aibubu-global@aibubu`。使用 API Key 而非登录授权时，在 `~/.codex/config.toml` 的 `[mcp_servers.aibubu]` 中设置 `bearer_token_env_var = "AIBUBU_API_KEY"`。

### Cursor

仓库提供 Cursor 插件（`.cursor-plugin/marketplace.json`），含 `aibubu` 与 `aibubu-global`。可在 Cursor 控制台把本仓库添加为团队插件市场，或将 `plugins/aibubu` 复制到 `~/.cursor/plugins/local/aibubu` 本地测试，然后在 **Customize** 中安装，并在 MCP 设置里登录授权。

不用插件时：[添加到 Cursor](https://cursor.com/install-mcp?name=aibubu&config=eyJ1cmwiOiJodHRwczovL2FpYnVidS5jbi9tY3AifQ==)（aibubu.cn），或在 `~/.cursor/mcp.json` 中加入 `{"mcpServers": {"aibubu": {"url": "https://aibubu.cn/mcp"}}}`。

### Gemini CLI

```bash
gemini extensions install https://github.com/aibubu2026/aibubu-skills
```

然后运行 `/mcp auth aibubu` 登录授权。扩展默认连接国际版 aibubu.ai；使用国内版 aibubu.cn 时，请在 `~/.gemini/settings.json` 中定义同名服务器 `aibubu`，设置 `"httpUrl": "https://aibubu.cn/mcp"`，它会覆盖扩展内的配置。

### CodeBuddy / WorkBuddy（腾讯）

在 CodeBuddy CLI / IDE，或 WorkBuddy 的对话框中输入：

```text
/plugin marketplace add aibubu2026/aibubu-skills
/plugin install aibubu@aibubu
```

仓库内置 `.codebuddy-plugin/marketplace.json` 和 `.workbuddy-plugin/marketplace.json`（内容相同），`aibubu` 插件连接 aibubu.cn，`aibubu-global` 连接 aibubu.ai。安装后在 MCP 面板（CodeBuddy CLI 里输入 `/mcp`）选择 `aibubu`，登录 Aibubu 账号授权；WorkBuddy 需在 **插件 → MCP 服务器** 中确认信任该服务。不使用插件时，可手动添加：

```bash
codebuddy mcp add --scope user --transport http aibubu https://aibubu.cn/mcp
```

WorkBuddy 则在 **插件 → MCP 服务器 → 配置 MCP** 中编辑 `~/.workbuddy/mcp.json`：

```json
{
  "mcpServers": {
    "aibubu": {
      "type": "streamableHttp",
      "url": "https://aibubu.cn/mcp"
    }
  }
}
```

WorkBuddy 官方连接器市场的提交包位于 [connectors/workbuddy/aibubu](connectors/workbuddy/aibubu)（`connector-meta.json`、`mcp.json`、`icon.svg`、`skills/`），提交步骤见其中的 `SUBMISSION.md`；审核通过后可在 WorkBuddy 连接器市场直接安装。

### 千问 Qwen Code

```bash
qwen extensions install https://github.com/aibubu2026/aibubu-skills
```

扩展（`qwen-extension.json`）默认连接 aibubu.cn。在会话中输入 `/mcp`，在对话框里完成登录授权。使用国际版 aibubu.ai 或 API Key 时，在 `~/.qwen/settings.json` 中定义同名服务器 `aibubu`（`httpUrl` 加 `headers`），它会覆盖扩展内的配置。

### Kimi Code

```text
/plugins install https://github.com/aibubu2026/aibubu-skills
```

根目录的 `kimi.plugin.json` 连接 aibubu.cn。安装后运行 `/new`（或 `/reload`）使其生效，再运行 `/mcp-config login aibubu` 登录授权。也可直接在 `~/.kimi-code/mcp.json` 中加入 `{"mcpServers": {"aibubu": {"url": "https://aibubu.cn/mcp"}}}`。

### 其他国内客户端（无插件文件，手动添加）

下列客户端没有可放进本仓库的插件格式，请手动添加远程 MCP 服务器 `https://aibubu.cn/mcp`，详见文档：

- 扣子 Coze：**扩展 → MCP → 添加自定义 MCP**，粘贴 `{"mcpServers": {"aibubu": {"url": "https://aibubu.cn/mcp"}}}`。
- Trae / Trae CN：**设置 → MCP → 添加 → 手动添加**，粘贴 `{"mcpServers": {"aibubu": {"url": "https://aibubu.cn/mcp", "headers": {"Authorization": "Bearer sk_aibubu_..."}}}}`（Trae 目前只支持 Header，需要 API Key）。
- 豆包桌面端：**插件 · 技能 · 伙伴 → 添加 → 新建自定义连接器**，传输类型选 HTTP，URL 填 `https://aibubu.cn/mcp`，Headers 填 `Authorization: Bearer sk_aibubu_...`。
- 千问办公 QwenWork：**扩展 → 连接器 → 添加 → 粘贴 JSON 配置**，使用 `streamable-http` 与 `Authorization` 头。

### Cline

让 Cline 按 [llms-install.md](llms-install.md) 安装，或在远程服务器中添加 `aibubu`，传输类型选 Streamable HTTP，地址为 `https://aibubu.cn/mcp`（国际版 `https://aibubu.ai/mcp`）。

### 其他编程 agent

```bash
npx skills add aibubu2026/aibubu-skills
```

Skill 用 `curl` 调用接口，从环境变量 `AIBUBU_API_KEY` 读取 Key。

文档：<https://docs.aibubu.cn/cn/agent-tools>

---

The skill files are generated from `agent-tools/skills` in the Aibubu product repository; edit them there and regenerate.
