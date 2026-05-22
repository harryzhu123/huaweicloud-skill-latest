# huaweicloud-skill Manual Validation 2026-04-23

本文件记录第二轮补强后的实际验证结果。

## 环境前提

- `hcloud` 可执行
- 当前 profile：`default`
- 当前 region：`cn-north-4`
- 当前环境网络对 API Explorer 和部分云服务域名解析受限

## 验证 1：本地 meta cache 服务列表

### Command

```bash
python3 scripts/hcloud_meta_lookup.py --list-services --limit=10 --pretty
```

### Result

- 成功返回 `services_en.json` 中的服务列表
- 能区分哪些服务有本地 template cache

## 验证 2：ECS 本地 operation 摘要

### Command

```bash
python3 scripts/hcloud_meta_lookup.py --service=ECS --limit=8 --pretty
```

### Result

- 成功返回 ECS 的本地 operation 摘要
- 当前环境下 `cached_operations_count=124`

## 验证 3：ECS `ListFlavors` 详细元数据

### Command

```bash
python3 scripts/hcloud_meta_lookup.py \
  --service=ECS \
  --operation=ListFlavors \
  --region=cn-north-4 \
  --pretty
```

### Result

- 成功返回：
  - 请求方法
  - 请求路径
  - 参数位置
  - 参数是否必填
  - `cn-north-4` 对应 endpoint

## 验证 4：查询类 dry-run

### Command

```bash
hcloud ECS ListFlavors \
  --cli-region=cn-north-4 \
  --project_id=0dd8cb41000000000000000000000000 \
  --dryrun
```

### Result

- 本地参数校验通过
- 成功打印 dry-run 请求
- 说明当前环境下至少这条 ECS 查询链可以做到：
  - 本地 meta lookup
  - 参数准备
  - 请求骨架验证

## 验证 5：`hcloud_safe_exec.py` 结果落盘

### Command

```bash
python3 scripts/hcloud_safe_exec.py \
  --command-part=configure \
  --command-part=list \
  --arg=--cli-output=json \
  --expect-json \
  --result-file=/tmp/hcloud_safe_exec_result.json \
  --parsed-json-file=/tmp/hcloud_safe_exec_parsed.json
```

### Result

- 成功打印结构化结果
- 成功写出：
  - `/tmp/hcloud_safe_exec_result.json`
  - `/tmp/hcloud_safe_exec_parsed.json`

## 验证 6：IMS / KPS / VPC 的 fallback 边界

### Commands

```bash
python3 scripts/hcloud_meta_lookup.py --service=IMS --allow-help-fallback --pretty
python3 scripts/hcloud_meta_lookup.py --service=KPS --allow-help-fallback --pretty
python3 scripts/hcloud_meta_lookup.py --service=VPC --allow-help-fallback --pretty
```

### Result

- 三个 service 都能在服务目录中被识别
- 当前机器没有对应本地 template cache
- help fallback 只能拿到 `Usage` 和 `APIE_ERROR`

### Meaning

- 这些服务当前更适合作为 discovery / planning / readiness guidance
- 不应伪装成已经有和 ECS 一样完整的本地参数级执行支持
