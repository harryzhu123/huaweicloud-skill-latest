# huaweicloud-skill Trigger Cases

## Positive Triggers

这些输入应该优先触发 `huaweicloud-skill`：

- 用 `hcloud` 帮我列出当前 region 的 ECS。
- 我这次明确要走 KooCLI，不要走 MCP。
- 帮我生成一个 `hcloud` 的 dry-run 创建命令。
- `hcloud` 报 `USE_ERROR`，你帮我排查一下。
- 帮我把华为云 CLI 的查询结果只保留关键字段。

## Negative Triggers

这些输入不应该优先触发 `huaweicloud-skill`：

- 帮我直接调用华为云 MCP 查询 ECS。
- 帮我用 Terraform 在华为云创建一台 ECS。
- 帮我通过自主 MCP 自然语言处理华为云任务。

## Borderline Cases

这些输入要看上下文：

- 帮我查华为云 ECS。
  - 如果上下文已经在谈 `hcloud`，应触发本 skill
  - 如果上下文已经在谈 MCP，不应硬切 CLI

- 帮我创建一台华为云 ECS。
  - 如果用户明确要 CLI，触发本 skill
  - 如果用户明确要 IaC，触发 `huawei-terraform-skill`
