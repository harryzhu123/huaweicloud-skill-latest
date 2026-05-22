# Huawei CLI Playbooks

这里收纳的是面向具体任务的执行手册。优先按用户当前目标选一个最贴近的 playbook，再回到 `references/workflow.md` 补通用规则。

## 索引

- `ecs-create-readiness.md`
  - 创建 ECS 前的依赖确认、规格和售卖策略检查。
- `ecs-inventory.md`
  - 查询当前 scope 下的 ECS 实例并整理成可读摘要。
- `iam-context-bootstrap.md`
  - 在执行云侧业务前先确认当前 profile、region、project 和认证上下文。
- `ims-image-discovery.md`
  - 创建 ECS 前的镜像发现路径和当前环境约束。
- `kps-keypair-discovery.md`
  - 创建 ECS 或 SSH 登录前的密钥对发现与风险检查。
- `vpc-network-readiness.md`
  - 面向网络前置条件的 readiness 检查方法。
- `vpc-resource-discovery.md`
  - 面向 VPC、子网、安全组等资源的 discovery 路径。

## 选择建议

- 目标是查现网 ECS：先看 `ecs-inventory.md`
- 目标是创建 ECS：先看 `ecs-create-readiness.md`
- 卡在上下文或认证：先看 `iam-context-bootstrap.md`
- 卡在镜像、密钥对或网络依赖：按 `ims`、`kps`、`vpc` 对应 playbook 进入
