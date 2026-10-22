# Aibubu Skills

Official Aibubu plugins for Claude Code and other coding agents: generate images and videos with Aibubu models through a remote MCP server and the `aibubu-models` Agent Skill.

[中文说明](#中文说明)

## Claude Code

```text
/plugin marketplace add aibubu2026/aibubu-skills
/plugin install aibubu-global@aibubu
```

Then run `/mcp`, choose `aibubu` and sign in to authorize. No API key needed.

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

Docs: <https://docs.aibubu.ai/en/agent-tools>

## 中文说明

适用于 Claude Code 等编程 agent 的 Aibubu 官方插件：通过远程 MCP 和 `aibubu-models` Skill，用 Aibubu 的模型生成图片和视频。

### Claude Code

```text
/plugin marketplace add aibubu2026/aibubu-skills
/plugin install aibubu@aibubu
```

安装后输入 `/mcp`，选择 `aibubu`，登录 Aibubu 账号授权即可，不需要 API Key。国际版请安装 `aibubu-global@aibubu`。

### 其他编程 agent

```bash
npx skills add aibubu2026/aibubu-skills
```

Skill 用 `curl` 调用接口，从环境变量 `AIBUBU_API_KEY` 读取 Key。

文档：<https://docs.aibubu.cn/cn/agent-tools>

---

The skill files are generated from `agent-tools/skills` in the Aibubu product repository; edit them there and regenerate.
