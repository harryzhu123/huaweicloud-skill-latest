# Huawei CLI Examples

这里放的是 `huaweicloud-skill` 的示例和模板，不是已经替你验证过的现网配置。

## 目标

这些示例主要解决三件事：

1. 避免把复杂创建参数硬塞进一行命令
2. 提供 `cli-jsonInput` 的稳定骨架
3. 给 dry-run 和实际执行提供可审查的起点

## 当前示例

- `ecs-create-servers.cli-jsonInput.json`
  - 适合 `CreateServers`
- `ecs-create-postpaid-servers.cli-jsonInput.json`
  - 适合 `CreatePostPaidServers`
- `ecs-create-dryrun.md`
  - 说明如何配合 dry-run 使用这些模板

## 使用方式

### 1. 复制模板

先复制一份模板，不要直接改原始示例。

### 2. 替换占位值

至少需要替换：

- `<project_id>`
- `<availability_zone>`
- `<flavor_id>`
- `<image_id>`
- `<subnet_id>`
- `<vpc_id>`
- `<security_group_id>`
- `<key_name>`

### 3. 先 dry-run

先用 dry-run 看请求是否能被本地校验通过。

### 4. 再真实执行

只有在依赖都确认过之后，才考虑真实创建。

## 注意

- 模板字段是“可复用骨架”，不是“所有字段都必须保留”
- 删除你不需要的字段，比保留一堆不确定字段更稳
- 模板里的字段名保持华为云 API 原生风格，可能同时出现 `camelCase`、`snake_case` 和类似 `vpcid` 的供应商字段；不要擅自“规范化”重命名
- 创建类任务涉及费用和资源变更，默认不要跳过 dry-run
