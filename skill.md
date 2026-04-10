---
name: poe
description: Default Poe API and web-search routing skill. Use this whenever the user asks to use Poe, call Poe models or bots, initialize or update the repo-local Poe config, generate images with Poe, or answer anything that needs web search, current information, latest news, latest docs, or online lookup. For search and current-information tasks, use Poe Responses API `web_search_preview` instead of Claude Code WebSearch. If the repo `config.json` is missing or incomplete, stop and guide the user to fill it before proceeding.
tools: Bash, Read, Write
---

# Poe

Use this skill as the default Poe workflow for this repository.

Core policy:
- If the user asks for Poe, use Poe.
- If the task needs current information, recent events, latest docs, online lookup, or web search, route it through Poe search.
- Do not use Claude Code `WebSearch` for search or current-information tasks.
- If repo `config.json` is missing, invalid, or incomplete for the requested task, stop and guide setup instead of falling back.

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
4. **Repo config missing or incomplete** → guide `config.json` setup and stop. Do not use `WebSearch`.

This rule applies even if the user does not explicitly mention Poe.

## Repo config standard

Use the tracked config file in this skill folder:

- `config.json`

This file is the canonical runtime config for the skill.
The skill must use the `api_key` stored in exactly this file for every Poe request.
Do not switch to a different key source unless the user explicitly asks to update `config.json`.

Required config shape:

```json
{
  "api_key": "",
  "base_url": "https://api.poe.com/v1",
  "default_text_model": "Claude-Sonnet-4.6",
  "default_search_model": "GPT-5.4",
  "default_image_model": "GPT-Image-1.5",
  "image_download_dir": ""
}
```

Field rules:
- `api_key`: required before any Poe request
- `base_url`: default is already provided; use it unless the user changes it
- `default_text_model`: default is already provided; use it unless the user changes it
- `default_search_model`: default is already provided; use it unless the user changes it
- `default_image_model`: default is already provided; use it unless the user changes it
- `image_download_dir`: required for image download tasks

Runtime rule:
- read `config.json` from this repo before every Poe request
- use exactly the `api_key` from `config.json`
- do not read API credentials from environment variables, user profile files, caches, or fallback files unless the user explicitly changes the setup
- do not write real credentials anywhere except the user's local `config.json` in this skill folder

## Setup guidance

If `config.json` is missing:

1. Tell the user this skill requires the repo-local `config.json` file.
2. Recreate `config.json` with the required shape shown above.
3. Tell the user to fill in `api_key`.
4. Tell the user to fill in `image_download_dir` if they want image generation downloads.
5. Re-read `config.json` before the next Poe request.

If `config.json` exists but `api_key` is empty:

1. Stop before any Poe request.
2. Tell the user to open `config.json` in this skill folder.
3. Ask them to fill in `api_key` there.
4. Re-read `config.json` after they confirm it was updated.
5. Use exactly that updated `api_key`.

If the task is image generation and `image_download_dir` is empty:

1. Stop before downloading any image.
2. Tell the user to fill in `image_download_dir` in `config.json`.
3. Re-read `config.json` after they confirm it was updated.
4. Save images to exactly that configured directory.

Never ask the user to paste the raw API key into chat if they can edit `config.json` directly.

## Reading config before requests

Before any Poe request:

1. Read `config.json`
2. Parse JSON
3. Extract:
   - `api_key`
   - `base_url`
   - `default_text_model`
   - `default_search_model`
   - `default_image_model`
   - `image_download_dir`
4. Validate that `api_key` is a non-empty string after trimming whitespace
5. If the task includes image download, validate that `image_download_dir` is also a non-empty string after trimming whitespace
6. If validation fails, stop and guide the user to edit `config.json`
7. For the current request, use exactly the validated `api_key` from `config.json`

Do not print the full key. If you need to confirm it exists, show only a masked form such as `p10...abcd`.

## API basics

Use Poe's OpenAI-compatible base URL from `config.json`.
If `base_url` is missing or empty, use:

- `https://api.poe.com/v1`

Use these headers:
- `Authorization: Bearer <api_key_from_config>`
- `Content-Type: application/json`

## Poe search workflow

Use this for current-information and web-search tasks.

Endpoint:

- `POST <base_url>/responses`

Payload shape:

```json
{
  "model": "GPT-5.4",
  "input": "What are the latest AI news today?",
  "tools": [{"type": "web_search_preview"}]
}
```

If `default_search_model` is present and non-empty in `config.json`, use it. Otherwise ask when model choice matters.

Output format:
1. concise answer
2. sources or citations if Poe returns them
3. model used

If Poe returns no citations, say that Poe did not include citations in the response.

## Text or chat workflow

Use this when the user wants a Poe model call that does not require search.

Endpoint:

- `POST <base_url>/chat/completions`

Payload shape:

```json
{
  "model": "Claude-Sonnet-4.6",
  "messages": [
    {"role": "user", "content": "Your prompt here"}
  ]
}
```

If `default_text_model` is present and non-empty in `config.json`, use it. Otherwise ask when model choice matters.

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
- model or bot name, unless a default image model exists in `config.json`
- aspect ratio, size, or custom parameters if needed

Before making the request, make sure `image_download_dir` in `config.json` is filled.
If it is empty, stop and ask the user to fill it.

Poe docs recommend media bots with:

- `stream: false`

Pass custom bot parameters such as aspect or size through `extra_body` when needed.

Download directory rule:

- save generated images to the directory in `config.json` under `image_download_dir`

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
out_dir = pathlib.Path(config["image_download_dir"]).expanduser()
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
- `config` is the parsed repo `config.json`.
- This works for the common Poe response pattern where image output appears as a Markdown image link plus a plain Poe CDN URL inside `choices[0].message.content`.
- Keep duplicate URLs deduplicated before downloading.
- If multiple images are returned, save all of them and report every local path.
- Infer the file extension from the `Content-Type` response header when possible, otherwise use `.png`.
- Return the saved local path plus model and prompt used.

If every download attempt fails, report that image generation succeeded but local download failed, include the HTTP status or error, and do not mark the image task as complete until the user chooses a next step.

## Error handling

Map common Poe errors to simple guidance:

- 400 `invalid_request_error`: malformed JSON or missing fields
- 401 `authentication_error`: invalid or expired Poe API key in `config.json`
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
- If `config.json` is missing, stop and guide setup instead of falling back.
- If `api_key` in `config.json` is empty, stop and ask the user to fill it before any Poe request.
- If an image task needs downloads and `image_download_dir` in `config.json` is empty, stop and ask the user to fill it before downloading.
- Never print or store the full API key outside the user's local `config.json`.
- For every Poe request, use exactly the `api_key` from `config.json` after validation.
- Do not guess private bot names.
- Do not claim Poe supports fields the docs say are ignored or unsupported.
- Use a cheap text verification call before expensive media calls when validating setup.

## Verification checklist

After editing this skill, verify:

1. frontmatter has `name`, `description`, and `tools`
2. description includes repo-local config routing
3. the body says not to use Claude Code `WebSearch`
4. the body says missing or incomplete `config.json` should stop and guide setup, not fallback
5. the body defines `config.json` as the canonical runtime config
6. the body says `api_key` and `image_download_dir` are the fields the user must fill
7. the body says all Poe calls must use exactly that `api_key`
8. no personal paths or user-identifying strings appear in the published repo
9. no instruction writes real secrets outside the repo-local config flow
