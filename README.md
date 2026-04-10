# Poe

[中文说明](README-zh.md)

A Claude Code skill that routes Poe-related tasks through the Poe API.

## What this skill does

- Uses Poe for text and chat requests when the user asks for Poe
- Uses Poe Responses API with `web_search_preview` for search and current-information tasks
- Guides local JSON config setup when Poe config is missing
- Supports Poe image generation with local download handling

## Included files

- `skill.md`: the skill definition and operating instructions
- `README.md`: English overview
- `README-zh.md`: Chinese overview
- `LICENSE`: open-source license
- `.gitignore`: local ignore rules

## Important security note

This repository does not include local Poe config files or API keys.
Store your real Poe credentials in a private local JSON file outside the repository.

## Local config location used by the skill

The skill expects a private local config at:

- `/Users/dylanchen/.claude/poe/config.json`

Minimum shape:

```json
{
  "api_key": "your_poe_api_key"
}
```

## Quick start

1. Copy this skill into your Claude Code skills directory.
2. Create your private Poe config JSON outside the repository.
3. Make sure the config contains a non-empty `api_key`.
4. Use Claude Code with Poe-related requests.

## Notes

- Do not commit secrets.
- Do not put Poe config into a tracked project.
- For web-backed tasks, this skill prefers Poe search over Claude Code web search.
