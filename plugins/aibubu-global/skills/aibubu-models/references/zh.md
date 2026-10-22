
# 在 Aibubu 上生成图片和视频

Aibubu(`https://aibubu.ai/v1`)用一套统一接口提供图片和视频生成模型。本文覆盖模型发现、报价、
余额检查、参考图上传、提交任务、轮询和下载结果的完整流程。

English: [references/en.md](references/en.md)

## 重要规则

1. **模型 ID 只能从 `GET /v1/models` 获取,不要凭记忆或训练数据写模型名。** 模型会新增、
   改名和下线,列表还会随区域和 API Key 权限变化。
2. **参数以模型的 `schema.fields` 为准。** 只传 schema 里声明过的字段,不要把一个模型的
   参数照搬到另一个模型。没有单独的 schema 接口,schema 就在模型列表的每一项里。
3. **提交前先报价、再查余额。** `POST /v1/quote` 返回本次请求的总积分 `estimated_credits`
   (已经包含 `n`,不要再乘)。余额不足时停下来告诉用户,不要硬提交。
4. **生成会消耗积分。** 提交前把模型、关键参数和预计积分告诉用户;批量生成或预计积分较高时,
   先得到用户确认。
5. **同一个任务只提交一次。** 提交返回 `task_id` 后,用 `GET /v1/tasks/{task_id}` 轮询。
   不要通过再次调用提交接口来查进度。每次提交都带 `Idempotency-Key`,网络重试时沿用同一个值。
6. **结果要及时下载。** 平台输出 URL 不承诺永久有效,任务成功后立即下载到本地。
7. **不要把 API Key 写进项目文件、代码或日志。** 只从环境变量 `AIBUBU_API_KEY` 读取。

## 先决条件

- 环境变量 `AIBUBU_API_KEY`:在 <https://aibubu.ai/api-keys> 创建 API Key。API Key 归属于团队工作空间,
  任务、扣费和生成结果都记在该工作空间下。
- 下文示例是 POSIX shell,需要 `curl` 和 `jq`。没有 `jq` 时:`brew install jq`、
  `apt install jq` 或 `winget install jqlang.jq`。
- Windows 请在 WSL 或 Git Bash 里运行。PowerShell 里的 `curl` 是 `Invoke-WebRequest`
  的别名,参数不兼容,要写 `curl.exe`。

开始前先确认 Key 已设置:

```bash
test -n "$AIBUBU_API_KEY" || echo "请先 export AIBUBU_API_KEY=sk_aibubu_..."
```

## 鉴权

每个请求都带:

```
Authorization: Bearer $AIBUBU_API_KEY
```

base URL 是 `https://aibubu.ai/v1`。

## 发现模型

```bash
curl -s "https://aibubu.ai/v1/models" -H "Authorization: Bearer $AIBUBU_API_KEY" \
  | jq '.data[] | select(.enabled) | {id, family, description, pricing: .pricing.display}'
```

- `id`:后续所有接口里传给 `model` 的值。
- `family`:`image`、`video` 或 `text`。本文只处理 `image` 和 `video`。
- `pricing`:只是展示信息,准确积分以 `/v1/quote` 为准。
- 接口不支持服务端筛选,用 `jq` 在本地按 `family` 或关键词过滤:

```bash
curl -s "https://aibubu.ai/v1/models" -H "Authorization: Bearer $AIBUBU_API_KEY" \
  | jq '[.data[] | select(.enabled and .family == "video") | .id]'
```

### 读取某个模型的参数

```bash
MODEL=gpt-image-2   # 用上一步查到的 id 替换
curl -s "https://aibubu.ai/v1/models" -H "Authorization: Bearer $AIBUBU_API_KEY" \
  | jq --arg m "$MODEL" '.data[] | select(.id == $m) | .schema.fields'
```

每个字段包含 `name`、`type`、`required`、`default`,以及可选的枚举 `options` 和数量限制。
`required: true` 的字段必须传;有 `options` 的字段只能传其中的值。

## 报价

报价请求和提交任务用**同一份扁平参数**。会影响价格的参数(`n`、`resolution`、`quality`、
`duration`、参考视频和输入视频时长等)必须和稍后提交时完全一致,否则两次结果会不同。

```bash
curl -s -X POST "https://aibubu.ai/v1/quote" \
  -H "Authorization: Bearer $AIBUBU_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "gpt-image-2", "n": 1, "resolution": "2K", "aspect_ratio": "1:1"}'
# → {"model":"gpt-image-2","family":"image","estimated_credits":11.2,"unit":"image",...}
```

报价是只读操作,不会创建任务或冻结积分。

## 查余额

```bash
curl -s "https://aibubu.ai/v1/user/balance" -H "Authorization: Bearer $AIBUBU_API_KEY"
# → {"quota_amount":1000,"authorized_amount":120,"currency":"credits",...}
```

`quota_amount` 是当前可用积分。它小于报价的 `estimated_credits` 时,提交会返回 `402
insufficient_credits`,应先告诉用户充值。`authorized_amount` 是运行中任务暂时占用的积分。

## 上传参考图

本地图片要先上传,拿到平台 URL 再放进 `reference_images`(或模型 schema 里声明的
`first_frame_image_url` 等字段)。只支持 PNG、JPEG、WebP、GIF,上传不扣积分。

```bash
curl -s -X POST "https://aibubu.ai/v1/uploads/images" \
  -H "Authorization: Bearer $AIBUBU_API_KEY" \
  -F "file=@./reference.png" | jq -r '.url'
```

使用接口返回的 `url`,不要自己拼对象存储地址。这个 URL 是约 15 分钟有效的签名链接,上传后尽快提交任务;
过期后重新上传。参考视频和参考音频没有上传接口,需要用户提供
可公开下载的 HTTPS 地址。

## 提交任务

图片和视频都用 `POST /v1/generations`,服务端根据 `model` 判断任务类型。

```bash
curl -s -X POST "https://aibubu.ai/v1/generations" \
  -H "Authorization: Bearer $AIBUBU_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: $(date +%s)-poster" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "一张极简风格的未来城市海报",
    "n": 1,
    "resolution": "2K",
    "aspect_ratio": "1:1"
  }'
# → {"task_id":"task_01jxyz...","object":"generation.task","progress":0,...}
```

- 返回 `201` 只代表任务已提交,结果要看任务终态。
- 记下 `task_id`。网络超时需要重试时,沿用同一个 `Idempotency-Key`;换了请求体就要换新的
  幂等键,否则会返回 `409 idempotency_conflict`。

## 取结果

```bash
TASK_ID=task_01jxyz...
curl -s "https://aibubu.ai/v1/tasks/$TASK_ID" -H "Authorization: Bearer $AIBUBU_API_KEY" \
  | jq '{status, progress, result, error}'
```

### 状态机

| `status` | 含义 | 下一步 |
|---|---|---|
| `queued`、`running` | 未完成 | 继续轮询 |
| `succeeded` | 成功 | 读取 `result.data[].url` 并下载 |
| `failed`、`canceled`、`expired` | 终态,未成功 | 读 `error` 告诉用户;是否重新提交由用户决定 |

轮询间隔:图片 3 秒左右,视频 10 秒左右;任务持续较久时逐步拉长间隔。
查询接口本身报错(网络、5xx)时继续轮询,不代表任务失败。

一个完整的等待并下载的写法:

```bash
mkdir -p ./output
while :; do
  T=$(curl -s "https://aibubu.ai/v1/tasks/$TASK_ID" -H "Authorization: Bearer $AIBUBU_API_KEY")
  S=$(echo "$T" | jq -r '.status')
  case "$S" in
    succeeded)
      echo "$T" | jq -r '.result.data[].url' | while read -r U; do
        curl -s -L -o "./output/$(basename "${U%%\?*}")" "$U"
      done
      break ;;
    failed|canceled|expired) echo "$T" | jq '.error'; break ;;
    *) sleep 5 ;;
  esac
done
```

视频任务有封面时,`result.data[]` 里还会有 `thumbnail_url`。

## 错误处理

所有错误都是 `{"error": {"code": "...", "message": "..."}}`。

| HTTP | `code` | 处理 |
|---|---|---|
| 400 | `invalid_request`、`quote_error` | 对照 `schema.fields` 修正参数,不要原样重试 |
| 401 | `unauthenticated` | 检查 `AIBUBU_API_KEY` |
| 402 | `insufficient_credits` | 停止,提示用户充值 |
| 403 | `scope_forbidden` | Key 缺少权限(生成需要 `v1:generate`),请用户换 Key |
| 404 | `model_not_found`、`task_not_found` | 重新从模型列表取 ID;检查任务是否属于这个 Key 的工作空间 |
| 409 | `idempotency_conflict` | 请求体变了,换新的 `Idempotency-Key` |
| 409 | `idempotency_in_progress` | 按 `Retry-After` 等待后用同一个键重试 |
| 429 | `rate_limit` | 按 `Retry-After` 退避 |
| 5xx | — | 用同一个 `Idempotency-Key` 有限次退避重试 |

## 更多

- 完整文档:<https://docs.aibubu.ai/en>
- 文档索引(适合 AI 读取):<https://docs.aibubu.ai/llms.txt>
