# WorkBuddy 官方连接器市场提交清单（Aibubu）

本目录是按 [WorkBuddy 连接器开发文档](https://open.workbuddy.cn/docs/connector) 准备的 MCP + Skill 连接器包。
本文件是给提交人看的说明，**提交时不要把 `SUBMISSION.md` 打进包里**（见“打包”）。

## 1. 包内容

| 文件 | 说明 |
| --- | --- |
| `connector-meta.json` | 元信息。`source` = `aibubu`，`type` = `mcp`，`version` = `0.3.245`，双语 `examples_*` 各 4 条。 |
| `mcp.json` | 单个 server：`type: streamableHttp`，`url: https://aibubu.cn/mcp`，`timeout: 30000`。无任何 token。 |
| `icon.svg` | 取自 aibubu 仓库 `apps/web-cn/app/icon.svg`（黑色圆角底 + 白色标志，357x357 viewBox，纯矢量）。 |
| `skills/aibubu/SKILL.md` | 面向 MCP 工具的 Skill（可选项，已提供）。 |

说明：
- `auth_mode` **省略**。文档规定：MCP Server 自带 OAuth 或无需认证时省略该字段，WorkBuddy 走标准 MCP 流程，需要授权时打开浏览器。`server-side`、`gateway` 是 WorkBuddy 云端托管 OAuth，文档要求接入前先与 WorkBuddy 团队确认；`token` 需要 `token-schema.json`，且不打开浏览器。
- Skill 没有直接复制 `plugins/aibubu/skills`：那份是 REST + `curl` + `AIBUBU_API_KEY` 的用法，与 OAuth 连接器的场景冲突，所以单独写了基于 MCP 工具的 `SKILL.md`。如果审核方要求与 CLI 版一致，可以再调整。
- 文档对 `SKILL.md` 的 frontmatter 没有说明，细节在文档站 `/docs/skill`（本次没有读到，**未验证**）；目前使用 `name` + `description` 两个字段。
- 图标：文档要求 SVG/PNG/JPG，PNG/JPG 建议 64x64，背景建议透明，要“小尺寸下可辨识”。我们的 SVG 是带黑色圆角底的方形图标，不是透明背景；`logo-mark.svg` 是黑色标志加透明背景，在深色界面会看不见，所以没有选它。如果 WorkBuddy 要求透明底，需要设计师另出一版。
- 文档没有规定压缩格式，也没有规定包名。

## 2. 提交方式

1. 提交入口：企业账号登录 <https://open.workbuddy.cn/> 后，进入「发布管理 → 连接器 → 发布连接器」（`/connector/publish`），上传本目录打包的 zip（顶层为 `aibubu/`，不含本文件），平台会解析并生成连接器 ID，再补充服务类目和资质材料后提交审核。
2. 审核通过后进入连接器市场；更新需要重新提交审核，通常 10 到 15 分钟生效。
3. `source` 全局唯一，只能用小写字母、数字、连字符。提交时先确认 `aibubu` 没被占用。
4. 如果同一服务想同时提供 OAuth 和 Token 两种方式，要提交两个连接器、用不同的 `source`。目前只提交 OAuth 这一个。

## 3. 打包

文档没要求 zip。如果入驻表单需要压缩包，用下面的命令（只打包四个必需文件和 skills，排除本说明）：

```bash
cd connectors/workbuddy
zip -r aibubu-workbuddy-connector.zip aibubu -x 'aibubu/SUBMISSION.md' -x '*/.DS_Store'
```

## 4. OAuth 2.1 + DCR 合规对照

WorkBuddy 要求见文档“认证与凭证”。下表中“我方状态”已对照我们的后端核实过。

| WorkBuddy 要求 | 我方状态 |
| --- | --- |
| OAuth 2.1 + PKCE（S256），公共客户端，无 `client_secret` | 满足 |
| `GET /.well-known/oauth-protected-resource` | 根路径已上线（aibubu.cn）；另有 `/.well-known/oauth-protected-resource/mcp` |
| `GET /.well-known/oauth-authorization-server` | 已上线（aibubu.cn） |
| 动态客户端注册（RFC 7591），文档写的路径是 `POST /oauth/register`；必须回显 `redirect_uris` | 满足，但**实际路径是 `/api/oauth/register`**，不是 `/oauth/register`，通过 metadata 的 `registration_endpoint` 发现。已回显 `redirect_uris`。需 WorkBuddy 确认客户端按 metadata 走，而不是写死 `/oauth/register` |
| `redirect_uri` 精确匹配；优先 `workbuddy://workbuddy/mcp/connector%3Aaibubu/oauth/callback` | 自定义 scheme `workbuddy://...`（带 path）可接受 |
| 私有协议被拒时回退 `http://127.0.0.1:{动态端口}/oauth/callback` | 回环地址任意端口可接受（RFC 8252） |
| 授权码一次性，约 10 分钟过期 | 满足：单次使用，TTL 10 分钟 |
| access token 约 1 小时 | 满足：1 小时 |
| refresh token 不少于 30 天，`/oauth/token` 支持 `refresh_token` 授权 | 满足：60 天 |
| 所有 OAuth 端点使用 HTTPS | 满足 |
| scope | `mcp:read`、`mcp:generate`、`mcp:history` |

## 5. 服务要求

- HTTPS：满足。
- 响应时间：文档原文是“单次请求建议在 30 秒内响应”，`mcp.json` 的 `timeout` 默认 30000 ms，可用性建议不低于 99.9%。
- 对我们的影响：
  - `generate_image`、`generate_video` 等提交类工具会立即返回 `task_id`，不会长时间阻塞。
  - `wait_task` 的上限是 60 秒、默认 50 秒，**会超过 30 秒**。
  - 文档没有说明 30000 ms 是单次工具调用的超时，还是整个连接的超时。若是前者，需要让 Agent 调用 `wait_task` 时传 `timeout_seconds` ≤ 25，未完成就循环再调。包内 `SKILL.md` 已经写了这条规则。
  - 如果 WorkBuddy 允许把 `timeout` 调大（例如 60000），可以在 `mcp.json` 里改，并把 Skill 里的 25 秒规则放宽。这一点需要向 WorkBuddy 确认。
- 错误信息可读、不得在文件中出现真实 token：满足。

## 6. 需要向 WorkBuddy 团队确认的事项

1. 本地测试方法：文档没有给出本地导入或测试连接器目录的方式，请索取测试包安装方式或测试环境。
2. redirect URI 白名单：确认 WorkBuddy 实际使用的回调地址（`workbuddy://workbuddy/mcp/connector%3Aaibubu/oauth/callback` 与回环回退），以及是否有其他形式。
3. `auth_mode`：确认省略即可，不需要 `server-side` 或 `gateway`。
4. 注册端点路径：确认 WorkBuddy 客户端读取 authorization server metadata 里的 `registration_endpoint`（我们是 `/api/oauth/register`），而不是固定 `/oauth/register`。同理确认授权和令牌端点也按 metadata 走。
5. 超时：30000 ms 是否针对单次工具调用；是否允许把 `timeout` 设得更大。
6. 图标：是否接受带黑色圆角底的 SVG，还是必须透明背景。
7. Skill 格式：`/docs/skill` 对 `SKILL.md` frontmatter 的具体要求。
8. 渲染：连接器返回的 `resource_link` 图片、MCP Apps 界面资源是否在 WorkBuddy 里渲染，以及是否需要在工具结果中额外带图片内容（官方文档没有说明，**未验证**）。
9. 最低版本：如需 `minWorkbuddyVersion`，请确认；`examples_zh` / `examples_en` 需要 WorkBuddy 4.24.0 及以上，`name_zh` 同样是 4.24.0 及以上。我们目前没有声明 `minWorkbuddyVersion`，是否要补请一并确认。

## 7. 提交前自查（对应文档“提交前检查”）

- [x] 接入方式：MCP + Skill，目录结构符合要求
- [x] `source` 为 kebab-case（`aibubu`）
- [x] 名称、描述、双语示例已填写（描述 20 到 100 字，示例 4 条）
- [x] `mcp.json` 仅一个 server，HTTPS URL
- [x] OAuth：元数据发现、动态注册、PKCE 校验已在我方后端核实
- [x] Skill 覆盖核心能力（查模型、报价、余额、生成、轮询、上传）
- [x] 文件中无真实凭证
- [ ] 图标是否合规，待 WorkBuddy 确认（见第 1 节、第 6 节）
- [ ] 在真实 WorkBuddy 客户端里端到端跑通一次授权和调用（尚未做，因为没有本地测试方式）
- [x] 超时、授权过期、参数非法的错误处理：服务端已返回可读错误；`wait_task` 超时规则见第 5 节
