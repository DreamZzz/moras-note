---
title: moras-agent 产品能力与架构
tags:
  - project/moras
  - moras-agent
  - architecture
  - control-plane
created: 2026-09-17
updated: 2026-09-17
---

# moras-agent 产品能力与架构

## 一句话定位

`moras-agent` 是 MORAS AI 业务的运行档案库、配置控制面和任务状态中心。它不负责主要的 LLM 推理，却保证 AI 生产过程可持续、可恢复、可治理、可追踪和可学习。

- 默认端口：`6092`，本地学习环境使用 `16092`。
- 数据库：拥有 `ai_agents` 和 `ai_journal`。
- 访问方式：内部 REST API。
- 通用鉴权：`X-Internal-Access-Key`；学习 Trace 还有独立能力密钥。
- 技术：Go、Gin、GORM、MySQL、Prometheus Metrics。

## 核心产品能力

### Agent 配置和发布

管理 Agent 注册信息、Profile Section、Learned Skill、Knowledge Block、Runtime Config 和 Brain Workflow。多个对象具有草稿、发布、下线、回滚和历史版本能力。

产品意义是把 Agent 从写死在代码里的角色，变成可配置、可审核和可逐步发布的线上能力。

### 会话、记忆和反馈

保存会话、父子会话、消息、工具调用、卡片、用户反馈和 Agent Memory。LUI 会话域进一步管理用户归属、幂等写入、历史召回、短期记忆和主动消息。

产品意义是提供连续的用户体验，并让反馈能够回流到质量和学习系统。

### 任务和 Workflow 运行账本

保存 Task、Event、Checkpoint、Heartbeat、Recovery、Durable Owner、Workflow Run、Stage Run 和 Pipeline Run。租约与条件更新用于避免多实例重复驱动同一阶段。

典型阶段：

```text
pa → video_plan → clip → composer
```

产品意义是让长时间、昂贵的 AI/视频流程可以恢复，避免重复执行和重复计费。

### 商品资料和 Agent 产物

管理 Product Profile、Revision、Media、Variant、Tag、Product Analysis Cache 和 Agent Output。产物保留会话、任务、Agent、Trace、尝试次数、缓存来源等关联。

产品意义是让多个 Agent 使用同一份可追溯的商品事实和结构化中间结果。

### 学习、质检和观测

保存 Trace Turn、Artifact、Output QA、人工 Task Review、模型价格和 Usage Stats。优秀经验可以形成 Learned Skill 或 Knowledge 新版本，再经过审核发布。

```text
线上执行 → Trace/产物 → 用户反馈/人工质检
        → 技能或知识候选 → 审核发布 → 改善后续执行
```

## 代码层次

```text
main.go
  ├── 加载环境配置
  ├── 建立 ai_agents / ai_journal 连接
  ├── 组装 DAO 与 Handler
  ├── 注册公共和内部路由
  └── 启动 Gin HTTP Server

api/
  ├── HTTP 参数、鉴权边界、响应码
  └── 调用 DAO 或领域服务

internal/dao/
  ├── GORM 查询与事务
  ├── 条件更新、版本、锁和幂等
  └── 数据库错误转换

internal/lui/session/
  └── LUI 会话领域状态机、归属、记忆和上下文策略

models/
  └── 数据库模型与 API 数据结构
```

## 最重要的架构判断

`moras-agent` 已经从早期 README 描述的“数据微服务”发展成 AI 运行控制面。它的核心价值不是 CRUD 数量，而是以下业务约束：

- 会话和消息的归属是否正确；
- 请求重试是否幂等；
- Workflow 是否固定在正确版本；
- 任务是否被两个执行器同时驱动；
- 哪个产物属于哪次尝试；
- 技能和知识是否经过审核后才影响生产；
- 服务中断后是否能从持久状态恢复。

## 当前本地验证

- `go build ./...` 通过。
- `go test ./... -count=1` 通过。
- MySQL 中 `ai_agents` 有 49 张表，`ai_journal` 有 4 张表。
- `/health`、内部鉴权、Agent 列表和 Metrics 已验证。
- 服务验证后已主动停止；MySQL 保持为 LaunchAgent 运行。

## 边界提醒

- 不直接面向终端用户暴露，通常由 Edge/API 代理并补充用户身份。
- 修改表结构时先改 `moras-schema`，不能只在 GORM Model 中增加字段。
- 涉及外部 OSS、TikTok 或视频系统的需求，还需要对应测试凭据和服务。
- `ai_video` 不属于 `moras-agent` 核心库，本机暂未初始化。
