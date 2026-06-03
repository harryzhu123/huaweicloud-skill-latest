# ELB HTTP Backend Readiness Playbook

## 目标

让公网或私网 ELB 的 HTTP 后端真正进入 `ONLINE`，并用入口地址完成协议验收。

## 适用场景

- 创建 ELB、listener、pool、member、health monitor
- 后端是 ECS 上的 HTTP 服务
- `operating_status` 为 `OFFLINE`、`NO_MONITOR`、`CONNECT_FAILED` 或公网访问超时

## 创建顺序

1. 确认 VPC、子网和后端 ECS 的内网地址。
2. 先让后端 ECS 服务可用，推荐使用 `ecs-user-data-service-readiness.md`。
3. 安全组至少允许：
   - 外部到 ELB listener 端口
   - ELB 到后端 ECS 的服务端口
4. 创建 ELB、listener、pool、member、health monitor。
5. 轮询 member 到 `ONLINE`。
6. 用 ELB 入口地址做 HTTP 探测。

## HTTP/Web 入口来源限制

不要把“HTTP 可访问”自动实现为安全组 `0.0.0.0/0`。对以下入方向端口，必须使用受限来源 CIDR：

- `80`、`443`
- 常见应用端口 `3000`、`5000`、`8000`、`8080`

推荐来源包括用户明确提供的客户端 CIDR、办公网/VPN CIDR、CDN/WAF/ELB 来源范围、同 VPC 私网 CIDR 或后端健康检查来源。若用户没有提供来源范围，应先确认，不要提交全网开放规则。

## CONNECT_FAILED 排查

遇到后端 `CONNECT_FAILED`，按顺序检查：

1. member address 是否是后端 ECS 的正确私网 IP。
2. member subnet 是否使用后端 ECS 网卡所属 subnet/neutron subnet。
3. 后端安全组是否允许来自 ELB/同 VPC 的目标端口。
4. 后端 ECS 上是否真的有进程监听端口，例如 Nginx 监听 80。
5. health monitor 的 protocol、port、path 是否与后端服务一致。

每修复一项后再轮询健康状态，不要重复创建新的 ELB 或后端 ECS。

## 保守轮询

- 轮询间隔建议 10 到 20 秒。
- 连续 6 到 10 次仍无状态变化时，进入失败分类和输出证据。
- 若发现新的可修复差异，可以修复一次后重置轮询。
- 若没有新证据，不要无限等待到外部超时。

## 最终输出

成功时给出：

- ELB ID、IP、provisioning status
- listener/pool/health monitor ID
- member ID、address、operating status
- `curl` 或 `web_fetch` 的 HTTP 状态

失败时不要说高可用已完成；应输出当前资源事实和剩余阻塞。
