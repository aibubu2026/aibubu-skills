
# Generating images and videos on Aibubu

Aibubu (`https://aibubu.cn/v1`) serves image and video generation models behind one API. This covers
the full flow: discovering models, quoting, checking balance, uploading references, submitting a
task, polling and downloading the result.

中文: [references/zh.md](references/zh.md)

## Critical rules

1. **Get model IDs from `GET /v1/models` only. Never write a model name from memory or training
   data.** Models are added, renamed and retired, and the list depends on region and API key.
2. **Parameters come from the model's `schema.fields`.** Send only declared fields and never copy
   one model's parameters to another. There is no separate schema endpoint; the schema is part of
   each item in the model list.
3. **Quote, then check balance, before submitting.** `POST /v1/quote` returns the total
   `estimated_credits` for the request (it already includes `n`; do not multiply again). If the
   balance is short, stop and tell the user instead of submitting.
4. **Generation spends credits.** Tell the user the model, key parameters and estimated credits
   before submitting; for batches or large estimates, get the user's confirmation first.
5. **Submit a task once.** After you get a `task_id`, poll `GET /v1/tasks/{task_id}`. Never call
   the submit endpoint again to check progress. Send an `Idempotency-Key` with every submission
   and reuse it when retrying after a network error.
6. **Download results promptly.** Output URLs are not guaranteed to be permanent; save them
   locally as soon as the task succeeds.
7. **Never write the API key into project files, code or logs.** Read it only from the
   `AIBUBU_API_KEY` environment variable.

## Prerequisites

- `AIBUBU_API_KEY` environment variable: create a key at <https://aibubu.cn/api-keys>. Keys belong to a team
  workspace; tasks, charges and outputs are recorded under that workspace.
- The snippets below are POSIX shell and need `curl` and `jq`. To install `jq`:
  `brew install jq`, `apt install jq` or `winget install jqlang.jq`.
- On Windows, run them in WSL or Git Bash. PowerShell's `curl` is an alias for
  `Invoke-WebRequest` and does not accept these flags; use `curl.exe`.

Check the key is set before you start:

```bash
test -n "$AIBUBU_API_KEY" || echo "Run: export AIBUBU_API_KEY=sk_aibubu_..."
```

## Authentication

Every request carries:

```
Authorization: Bearer $AIBUBU_API_KEY
```

The base URL is `https://aibubu.cn/v1`.

## Discovering models

```bash
curl -s "https://aibubu.cn/v1/models" -H "Authorization: Bearer $AIBUBU_API_KEY" \
  | jq '.data[] | select(.enabled) | {id, family, description, pricing: .pricing.display}'
```

- `id`: the value to pass as `model` everywhere else.
- `family`: `image`, `video` or `text`. This skill covers `image` and `video`.
- `pricing`: display text only; exact credits come from `/v1/quote`.
- The endpoint has no server-side filters. Filter locally with `jq`:

```bash
curl -s "https://aibubu.cn/v1/models" -H "Authorization: Bearer $AIBUBU_API_KEY" \
  | jq '[.data[] | select(.enabled and .family == "video") | .id]'
```

### Reading a model's parameters

```bash
MODEL=gpt-image-2   # replace with an id from the step above
curl -s "https://aibubu.cn/v1/models" -H "Authorization: Bearer $AIBUBU_API_KEY" \
  | jq --arg m "$MODEL" '.data[] | select(.id == $m) | .schema.fields'
```

Each field has `name`, `type`, `required`, `default`, and optionally an `options` enum and count
limits. Fields with `required: true` must be sent; fields with `options` accept only those values.

## Quoting

A quote takes **the same flat body** as the submission. Every price-affecting parameter (`n`,
`resolution`, `quality`, `duration`, reference videos, input video duration and so on) must match
the later submission exactly, or the two numbers will differ.

```bash
curl -s -X POST "https://aibubu.cn/v1/quote" \
  -H "Authorization: Bearer $AIBUBU_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"model": "gpt-image-2", "n": 1, "resolution": "2K", "aspect_ratio": "1:1"}'
# → {"model":"gpt-image-2","family":"image","estimated_credits":11.2,"unit":"image",...}
```

Quoting is read-only: it creates no task and holds no credits.

## Checking the balance

```bash
curl -s "https://aibubu.cn/v1/user/balance" -H "Authorization: Bearer $AIBUBU_API_KEY"
# → {"quota_amount":1000,"authorized_amount":120,"currency":"credits",...}
```

`quota_amount` is the credits available now. If it is lower than the quote's
`estimated_credits`, submission fails with `402 insufficient_credits`; ask the user to top up
first. `authorized_amount` is held by running tasks.

## Uploading reference images

Upload local images first and put the returned URL in `reference_images` (or a field the schema
declares, such as `first_frame_image_url`). PNG, JPEG, WebP and GIF only; uploads are free.

```bash
curl -s -X POST "https://aibubu.cn/v1/uploads/images" \
  -H "Authorization: Bearer $AIBUBU_API_KEY" \
  -F "file=@./reference.png" | jq -r '.url'
```

Use the returned `url`; do not build storage URLs yourself. It is a signed link valid for about 15
minutes, so submit the task soon after uploading and re-upload if it expires. There is no upload endpoint for
reference videos or audio; the user must provide a publicly downloadable HTTPS URL.

## Submitting a task

Images and videos both use `POST /v1/generations`; the server infers the task type from `model`.

```bash
curl -s -X POST "https://aibubu.cn/v1/generations" \
  -H "Authorization: Bearer $AIBUBU_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: $(date +%s)-poster" \
  -d '{
    "model": "gpt-image-2",
    "prompt": "A minimalist poster of a futuristic city",
    "n": 1,
    "resolution": "2K",
    "aspect_ratio": "1:1"
  }'
# → {"task_id":"task_01jxyz...","object":"generation.task","progress":0,...}
```

- `201` only means the task was accepted; the result depends on its final state.
- Keep the `task_id`. When retrying a timed-out request, reuse the same `Idempotency-Key`; a
  different body needs a new key or you get `409 idempotency_conflict`.

## Getting the result

```bash
TASK_ID=task_01jxyz...
curl -s "https://aibubu.cn/v1/tasks/$TASK_ID" -H "Authorization: Bearer $AIBUBU_API_KEY" \
  | jq '{status, progress, result, error}'
```

### State machine

| `status` | Meaning | Next |
|---|---|---|
| `queued`, `running` | Not finished | Keep polling |
| `succeeded` | Done | Read `result.data[].url` and download |
| `failed`, `canceled`, `expired` | Final, unsuccessful | Show `error` to the user; let them decide whether to resubmit |

Poll images about every 3 seconds and videos about every 10 seconds; back off further for long
tasks. A failed status request (network error, 5xx) is not a failed task; keep polling.

Wait and download:

```bash
mkdir -p ./output
while :; do
  T=$(curl -s "https://aibubu.cn/v1/tasks/$TASK_ID" -H "Authorization: Bearer $AIBUBU_API_KEY")
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

Video results may also include a `thumbnail_url` in `result.data[]`.

## Errors

Every error is `{"error": {"code": "...", "message": "..."}}`.

| HTTP | `code` | Action |
|---|---|---|
| 400 | `invalid_request`, `quote_error` | Fix parameters against `schema.fields`; do not retry unchanged |
| 401 | `unauthenticated` | Check `AIBUBU_API_KEY` |
| 402 | `insufficient_credits` | Stop and ask the user to top up |
| 403 | `scope_forbidden` | The key lacks a scope (generation needs `v1:generate`); ask for another key |
| 404 | `model_not_found`, `task_not_found` | Re-read IDs from the model list; check the task belongs to this key's workspace |
| 409 | `idempotency_conflict` | The body changed; use a new `Idempotency-Key` |
| 409 | `idempotency_in_progress` | Wait for `Retry-After`, then retry with the same key |
| 429 | `rate_limit` | Back off per `Retry-After` |
| 5xx | — | Retry a few times with backoff and the same `Idempotency-Key` |

## More

- Full documentation: <https://docs.aibubu.cn/cn>
- Docs index for AI tools: <https://docs.aibubu.cn/llms.txt>
