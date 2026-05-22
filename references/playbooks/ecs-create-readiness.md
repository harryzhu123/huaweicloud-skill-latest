# ECS Create Readiness Playbook

## 目标

在真正创建 ECS 前，把依赖项和购买约束查清楚，避免直接拼 `CreateServers` 失败。

## 适用场景

- 用户要创建 ECS
- 用户要生成创建命令或 `cli-jsonInput`
- 用户要评估某个 region / AZ / flavor 是否可用

## 已验证可直接参考的 ECS operation

- `ListServerAzInfo`
- `ListFlavors`
- `ListFlavorSellPolicies`
- `CreateServers`
- `CreatePostPaidServers`

## 默认执行顺序

### 1. 上下文确认

先确认：

- 当前 region
- 当前 project
- 当前 profile

推荐：

```bash
python3 scripts/hcloud_context_inspect.py --pretty
```

### 2. 可用区确认

```bash
python3 scripts/hcloud_safe_exec.py \
  --service ECS \
  --operation ListServerAzInfo \
  --arg=--cli-region=cn-north-4 \
  --arg=--project_id=<project-id> \
  --arg=--cli-output=json \
  --expect-json \
  --pretty
```

### 3. 规格确认

```bash
python3 scripts/hcloud_safe_exec.py \
  --service ECS \
  --operation ListFlavors \
  --arg=--cli-region=cn-north-4 \
  --arg=--project_id=<project-id> \
  --arg=--limit=50 \
  --arg=--cli-output=json \
  --expect-json \
  --pretty
```

### 4. 售卖策略确认

```bash
python3 scripts/hcloud_safe_exec.py \
  --service ECS \
  --operation ListFlavorSellPolicies \
  --arg=--cli-region=cn-north-4 \
  --arg=--project_id=<project-id> \
  --arg=--limit=50 \
  --arg=--cli-output=json \
  --expect-json \
  --pretty
```

## 还需要确认的外部依赖

除了 ECS 本身，还要确认：

- 镜像
- 密钥对
- VPC
- 子网
- 安全组
- 根盘和数据盘类型

当前首版 skill 对这些依赖不硬编码 operation 名，而是要求先发现当前 CLI 中可用的 operation：

- `hcloud IMS --help`
- `hcloud KPS --help`
- `hcloud VPC --help`

如果当前环境下 service 级帮助都因 metadata 失败拿不到，就退回本地缓存和 raw materials，不要直接猜。

## 创建命令构造原则

### 1. 默认不要直接手拼大 body

优先：

- `--skeleton`
- `--cli-jsonInput`

### 2. 默认先 `--dryrun`

例如：

```bash
python3 scripts/hcloud_safe_exec.py \
  --service ECS \
  --operation CreateServers \
  --arg=--cli-region=cn-north-4 \
  --arg=--dryrun \
  --json-input-file=<path-to-json>
```

### 3. 真执行前先讲清前提

建议在真正执行前至少说明：

- 用哪个 region / project
- 用哪个 flavor / AZ
- 用哪个 image / keypair / subnet
- 这是试运行还是真实创建

## 不要做的事

- 不要在镜像、网络、keypair 未确认时直接创建
- 不要把几十个参数都硬塞进一行命令
- 不要先真执行再补解释
