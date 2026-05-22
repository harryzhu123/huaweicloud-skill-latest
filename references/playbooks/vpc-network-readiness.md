# VPC Network Readiness Playbook

## 目标

在涉及 ECS 创建、迁移或网络排障时，先确认网络侧前置条件是否完整。

## 适用场景

- 创建 ECS 前检查网络依赖
- 排查实例无法连通
- 确认 VPC、子网、安全组是否具备继续执行条件

## 当前首版策略

由于当前环境下部分 service 级帮助可能依赖 live metadata，本 playbook 采用：

1. 先讲方法论
2. 再在当前 CLI 中动态发现具体 operation

## 标准步骤

### 1. 确认当前上下文

```bash
python3 scripts/hcloud_context_inspect.py --pretty
```

### 2. 发现当前 CLI 可用的 VPC operation

先尝试：

```bash
hcloud VPC --help
```

如果当前环境无法拿到完整 operation 列表：

- 不要直接猜 operation 名
- 回退到本地 meta cache 和 raw materials

### 3. 最少要确认的网络对象

- VPC
- 子网
- 安全组
- 可选的 EIP 或公网接入方式

## 与 ECS 创建准备的关系

当用户要创建 ECS 时，网络侧至少要回答这些问题：

1. 目标 VPC 是哪个
2. 目标子网是哪个
3. 安全组是否已有，还是要新建
4. 是否需要公网 IP

如果这些问题没有答案，说明当前还不适合进入真实创建。

## 推荐交付

优先交付“前置检查结论”，而不是急着给最终创建命令。

例如：

- 已确认 VPC / 子网 / 安全组
- 还缺镜像或密钥对
- 还缺公网接入策略

## 避免事项

- 不要在 VPC、子网、安全组都没确认时直接构造 ECS 创建命令
- 不要把网络问题和规格问题混在一起一次性乱修
