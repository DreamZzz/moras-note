---
title: moras-agent 安全开发全生命周期
tags:
  - project/moras
  - moras-agent
  - development-workflow
  - git
created: 2026-09-17
updated: 2026-09-17
---

# moras-agent 安全开发全生命周期

## 目标

在不修改原仓库 `main`、不影响线上服务的情况下，完成一次需求的代码、数据库、测试、提交、PR 和只读上线观察全过程。

## 1. 更新代码基线

在开始新需求前确认远端 `main`：

```bash
cd /Users/zhao/Documents/Workspace/.worktrees/moras-agent-version-endpoint
git status --short --branch
git fetch origin main
git rebase origin/main
```

必须满足：

- 工作区没有未知改动；
- 当前分支不是 `main`；
- Rebase 没有未解决冲突。

## 2. 需求归属判断

编码前先确认改动属于哪个层次：

| 变化 | 仓库/层次 |
|---|---|
| HTTP 参数、状态码、鉴权 | `moras-agent/api` |
| 数据库查询、事务、锁、CAS | `moras-agent/internal/dao` |
| LUI 会话状态规则 | `moras-agent/internal/lui/session` |
| 数据模型映射 | `moras-agent/models` |
| 数据库表/字段/索引 | `moras-schema` |
| Agent 推理、Prompt 或工具决策 | `moras-brain` |

若新增数据库字段，顺序应是：

```text
moras-schema migration
    ↓
本地 dry-run / apply
    ↓
moras-agent model + DAO + API
    ↓
集成验证
```

## 3. 最小实现与验证

推荐顺序：

1. 先阅读现有 Handler、DAO、Model 和相邻测试。
2. 实现最小业务闭环。
3. 对重要状态转换、鉴权、幂等或事务补充测试。
4. 执行格式化与测试。
5. 启动本地服务做 HTTP 验证。

```bash
gofmt -w <changed-go-files>
go test ./... -count=1
go build ./...
```

对数据库需求，还要验证：

- 空库/最新 Migration 是否能运行；
- 重复请求是否产生重复行；
- 状态冲突是否返回明确错误；
- 事务失败是否整体回滚；
- 查询是否有合适索引。

## 4. 本地运行

参考 [[06-本机开发环境与运行手册]] 启动服务。验证至少包括：

- `/health` 正常；
- 缺少内部 Key 时返回未授权；
- 合法 Key 下新接口成功；
- 错误参数、资源不存在和状态冲突符合契约；
- 数据库结果与 API 响应一致。

## 5. 提交与 PR

```bash
git status
git diff --check
git diff
git add <明确的文件>
git commit -m "feat: ..."
git push -u origin zhao/<feature-name>
```

PR 描述应包含：

- 具体问题和触发条件；
- 修改后的用户/调用方行为；
- 数据库或接口兼容性；
- 已运行的测试；
- 发布、回滚和观测方式。

学习阶段优先创建 Draft PR，不合并 `main`，不修改部署环境。

## 6. CI 与发布边界

- 可以查看 CI 日志和测试结果。
- 当前不把飞书链路接入 CI/CD。
- 当前 Argo CD 用于线上状态查询和只读排障。
- 不执行 Sync、Refresh with mutation、Rollback、Restart、Scale 或 Secret 修改。
- 真正上线前需要团队正常的 Review、Merge、镜像构建和 Argo CD 流程。

## 7. 上线后只读观察

即使没有部署权限，也可以完成有价值的验证：

- 查看 Argo CD Application 的 Sync/Health 状态；
- 查看镜像 Tag、Commit 与期望版本是否一致；
- 查看 Pod 状态、重启次数、健康检查；
- 查看 Metrics、日志和错误率；
- 将现象与本地 Trace、任务 ID 和接口契约关联。

## 完成标准

一次需求只有满足以下条件才算形成完整学习闭环：

- 需求和服务边界明确；
- 代码和 Schema 变化一致；
- 单元测试与必要的数据库测试通过；
- 本地 HTTP 行为验证；
- 分支和 PR 不污染 `main`；
- CI 结果可解释；
- 发布影响、回滚方法和观测指标明确。
