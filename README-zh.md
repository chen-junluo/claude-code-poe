# Poe

[English README](README.md)

这是一个 Claude Code skill，用来把与 Poe 相关的任务路由到 Poe API。

## 这个 skill 做什么

- 当用户明确要用 Poe 时，使用 Poe 做文本或对话请求
- 对需要搜索或当前信息的任务，使用 Poe Responses API 的 `web_search_preview`
- 当本地 Poe 配置缺失时，停止执行并引导用户完成本地 JSON 配置
- 支持 Poe 文生图和本地下载

## 仓库包含内容

- `skill.md`：skill 定义与执行说明
- `README.md`：英文说明
- `README-zh.md`：中文说明
- `LICENSE`：开源许可证
- `.gitignore`：本地忽略规则

## 安全说明

这个仓库不包含本地 Poe 配置文件，也不包含 API key。
请把真实凭证保存在仓库之外的私有本地 JSON 文件中。

## skill 使用的本地配置路径

该 skill 默认读取这个私有本地配置：

- `/Users/dylanchen/.claude/poe/config.json`

最小格式：

```json
{
  "api_key": "your_poe_api_key"
}
```

## 快速开始

1. 把这个 skill 放到 Claude Code 的 skills 目录。
2. 在仓库外创建私有 Poe 配置 JSON。
3. 确认配置里有非空的 `api_key`。
4. 在 Claude Code 中直接发起 Poe 相关请求。

## 备注

- 不要提交任何 secret。
- 不要把 Poe 配置写进受版本控制的项目。
- 对需要联网搜索的任务，这个 skill 优先使用 Poe search，而不是 Claude Code web search。
