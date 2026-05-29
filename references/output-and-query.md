# Huawei CLI Output and Query

## 目标

让 `hcloud` 的返回结果：

- 稳定
- 易解析
- 易筛选
- 不把大量无关数据直接塞回对话

## 一、默认输出格式

### 推荐默认值

- 机器处理：`json`
- 人工浏览：`table`
- 管道拼值：`tsv`

如果返回结果后面还要被脚本或代码继续处理，默认强制：

```bash
--cli-output=json
```

## 二、`--cli-query`

`--cli-query` 基于 JMESPath。

典型用途：

- 只保留某些字段
- 只取前几项
- 做简单投影
- 降低结果体积

### 示例 1：只看第一条

```bash
--cli-query=items[0]
```

### 示例 2：只取部分字段

```bash
--cli-query=items[].{Name:name,Status:status}
```

### 示例 3：取数组中的某个值

```bash
--cli-query=servers[].id
```

## 三、默认输出策略

### 查询类

先这样想：

1. 用户真正要什么字段
2. 是不是只要摘要
3. 是不是只要 Top N

不要默认把整份原始 JSON 直接返回。

### 表格类

`table` 适合人工看，不适合后续再喂脚本。

如果已经选择 `table`：

- 可选 `--cli-output-num`
- 但不要把它当成后续自动处理输入

## 四、对 agent 的推荐规则

- 默认先 `json`
- 先用查询参数限制范围，再用 `--cli-query` 提炼
- 真需要表格给用户看时，再切换成 `table`
- 如果当前结果很大，优先汇总为：
  - 数量
  - 状态分布
  - 满足条件的候选项
  - Top N

## 五、高风险大输出 API

以下 API 容易返回很大的列表或宽字段对象，执行前要先判断是否真的需要全量进入上下文：

- `IMS ListImages`：镜像列表，公共镜像和共享镜像可能很多。
- `ECS ListFlavors`：规格列表，字段多，按区域和规格族展开后可能很大。
- `ECS ListFlavorSellPolicies`：售卖策略列表，常需要和规格、AZ 做交叉分析。

推荐处理顺序：

1. 能用服务端过滤时，先加过滤参数，例如 `limit`、`marker`、`name`、`status`、`visibility`、`architecture`、`flavor_id`、`availability_zone` 等，真实参数以 operation help 为准。
2. 只需要少量展示时，再用 `--cli-query` 做字段投影或 Top N。
3. 需要全量核验、跨 API join、排查售卖策略差异时，优先把完整结果落盘，不要把完整 JSON 打印回对话。
4. 落盘后只向对话返回小摘要：命令是否成功、文件位置、顶层 key、候选数组条数、少量关键字段样本、下一步筛选方式。
5. 后续读取文件时也不要 `cat` 全量 JSON；用 `jq`、短脚本或 `--cli-query` 等方式只取必要字段。

推荐落盘命令形态：

```bash
python3 scripts/hcloud_safe_exec.py \
  --service ECS \
  --operation ListFlavors \
  --arg=--cli-region=<region> \
  --arg=--project_id=<project-id> \
  --arg=--cli-output=json \
  --expect-json \
  --result-file=<result-json-file> \
  --parsed-json-file=<parsed-json-file>
```

注意：`hcloud_safe_exec.py` 会把结构化结果写入 `result-file`，把解析后的主体写入 `parsed-json-file`。如果调用方同时要求 `--pretty`，仍要确认不会把巨大 `parsed_json` 打回当前上下文；必要时不加 `--pretty`，或者先用更小的 `limit` 确认结构后再全量落盘。

建议摘要字段：

- `ListImages`：`id`、`name`、`status`、`visibility`、`__platform`、`__imagetype`、`os_version`、`min_disk`、`min_ram`、`created_at`。
- `ListFlavors`：`id`、`name`、`vcpus`、`ram`、`disk`、`os_extra_specs` 中和虚拟化、规格族、可用区相关的字段。
- `ListFlavorSellPolicies`：`flavor_id` / `flavor_name`、`availability_zone`、`sell_status`、`sell_mode`、`spot_options`、限制原因类字段。

## 六、推荐例子

### 1. 看配置项列表

```bash
hcloud configure list --cli-output=json
```

### 2. 看 ECS 规格前 20 条

```bash
hcloud ECS ListFlavors \
  --cli-region=cn-north-4 \
  --project_id=<project-id> \
  --limit=20 \
  --cli-output=json
```

### 3. 只看规格名和可用区

```bash
hcloud ECS ListFlavors \
  --cli-region=cn-north-4 \
  --project_id=<project-id> \
  --limit=20 \
  --cli-output=json \
  --cli-query=flavors[].{Name:name,AZ:os_extra_specs.ecsperformancetype}
```

上面的字段表达式只是示意，真实字段名应以当前返回体为准。

## 七、何时不要强上 `--cli-query`

以下情况不要先写复杂表达式：

- 还不知道返回体结构
- 当前 operation 帮助都拿不到
- 当前结果很可能是错误体而不是正常数据

此时先拿一版小样本原始 JSON，再决定 query。

## 八、结果落盘

当查询结果后面还要继续被脚本消费时，可以直接用包装脚本落盘：

```bash
python3 scripts/hcloud_safe_exec.py \
  --command-part=configure \
  --command-part=list \
  --arg=--cli-output=json \
  --expect-json \
  --result-file=/tmp/hcloud_safe_exec_result.json \
  --parsed-json-file=/tmp/hcloud_safe_exec_parsed.json
```

用途：

- `result-file`
  - 保存完整结构化执行结果
- `parsed-json-file`
  - 只保存解析后的 JSON 主体
