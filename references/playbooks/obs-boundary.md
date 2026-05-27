# OBS hcloud obs Boundary

## 当前结论

`data-by-changping/data.xlsx` 的人工 E2E 数据里包含 OBS 桶和生命周期规则任务。OBS 不出现在普通 KooCLI service metadata 中，因此不能用 `hcloud OBS ListBuckets` 这类 OpenAPI-style 命令。

KooCLI 集成了 obsutil，可通过 `hcloud obs` 管理 OBS 数据。本 skill 因此把 OBS 作为专用 runner：

- 只读查询：`scripts/hcloud_obs_readonly.py`
- planner-only 变更：`scripts/hcloud_obs_change_plan.py`

## 处理原则

- 不要生成 `hcloud OBS <Operation>` 命令。
- bucket list 用 `hcloud obs ls`，通过 `hcloud_obs_readonly.py --operation ListBuckets` 生成。
- bucket 级查询必须显式传 `--bucket`，例如 `--operation GetBucketLifecycle --bucket <bucket>`。
- OBS 输出是 obsutil 文本，不是标准 OpenAPI JSON；最终回复只摘要关键信息，不展开 bucket policy、生命周期细节或认证参数。
- bucket、lifecycle、policy 写类操作只生成 planner-only 命令；真实 submit 需要单独确认。

## 已支持能力

- `ListBuckets` -> `hcloud obs ls`
- `StatBucket` -> `hcloud obs stat obs://bucket`
- `GetBucketLifecycle` -> `hcloud obs lifecycle obs://bucket -method=get`
- `GetBucketPolicy` -> `hcloud obs bucketpolicy obs://bucket -method=get`
- `CreateBucket` planner -> `hcloud obs mb obs://bucket`
- `DeleteBucket` planner -> `hcloud obs rm obs://bucket`
- `PutBucketLifecycle` / `DeleteBucketLifecycle` planner
- `PutBucketPolicy` / `DeleteBucketPolicy` planner

## 验证注意

- `hcloud obs` 会写 obsutil 日志；受限沙箱里可能出现 `.obsutil_log` 写入权限警告。
- 如果 live 查询失败，优先检查 obsutil endpoint、AK/SK/token、`.obsutilconfig` 和网络。
- 不要把 OBS 写类 planner 的 submit 命令当成已经执行。
