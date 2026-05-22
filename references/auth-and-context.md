# Huawei CLI Auth and Context

## 目标

在执行任何华为云 CLI 任务前，先确认：

- 当前使用哪个 profile
- 当前 region 是什么
- 当前 project 和 domain 是否已显式配置
- 当前是 online 还是 offline mode
- 当前任务是项目级服务还是全局服务

## 认证优先级

根据 KooCLI 常见问题文档，认证优先级大致如下：

1. 命令中直接传入的 AK/SK 或临时安全凭证
2. 显式指定的 profile 或默认 profile
3. `ecsAgency`

含义：

- 如果命令里显式传了 AK/SK，它会压过 profile。
- 如果 profile 已经可用，不要为了默认上下文再追问 AK/SK。
- 如果高优先级认证解析失败，KooCLI 不会自动回退到低优先级认证。

## 默认检查顺序

推荐顺序：

1. `python3 scripts/hcloud_context_inspect.py --pretty`
2. `hcloud configure show`
3. `hcloud configure list --cli-output=json`

需要切 profile 时，再显式使用 `--cli-profile=<name>`。

## 关于 region

大多数任务都需要 `cli-region`。

默认规则：

- 命令里显式指定的 `--cli-region` 优先
- 命令里没指定时，才使用当前 profile 中的 region

因此：

- 如果当前任务跨 region，不要偷用默认 region
- 如果用户没说 region，但当前 profile 已有明确 region，可先按默认值工作，再在回复里说明当前使用范围

## 关于 project_id 和 domain_id

### `project_id`

适用于项目级服务。

例如：

- ECS
- VPC
- IMS
- EVS

如果当前 profile 里没有 `projectId`：

- 先确认当前任务是否真的需要它
- 如果需要，再通过当前上下文或服务侧发现路径补齐

### `domain_id`

适用于全局服务或全局认证场景。

文档明确提到：

- AK/SK 模式访问全局服务时，可能需要 `cli-domain-id`

因此：

- 如果看到错误提示在追 `cli-domain-id`，不要继续盲试项目级参数

## Offline Mode

KooCLI 支持 online 和 offline mode。

### Offline mode 的优点

- 固定脚本更稳定
- 已下载的离线元数据不会频繁变化

### Offline mode 的风险

- 新服务或新 operation 可能不存在
- 老缓存可能不包含最新参数

### 实际策略

- 当前任务是固定脚本式自动化：offline mode 通常更稳
- 当前任务是临时探索新服务或新 operation：online mode 更灵活

## 当前上下文缺失时的处理原则

### 可以默认继续的场景

- 当前 profile 明确可用
- region 明确
- 当前任务只是查询类

### 不应该硬推进的场景

- profile 不明确
- region 缺失
- 任务是费用敏感或高风险变更
- project 或 domain 明显缺失，且目标服务确实依赖它

## 推荐做法

### 先给出当前上下文摘要

例如：

- 当前 profile：`default`
- 当前 region：`cn-north-4`
- 当前 project：未显式配置
- 当前 mode：`AKSK`

### 再说明本轮作用域

例如：

- 本轮先按 `default` profile 和 `cn-north-4` 执行查询

这会让后续变更更可审查。
