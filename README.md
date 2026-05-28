# huaweicloud-skill

`huaweicloud-skill` 是一个面向华为云 KooCLI / `hcloud` 的 Agent Skill。它让通用 Agent 能够用更安全、可审计、可复现的方式发现华为云 API、查询资源、诊断配置问题，并在需要时规划受保护的变更流程。

它适合这些场景：

- 你希望 Agent 直接基于 `hcloud` 操作华为云，而不是靠记忆猜 API。
- 你需要先盘点账号、区域、项目和资源，再决定下一步动作。
- 你希望变更前有 dry-run、风险识别、确认门禁和变更后验证。
- 你希望把认证、区域、项目、参数、输出格式等 CLI 问题转成 Agent 能理解的结构化错误。

## 快速开始

### 1. 准备 KooCLI

先确认本机可以运行 `hcloud`：

```bash
hcloud version
hcloud configure list
```

如果还没有安装或配置 KooCLI，请参考华为云官方文档：

- [KooCLI 国际站文档](https://support.huaweicloud.com/intl/zh-cn/cli/index.html)
- [华为云支持中心](https://support.huaweicloud.com/intl/zh-cn/)

如果要使用 OBS 能力，还需要让 `hcloud obs ...` 或 `obsutil` 使用同一套可用的 AK/SK。OBS 的认证错误会被保留在结构化输出里，便于 Agent 继续诊断。

### 2. 安装 Skill

安装后，在支持本地 Skill 的 Agent 中启用 `huaweicloud-skill`。例如 OpenClaw：

- [ClawHub: huaweicloud](https://clawhub.ai/zfish-lu/huaweicloud)
- [OpenClaw 技能市场：huaweicloud-skill](https://github.com/OpenClawAgent/OpenClaw/blob/main/docs/skill-marketplace.md#available-skills)

你也可以直接在仓库中运行脚本进行验证：

```bash
python3 scripts/hcloud_context_inspect.py --pretty
```

### 3. 让 Agent 使用它

可以直接用自然语言说明目标，Agent 会按 Skill 的规则先检查上下文、发现服务和操作、构造命令，再决定是否执行：

```text
使用 huaweicloud-skill，通过 hcloud 检查当前 profile、region、project，
然后列出当前区域的 ECS、VPC、EIP 概览。只读查询，不做任何变更。
```

## 使用样例

### 安全盘点当前账号资源

```text
使用 huaweicloud-skill，先检查当前 hcloud 配置，再盘点 cn-north-4
的 ECS、VPC、Subnet、EIP 和安全组资源，输出资源摘要和发现的风险点。
```

对应的只读命令可以这样开始：

```bash
python3 scripts/hcloud_service_readiness.py \
  --service ECS \
  --service VPC \
  --service EIP \
  --region cn-north-4 \
  --project-id <project-id> \
  --pretty
```

### 把 hcloud 报错转成可诊断结果

```text
使用 huaweicloud-skill 执行一次 ECS 列表查询。如果失败，请解释是认证、
区域、project_id、权限、参数还是输出格式问题，并给出下一步修复建议。
```

对应脚本会保留 `stderr`、退出码、JSON 解析状态和归一化后的 `error_details`：

```bash
python3 scripts/hcloud_safe_exec.py \
  --service ECS \
  --operation ListServersDetails \
  --arg=--cli-region=cn-north-4 \
  --arg=--cli-output=json \
  --expect-json \
  --pretty
```

### 创建 ECS 前先检查请求体

```text
使用 huaweicloud-skill 检查这个 ECS 创建 JSON 是否完整、安全、幂等。
不要直接创建云服务器，只输出缺失字段、风险点和推荐修复方式。
```

```bash
python3 scripts/hcloud_ecs_create_plan.py \
  --json-input-file examples/ecs-create-servers.cli-jsonInput.json \
  --operation CreateServers \
  --region cn-north-4 \
  --pretty
```

### 规划一次受保护的网络变更

```text
使用 huaweicloud-skill 规划新增一条安全组规则。先做 dry-run 和风险识别，
列出需要我确认的参数；在我明确确认前不要提交变更。
```

```bash
python3 scripts/hcloud_guarded_change_flow.py \
  --service VPC \
  --operation CreateSecurityGroupRule \
  --arg=--security_group_id=<security-group-id> \
  --region cn-north-4 \
  --project-id <project-id> \
  --pretty
```

### 快速确认 OBS 配置

```text
使用 huaweicloud-skill 检查 OBS 是否配置正确。如果 list bucket 失败，
请说明是 AK/SK、endpoint、权限还是账号侧问题。
```

```bash
python3 scripts/hcloud_obs_readonly.py \
  --operation ListBuckets \
  --limit 20 \
  --execute \
  --pretty
```

## 能力亮点

- **CLI-first**：优先基于本机 `hcloud` 的真实 service、operation 和 help 信息工作，减少凭空猜测。
- **结构化上下文**：自动整理 profile、region、project、认证模式、CLI 路径、版本和常见配置问题。
- **多服务发现**：通过 registry、playbook 和 discovery 工具覆盖 ECS、VPC、EIP、EVS、IMS、KPS、RDS、ELB、OBS、CDN、IAM 等常用服务。
- **安全执行封装**：`hcloud_safe_exec.py` 统一处理超时、敏感信息脱敏、JSON 解析、错误分类和输出裁剪。
- **变更门禁**：变更类流程默认包含 dry-run、风险识别、显式确认、执行记录和变更后验证。
- **开发者友好**：架构、扩展方式、服务覆盖策略和脚本契约都沉淀在 `docs/` 中，便于继续贡献。

## 用户协助配置项

为了让 Agent 能可靠完成云资源查询或变更，建议先准备好这些信息：

- `hcloud` 已安装，并且在当前终端可执行。
- 至少配置一个可用 profile，包含 AK/SK 或其他认证方式。
- 明确默认 region，例如 `cn-north-4`、`cn-east-3`。
- 对项目级服务准备 project id；可以通过 IAM、控制台或 `hcloud` 查询。
- 对账号级或全局服务确认是否需要特殊 endpoint 或 global project。
- OBS 查询需要额外确认 OBS 认证和 endpoint 是否可用。
- 变更类请求需要提供目标资源 id、期望状态和可接受的回滚方式。

如果配置有问题，Skill 会尽量把失败原因结构化，例如：

- 认证失败：AK/SK 不存在、签名失败、token 过期。
- 权限不足：IAM policy、项目权限、OBS bucket policy。
- 区域或 project 错误：region 不存在、project id 与 region 不匹配。
- 参数错误：缺少必填字段、字段名不符合当前 operation。
- 输出问题：命令成功但不是合法 JSON，或 stdout 被额外文本污染。

## 在常用 Agent 中使用

### OpenClaw

```bash
openclaw skills search huaweicloud
openclaw skills install zfish-lu/huaweicloud
openclaw skills list --eligible
```

在 OpenClaw 中提出这类请求即可触发：

```text
用 hcloud 帮我查一下 cn-north-4 当前有哪些 ECS 和 VPC。
```

```text
先规划一次给某台 ECS 绑定 EIP 的操作，只输出命令、风险点和验证方式，不要直接执行。
```

### Codex CLI / Codex App

把本仓库作为本地 Skill 安装或链接后，适合用于：

- 读取 `SKILL.md` 中的执行约束。
- 调用 `scripts/` 中的安全封装脚本。
- 扩展 `references/service-registry.json` 与 `references/playbooks/`。
- 运行 `tests/` 保持服务覆盖和脚本契约不退化。

### Claude Code

可以把本仓库放入 Claude Code 的 skills 目录，或在项目说明中引用 `SKILL.md`。推荐提示：

```text
请使用 huaweicloud-skill。所有华为云查询都必须通过 hcloud 或本仓库 scripts，
变更前必须先 dry-run，并等待我确认。
```

## 目录结构

```text
.
├── SKILL.md                         # Agent 入口说明和执行规则
├── README.md                        # 用户快速上手文档
├── CHANGELOG.md                     # 版本变更记录
├── RELEASE_NOTES.md                 # 发布说明
├── docs/                            # 开发者文档：架构、实现和覆盖策略
├── examples/                        # 常用 hcloud JSON 输入样例
├── references/
│   ├── service-registry.json        # 服务能力注册表
│   └── playbooks/                   # 常见服务工作流
├── scripts/                         # hcloud 上下文、发现、查询、规划和执行封装
└── tests/                           # 单元测试、契约测试和人工验证记录
```

## 开发与验证

开发者可以先阅读：

- [`docs/technical-overview.md`](docs/technical-overview.md)
- [`docs/architecture.md`](docs/architecture.md)
- [`docs/implementation-details.md`](docs/implementation-details.md)
- [`docs/data-and-coverage.md`](docs/data-and-coverage.md)

常用本地验证：

```bash
python3 -m unittest discover tests
python3 scripts/check_materials_drift.py --pretty
git diff --check
```

如果要验证真实华为云账号，请先确认当前 profile、region、project 和权限，再从只读脚本开始：

```bash
python3 scripts/hcloud_context_inspect.py --pretty
python3 scripts/hcloud_readonly_smoke.py --service VPC --region cn-north-4 --project-id <project-id> --pretty
```

## 贡献服务覆盖

新增服务时建议按这个顺序迭代：

1. 在 `references/service-registry.json` 中补齐 service、operation、scope、常用 discovery 和 verifier。
2. 在 `references/playbooks/` 增加服务工作流，明确只读查询、变更前检查和变更后验证。
3. 如有特殊参数、风险或验证逻辑，补充专用脚本。
4. 在 `tests/` 中增加契约测试和核心路径测试。
5. 更新 `docs/` 中的架构、实现或覆盖说明。

## License

MIT License. See [LICENSE](LICENSE).
