# Huawei CLI Skill Source Map

本文件说明 `huaweicloud-skill` 在构建和维护时，应该如何使用 `materials/` 下的原始资料。

## 资料分层

默认使用顺序：

1. `references/`
2. `materials/hcloud-docs-md/`
3. `materials/hcloud-docs-json/`
4. `materials/hcloud-docs-pdf/`

解释：

- `references/`
  - 是清洗后的技能资料。
  - 只保留当前 skill 真的需要的规则、流程、例子和约束。
- `materials/hcloud-docs-md/`
  - 是主要阅读源。
  - 适合 `rg`、摘取命令示例、整理章节内容。
- `materials/hcloud-docs-json/`
  - 是脚本抽取源。
  - 适合按页定位、过滤目录页、提取高分页面内容。
- `materials/hcloud-docs-pdf/`
  - 是权威原件。
  - 当 `md` 或 `json` 的内容明显损坏、断行或歧义时，用它回查。

## 当前原始文档用途

### 用户指南

- 主用途：
  - 配置项
  - 选项说明
  - `--cli-jsonInput`
  - `--cli-output`
  - `--cli-query`
  - `--cli-waiter`
- 主要来源：
  - `materials/hcloud-docs-md/华为云命令行工具服务 KooCLI 用户指南_md_dollar/output.md`
  - `materials/hcloud-docs-json/华为云命令行工具服务 KooCLI 用户指南.json`

### 常见问题

- 主用途：
  - 认证优先级
  - 缓存位置
  - 日志位置
  - 网络超时
  - 不支持的服务或 operation
  - 空响应体判断
  - 区域参数问题
- 主要来源：
  - `materials/hcloud-docs-md/华为云命令行工具服务 KooCLI 常见问题_md_dollar/output.md`
  - `materials/hcloud-docs-json/华为云命令行工具服务 KooCLI 常见问题.json`

### 快速入门

- 主用途：
  - 安装方式
  - 初始化配置
  - 新用户的最短上手路径
- 主要来源：
  - `materials/hcloud-docs-md/华为云命令行工具服务 KooCLI 快速入门_md_dollar/output.md`

## 原始资料的已知问题

- 目录页噪声较多
- 页码残留
- 命令换行被打断
- `说明` / `注意` 等块可能被转成异常字符
- 图片类示例会变成图片占位，而不是文本

因此：

- skill 运行时优先看 `references/`
- 只有在 `references/` 没覆盖时，才回到 `materials/`
- 回到 `materials/` 时，优先用 `md`，必要时配合 `json` 和 `pdf`
