# Poe

[English README](README.md)

> 让你的 Claude Code 可以利用 Poe 来浏览网页获取最新的信息+使用文生图模型。

这是一个 Claude Code skill，适合想在 Claude Code 里直接用 Poe 搜网页、拿最新信息、跑文生图的人。

## 这玩意是干啥的 (What this does)

- 把搜索和最新信息这类任务转到 Poe Responses API 的 `web_search_preview`
- 把普通 Poe 对话任务转到 Poe chat completions
- 运行时直接读取仓库里的 `config.json`
- 在请求前提醒你填写 `api_key` 和 `image_download_dir`
- 生成图片后，按 `config.json` 里写的目录下载到本地

## 能拿它干啥 (Use cases)

- “用 Poe 查一下这个”
- “帮我找最新的 Claude Code 文档”
- “用 Poe 搜今天的 AI 新闻”
- “用 Poe 生成一张图”
- “帮我把 Poe 配一下”

## 咋整，先跑起来 (Quick start)

1. 把这个 skill 放进 Claude Code 的 skills 目录。
2. 打开这个文件夹里的 `config.json`。
3. 填上 `api_key`。
4. 如果你要下载图片，再填上 `image_download_dir`。
5. 然后就可以在 Claude Code 里直接提 Poe 相关请求。

- 本地访问权限：你的本地环境得能正常访问 Poe。要是本地挂了 VPN 才能通，那就先挂好再用。
- 常见排查：如果还是用不起来，通常就是 `api_key` 没配对，或者保存 API Key 的配置文件路径、输出目录路径配错了。

## 它会问你啥 (What it asks from you)

第一次用之前，这个 skill 会要求你先改 `config.json`。

这些字段已经有默认值：
- `base_url`
- `default_text_model`
- `default_search_model`
- `default_image_model`

这些字段需要你自己填：
- `api_key`
- `image_download_dir`

每一次 Poe 请求，都应该先读这个仓库里的 `config.json`，并且使用里面 exactly 这个 `api_key`。

## 它会产出啥 (What it produces)

按任务不同，它会给你：
- 带引用的 Poe 搜索结果（如果 Poe 返回引用）
- 带模型信息的 Poe 文本结果
- 下载到本地目录里的图片文件

## 别指望它干这些 (Limits)

- 它不会用 Claude Code 的 `WebSearch` 来做最新信息查询。
- 如果 `config.json` 不存在，或者 `api_key` 是空的，它会先停下来。
- 如果你要下载图片，但 `image_download_dir` 是空的，它也会先停下来。
- 它不应该把真实 secret 存到别的地方，运行时就认这个本地 `config.json`。

## 这堆文件都是干啥的 (File map)

- `skill.md`：skill 规则和执行流程
- `config.json`：运行时配置模板
- `README.md`：英文说明
- `README-zh.md`：中文说明

## 它咋这么设计 (Design choices)

- 查最新信息时，默认走 Poe search。
- 运行时配置只认仓库里的 `config.json`。
- 缺配置就停下来问，不偷偷 fallback。
