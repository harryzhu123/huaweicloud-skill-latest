# Executive Summary

`huaweicloud-skill` 是一个基于华为云 KooCLI (`hcloud`) 的可执行云操作 skill。它不是把云命令写进提示词，而是把 LLM 的云资源操作拆成可发现、可规划、可执行、可验证、可回归的工程链路。

## 一句话定位

让 agent 能用 `hcloud` 安全地查询、分析、规划和验证华为云资源操作；对会影响费用、网络、可用性或数据状态的变更，默认走 plan、dry-run、显式确认和后置验证，而不是直接提交。

## 为什么这件事有价值

云资源操作和普通代码生成不同，失败成本更高：

- 参数错了会直接请求云 API。
- region、project、profile、OBS 凭证等上下文错了会误判资源不存在。
- job 成功不等于资源可用。
- 部分服务本地 KooCLI metadata 不完整，模型容易猜参数。
- 写类操作可能产生费用、影响网络连通性或破坏数据状态。

这个 skill 的核心贡献，是把这些风险变成框架层的默认行为：先发现、再计划、再执行、最后验证，并且每一步都有机器可读输出。

## v0.2 的主要成果

v0.2 已经从 v0.1 的 ECS/基础工具能力，扩展为多服务执行框架：

| 指标 | 当前状态 |
| --- | --- |
| 已登记服务 | 16 个：ECS、IAM、VPC、IMS、KPS、EIP、ELB、EVS、NAT、RDS、CCE、CDN、DNS、SCM、OBS、CES |
| 通用发现操作 | 146 个 `query_operations` |
| 资源级查询操作 | 61 个 `resource_query_operations` |
| 变更操作入口 | 80 个 `change_operations` |
| 自动化测试 | 94 个单测通过 |
| 数据集回归 | 26 个 generated question 文件、448 个唯一 operation、38 条 Excel E2E 记录 |
| 发布版本 | `v0.2 / 0.2.0` |

## 架构亮点

```mermaid
flowchart LR
    Intent["User intent"] --> Registry["Registry control plane"]
    Registry --> Plan["Planner and risk gate"]
    Registry --> Query["Read-only query builders"]
    Plan --> SafeExec["Safe execution wrapper"]
    Query --> SafeExec
    SafeExec --> HCloud["hcloud / hcloud obs"]
    HCloud --> Diagnosis["Structured result and error_details"]
    Diagnosis --> Verify["Resource/job/readiness verification"]
    Verify --> Report["Auditable answer"]
    Dataset["generated_questions + data.xlsx"] --> Gates["Coverage and safety gates"]
    Gates --> Registry
```

### 1. Registry 控制面

`references/service-registry.json` 是机器可读服务能力索引。它把每个服务的 query、resource query、change operation、planner、verifier、playbook 和 known limits 统一登记。

这让 agent 不需要凭记忆猜“哪个服务能做什么”，而是先查 registry，再决定走发现、资源查询、readiness、planner 或专用 flow。

### 2. Safe exec 执行面

`hcloud_safe_exec.py` 统一包装真实命令执行：

- 默认 JSON 输出。
- 命令和输出脱敏。
- 结构化 stdout/stderr/return code。
- 解析 JSON。
- 生成 `error_details`，区分 credential、permission、region/project、quota、parameter、not_found、network、metadata、cloud_api 等错误。

这使 agent 能判断下一步是让用户修配置、换 region、补参数、处理权限，还是停止重试。

### 3. Guarded change 安全面

写类操作默认不直接执行。当前已有三层能力：

- ECS 专用闭环：创建 JSON 校验、dry-run、submit 命令、`ShowJob` 轮询、`ACTIVE` 验证。
- EIP 专用闭环：Plan -> dry-run -> guarded submit -> `ShowPublicip` verify。
- 多服务通用闭环：VPC、ELB、EVS、NAT、RDS、CDN、DNS、SCM 走 Plan -> dry-run -> guarded submit -> resource Show* verify -> service smoke。

通用 guarded flow 支持从 submit 结果提取资源 ID，也支持显式 `--verify-param KEY=VALUE`。缺少目标 ID 时返回 `missing_params`，不会猜资源。

### 4. 验证面

v0.2 明确区分几类验证：

- job 终态验证：例如 ECS `ShowJob`。
- 资源终态验证：例如 ECS `ListServersDetails` 达到 `ACTIVE`。
- 资源级 Show* 验证：例如 `ShowSecurityGroupRule`、`ShowListener`、`ShowVolume`、`ShowNatGatewayDnatRule`、`ShowDomain`、`ShowRecordSet`。
- 服务级 readiness：例如 VPC/RDS/ELB 等服务的一组只读状态检查。
- JSON verifier：对 EIP、VPC、ELB、EVS、NAT、RDS、CCE、CDN、DNS、SCM 等服务返回结构做 ID/name/status/CIDR/绑定关系验证。

核心原则是：请求提交成功不等于业务完成；必须继续验证目标资源状态。

### 5. 数据集回归面

`scripts/check_question_coverage.py` 把外部人造问题集和 Excel E2E 验证集变成质量门禁：

- 检查 JSON schema。
- 检查 CRUD 类型和风险分类。
- 检查 registry 覆盖率。
- 检查 Excel 里的验证 operation 是否有可执行路径。
- 标记外部验证依赖和副作用验证步骤。

这让服务覆盖不是靠口头承诺，而是可以持续回归。

## 当前最能体现价值的场景

- 查询现有云资源：稳定走 JSON 输出、脱敏和错误诊断。
- 创建 ECS 前做依赖发现和 JSON 校验：避免占位符、数量、登录方式、安全组等常见风险。
- EIP/VPC/ELB/EVS/NAT/RDS/CDN/DNS/SCM 变更前做风险计划：先给 dry-run/submit/verify 命令链，不直接动资源。
- OBS 场景：识别 OBS 不走普通 OpenAPI-style metadata，而是走 `hcloud obs`/obsutil 专用适配器。
- 排查配置问题：普通 hcloud profile 和 OBS obsutil 凭证分开诊断。

## 和普通云操作 agent 的区别

| 普通做法 | 本 skill 的做法 |
| --- | --- |
| 让模型直接拼 hcloud 命令 | 通过 registry、metadata 和脚本生成命令 |
| 查询和写操作混在一起 | 查询、资源级查询、planner、guarded submit 分层 |
| 提交成功就说完成 | job、资源状态、readiness 分层验证 |
| 失败后靠自然语言猜原因 | `error_details` 给机器可读错误类别 |
| 覆盖多少靠人工印象 | generated questions、Excel E2E、契约测试回归 |
| OBS 当普通服务处理 | OBS 走 `hcloud obs`/obsutil 专用路径 |

## 已知边界

- ECS 是完整度最高的闭环；其他服务已具备广度优先的 P0 风险门禁，但复杂业务语义 verifier 还需要继续扩展。
- 非 ECS 服务的部分 KooCLI operation detail 在本地 metadata 中不完整，所以 v0.2 优先采用显式参数和 planner-first。
- 通用 Show* 后置验证确认基础资源状态，不等同于完整业务验收。
- 所有真实写操作仍需要用户按具体资源、region、project、风险和回滚方式确认。

## 下一阶段路线

1. 扩展更多服务专用 verifier，把通用 Show* 验证升级为更强的业务语义验证。
2. 增加更多真实只读样本和 dry-run 样本，继续校准 registry 和参数白名单。
3. 对高频服务补充更完整 playbook，尤其是 ELB、RDS、EVS、NAT、DNS、CDN。
4. 把 run journal 用到更多多步操作中，增强真实变更的审计和恢复能力。
