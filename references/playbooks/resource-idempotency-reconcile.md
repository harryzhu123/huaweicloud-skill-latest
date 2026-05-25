# Resource Idempotency Reconcile Playbook

## 目标

把用户给出的资源名当作幂等键，避免失败重试时创建一批同名资源。

## 适用场景

- 用户要求创建或修复命名资源
- 上一次执行可能中断、超时或部分完成
- 查询发现同名 ECS、EIP、ELB、安全组、RDS、OBS 桶等资源

## 默认流程

### 1. 先按名称查询

创建前先调用对应 `List*` / `Show*`：

- ECS: `ListServers`
- EIP: `ListPublicips`
- ELB: `ListLoadBalancers`
- Security Group: `ListSecurityGroups`
- RDS: `ListInstances`
- VPC/Subnet: `ListVpcs` / `ListSubnets`

查询结果必须记录资源 ID、名称、状态、创建时间、region/project、关键绑定关系。

### 2. 选择 canonical resource

若没有同名资源，继续创建。

若只有一个同名资源：

- 规格、状态、网络接近目标：修复它
- 明显不属于目标或状态不可恢复：说明原因，除非用户授权删除，否则不要直接释放

若有多个同名资源，不要继续创建新的同名资源。选择 canonical resource 的顺序：

1. 状态健康或最接近健康终态
2. 规格、镜像、端口、VPC、子网最接近用户要求
3. 已绑定更多目标依赖，例如 EIP、ELB member、磁盘
4. 创建时间最新

其他同名资源只做只读记录，不要自动删除。

### 3. 修复而不是重建

对 canonical resource 执行缺口修复：

- 缺安全组规则：补规则
- 缺 EIP 绑定：绑定 EIP
- 缺 listener/member：补 ELB 依赖
- ECS `ACTIVE` 但应用不可达：优先排查 cloud-init、服务端口、安全组和协议探测

### 4. 最终输出

最终必须说明：

- 选定的 canonical resource ID
- 发现了多少同名资源
- 已修复哪些差异
- 哪些重复资源需要人工清理

## 不要做的事

- 不要在同名资源已存在时继续创建同名资源。
- 不要为了幂等直接删除旧资源，除非用户明确授权。
- 不要只按名称模糊匹配后宣布成功，必须核对状态和关键配置。
