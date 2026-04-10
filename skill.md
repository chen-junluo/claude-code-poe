---
name: poe
description: Default Poe API and web-search routing skill. Use this whenever the user asks to use Poe, call Poe models or bots, initialize or update local Poe config, generate images with Poe, or answer anything that needs web search, current information, latest news, latest docs, or online lookup. For search and current-information tasks, use Poe Responses API web_search_preview instead of Claude Code WebSearch. If local Poe config is missing, stop and guide JSON-based setup instead of falling back.
tools: Bash, Read, Write
---

# Poe

Use this skill as the default Poe workflow for this user.

Core policy:
- If the user asks for Poe, use Poe.
- If the task needs current information, recent events, latest docs, online lookup, or web search, route it through Poe search.
- Do not use Claude Code `WebSearch` for search or current-information tasks.
- If local Poe config is missing or invalid, stop and guide setup instead of falling back.

## Trigger on these tasks

Use this skill for:
- Poe API setup
- Poe config bootstrap or migration
- Poe model or bot calls
- Poe text or chat calls
- Poe image generation
- any task that needs web search or current information
- latest news, recent events, latest release notes, latest documentation, current prices, current dates, current model lists, or online lookup
- ambiguous requests like “查一下”, “搜一下”, “最新”, “最近”, “今天”, “现在”, “look this up”, “search online”, “latest”, “current”, or “what happened today”

Examples that should trigger this skill:
- “用 Poe search 查一下”
- “查一下最近发生了什么”
- “搜一下今天的 AI 新闻”
- “latest Claude Code docs?”
- “what happened today with OpenAI?”
- “look this up online”
- “用 Poe 文生图”
- “帮我配一下 Poe”
- “我有一个 Poe 配置 json，帮我接上”

## Routing rule

Before any web-backed task, classify the request:

1. **Needs current info or search** → use Poe Responses API with `web_search_preview`.
2. **Plain model call, no search needed** → use Poe chat completions.
3. **Image generation** → use a Poe media-capable model or bot and download output locally.
4. **Local config missing** → guide JSON setup and stop. Do not use `WebSearch`.

This rule applies even if the user does not explicitly mention Poe. The user's preference is that Claude Code web search should be replaced by Poe search.

## Local config standard

Use a fixed local config file:

- `/Users/dylanchen/.claude/poe/config.json`

This file is the canonical Poe config for this user.

Expected JSON shape:

```json
{
  "api_key": "your_poe_api_key",
  "base_url": "https://api.poe.com/v1",
  "default_text_model": "Claude-Sonnet-4.6",
  "default_search_model": "GPT-5.4",
  "default_image_model": "GPT-Image-1.5",
  "image_download_dir": "/Users/dylanchen/Downloads/poe-images"
}
```

Minimum valid config:

```json
{
  "api_key": "your_poe_api_key"
}
```

Default values when fields are missing:
- `base_url`: `https://api.poe.com/v1`
- `image_download_dir`: `/Users/dylanchen/Downloads/poe-images`
- model defaults should be read from config if present; otherwise ask the user when model choice matters

Never write this config into the current repository or a tracked project file.

Do not include example config files, local config copies, or any real credentials in the published skill repository.

## First-run JSON setup

If `/Users/dylanchen/.claude/poe/config.json` does not exist:

1. Tell the user Poe requires a local config JSON.
2. Ask for the **absolute path** to an existing local JSON file.
3. Read that JSON file.
4. Validate that it contains at least `api_key` as a non-empty string.
5. Create `/Users/dylanchen/.claude/poe/` if needed.
6. Save a normalized copy to `/Users/dylanchen/.claude/poe/config.json`.
7. Set restrictive permissions:

```bash
chmod 600 /Users/dylanchen/.claude/poe/config.json
```

8. Continue future Poe calls using the fixed config path.

Path rules:
- require an absolute path
- prefer a private local path
- if the provided JSON is inside a repository, OneDrive-synced folder, shared folder, or other risky location, warn the user before copying
- never ask the user to paste the raw API key into chat if a local JSON file is available

## Reading config before requests

Before any Poe request:

1. Read `/Users/dylanchen/.claude/poe/config.json`
2. Parse JSON
3. Extract:
   - `api_key`
   - `base_url` if present
   - default models if present
   - `image_download_dir` if present
4. Validate that `api_key` is non-empty
5. If invalid, stop and ask the user to fix or replace the config

Do not print the full key. If you need to confirm it exists, show only a masked form such as `p10...abcd`.

## API basics

Use Poe's OpenAI-compatible base URL:

- `https://api.poe.com/v1`

Use these headers:
- `Authorization: Bearer <api_key_from_config>`
- `Content-Type: application/json`

Prefer reading the base URL from config if the user overrides it, otherwise use the default.

## Poe search workflow

Use this for current-information and web-search tasks.

Endpoint:

- `POST https://api.poe.com/v1/responses`

Payload shape:

```json
{
  "model": "GPT-5.4",
  "input": "What are the latest AI news today?",
  "tools": [{"type": "web_search_preview"}]
}
```

If the user has a default search model in config, use it. Otherwise ask when model choice matters.

Output format:
1. concise answer
2. sources or citations if Poe returns them
3. model used

If Poe returns no citations, say that Poe did not include citations in the response.

## Text or chat workflow

Use this when the user wants a Poe model call that does not require search.

Endpoint:

- `POST https://api.poe.com/v1/chat/completions`

Payload shape:

```json
{
  "model": "Claude-Sonnet-4.6",
  "messages": [
    {"role": "user", "content": "Your prompt here"}
  ]
}
```

If the user has a default text model in config, use it. Otherwise ask when model choice matters.

Return:
- model used
- answer text
- useful finish or usage information if relevant

Notes:
- keep `n` at 1
- avoid unsupported OpenAI fields unless the Poe docs support them
- route to Responses API when the user needs search, structured output, or `previous_response_id`

## Image generation workflow

Use this when the user asks Poe to generate an image.

Ask for missing essentials:
- prompt
- model or bot name, unless a default image model exists in config
- aspect ratio, size, or custom parameters if needed

Poe docs recommend media bots with:

- `stream: false`

Pass custom bot parameters such as aspect or size through `extra_body` when needed.

Default download directory:

- `/Users/dylanchen/Downloads/poe-images`

If config provides `image_download_dir`, use that instead.

Create the download directory if needed. Poe image bots may return a Markdown image link plus a Poe CDN URL in `choices[0].message.content`. Extract `https://...` URLs from the content and download image URLs to the configured directory.

When downloading Poe CDN image URLs, send a browser-like `User-Agent` header. Plain `urllib.request.urlretrieve()` can receive `HTTP 403 Forbidden` from Poe CDN even when the image generation succeeded.

Use this reusable Python pattern when you need a reliable local download:

```python
import json
import mimetypes
import pathlib
import re
import urllib.request

content = body["choices"][0]["message"]["content"] or ""
out_dir = pathlib.Path(config.get("image_download_dir", "/Users/dylanchen/Downloads/poe-images")).expanduser()
out_dir.mkdir(parents=True, exist_ok=True)

urls = list(dict.fromkeys(re.findall(r"https://[^\\s)]+", content)))
saved_paths = []

for i, url in enumerate(urls, start=1):
    req = urllib.request.Request(url, headers={"User-Agent": "Mozilla/5.0"})
    with urllib.request.urlopen(req, timeout=60) as response:
        data = response.read()
        content_type = response.headers.get("content-type", "image/png").split(";")[0]
    ext = mimetypes.guess_extension(content_type) or ".png"
    path = out_dir / f"poe-image-{i}{ext}"
    path.write_bytes(data)
    saved_paths.append(str(path))

print(json.dumps({"saved_paths": saved_paths}, ensure_ascii=False))
```

Notes:
- `body` is the parsed JSON response from Poe.
- `config` is the parsed local Poe config JSON.
- This works for the common Poe response pattern where image output appears as a Markdown image link plus a plain Poe CDN URL inside `choices[0].message.content`.
- Keep duplicate URLs deduplicated before downloading.
- If multiple images are returned, save all of them and report every local path.
- Infer the file extension from the `Content-Type` response header when possible, otherwise use `.png`.
- Return the saved local path plus model and prompt used.

If every download attempt fails, report that image generation succeeded but local download failed, include the HTTP status or error, and do not mark the image task as complete until the user chooses a next step.

Do not download into the current repository or a shared or synced folder unless the user explicitly asks and confirms the path.

## Error handling

Map common Poe errors to simple guidance:

- 400 `invalid_request_error`: malformed JSON or missing fields
- 401 `authentication_error`: invalid or expired Poe API key in config
- 402 `insufficient_credits`: Poe points or add-on points are exhausted
- 403 `moderation_error`: permission, authorization, or policy issue
- 404 `not_found_error`: wrong endpoint or model or bot name
- 408 `timeout_error`: model did not start in time
- 413 `request_too_large`: prompt exceeds context or size limit
- 429 `rate_limit_error`: rate limit; respect retry timing
- 500 / 502 / 529: provider or upstream issue

Do not leak the API key in error reports.

## Guardrails

- Use Poe search instead of Claude Code `WebSearch` for current-information tasks.
- If local Poe config is missing, stop and guide JSON setup instead of falling back.
- Never print or store the full API key in the repo.
- Keep the canonical config at `/Users/dylanchen/.claude/poe/config.json`.
- Do not guess private bot names.
- Do not claim Poe supports fields the docs say are ignored or unsupported.
- Use a cheap text verification call before expensive media calls when validating setup.
- Download generated images to `/Users/dylanchen/Downloads/poe-images` by default.
- Confirm before writing images into repo, synced, or shared locations.

## Verification checklist

After editing this skill, verify:

1. frontmatter has `name`, `description`, and `tools`
2. description includes default routing for current web information and local JSON config
3. the body says not to use Claude Code `WebSearch`
4. the body says missing local config should stop and guide setup, not fallback
5. the body defines `/Users/dylanchen/.claude/poe/config.json` as the canonical config
6. the body defines first-run setup from a user-provided absolute JSON path
7. image output defaults to `/Users/dylanchen/Downloads/poe-images`
8. no instruction writes secrets into a repository
