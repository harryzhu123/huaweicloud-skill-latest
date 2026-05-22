# Huawei CLI Local Meta Discovery

## 目标

在 live help 不稳定、`APIE_ERROR` 常出现的环境里，尽量利用本地 `~/.hcloud/metaRepo` 做 discovery。

## 为什么需要这一步

当前环境已经验证到：

- `hcloud ECS --help` 可以列出 operation
- 但很多 `hcloud <service> --help` 或 `hcloud <service> <operation> --help` 会因为 API Explorer 元数据失败而拿不到完整信息

因此，必须把“本地 meta cache discovery”作为正式流程的一部分。

## 推荐脚本

```bash
python3 scripts/hcloud_meta_lookup.py --service=ECS --pretty
```

## 脚本能做什么

### 1. 列本地已知服务

```bash
python3 scripts/hcloud_meta_lookup.py --list-services --limit=20 --pretty
```

返回：

- `services_en.json` 里的服务名
- 服务描述
- 分类
- 是否全局服务
- 是否已在本地缓存出 template 目录

### 2. 看某个 service 的本地缓存情况

```bash
python3 scripts/hcloud_meta_lookup.py --service=ECS --pretty
```

返回：

- 本地是否缓存了该 service
- 缓存了多少 operation 摘要
- 哪些 operation 有详细元数据
- endpoint / region 信息

### 3. 看某个 operation 的本地细节

```bash
python3 scripts/hcloud_meta_lookup.py \
  --service=ECS \
  --operation=ListFlavors \
  --region=cn-north-4 \
  --pretty
```

返回：

- 请求方法
- 请求路径
- 参数位置
- 参数类型
- 是否必填

## `--allow-help-fallback`

当本地没有 cache 时，可以尝试：

```bash
python3 scripts/hcloud_meta_lookup.py --service=IMS --allow-help-fallback --pretty
```

用途：

- 至少拿到 `Usage` 和 service 级失败信息

但当前环境下，它通常只能拿到：

- service 名
- `Usage`
- `APIE_ERROR`

也就是说，这一步不是万能补救，只是把失败上下文结构化。

## 当前环境的实际覆盖情况

- `ECS`
  - 有本地 operation 摘要缓存
  - 有少量 operation 详细缓存
- `IAM`
  - 目前只有 endpoint 级缓存
- `VPC` / `IMS` / `KPS`
  - 当前机器无本地 template cache
  - service help fallback 也会被网络限制卡住

## 如何在 workflow 里使用

推荐顺序：

1. `hcloud_context_inspect.py`
2. `hcloud_meta_lookup.py`
3. `hcloud --help`
4. `hcloud <service> --help`
5. `hcloud_safe_exec.py`

含义：

- 先看本地缓存里有什么
- 再决定是否继续依赖 live help

## 不要做的事

- 不要看到 service 在 `services_en.json` 里，就假设本地一定有 operation 详情
- 不要把没有本地 detail cache 的 operation 参数直接写死
- 不要忽略 `detail_cached=false` 这一信号
