---
title: MORAS 系统全景与调用链
tags:
  - project/moras
  - architecture
  - call-chain
created: 2026-09-17
updated: 2026-09-17
---

# MORAS 系统全景与调用链

## 业务分层

```mermaid
flowchart TD
    U[用户 / 运营人员] --> API[moras-api / MORAS App API]
    U --> EDGE[moras-edge / 内部网关]
    EDGE --> BRAIN[moras-brain]
    API --> BRAIN

    BRAIN <--> AGENT[moras-agent]
    BRAIN --> PRODUCT[moras-product]
    BRAIN --> VIDEO[moras-video]

    API --> MEDIA[moras-media]
    PRODUCT <--> FINDER[moras-finder 数据结果]
    FINDER --> MEILI[Meilisearch]

    VIDEO --> PROVIDER[外部图片/视频/TTS/ASR 提供商]
    MEDIA --> ML[本地/外部媒体模型与 FFmpeg]

    SCHEMA[moras-schema] -.统一 DDL.-> AGENT
    SCHEMA -.统一 DDL.-> VIDEO
    SCHEMA -.统一 DDL.-> FINDER

    COMPOSER[moras-composer] -.部署、配置、监控.-> EDGE
    COMPOSER -.部署、配置、监控.-> BRAIN
    COMPOSER -.部署、配置、监控.-> AGENT
    COMPOSER -.部署、配置、监控.-> PRODUCT
    COMPOSER -.部署、配置、监控.-> VIDEO
    COMPOSER -.部署、配置、监控.-> MEDIA
```

## 各层的产品职责

### 入口与身份层

- `moras-api`：面向产品用户的 Node.js 后端，承担 JWT、用户、文件、任务以及图片/视频能力的统一产品 API。
- `moras-edge`：内部平台网关，处理 Keeper OAuth、角色校验、CORS 和服务反向代理；它也会根据 `moras-agent` 的 Agent 注册表决定未知 Agent slug 是否允许进入 Brain Pool。

### 智能决策层

- `moras-brain`：加载 Agent Profile，选择模型，组织上下文，调用工具，并在 Multi-Agent 模式下委派子任务。
- 它是“思考和调度”的主要位置，不应把持久化细节散落回 Brain 内部。

### 运行状态与治理层

- `moras-agent`：拥有 `ai_agents` 和 `ai_journal`，保存会话、消息、任务、工作流运行、Agent 产物、Trace、技能、知识和运行配置。
- 它解决的是一致性、审计、恢复、版本和生命周期管理问题。

### 商品与选品层

- `moras-finder`：从 FastMoss 等来源采集商品，进行趋势和内容评分，同步到业务库与 Meilisearch。
- `moras-product`：提供商品检索、推荐及内部商品 API，消费 Finder 整理后的商品数据和搜索索引。

### 媒体执行层

- `moras-video`：异步图片、视频、TTS、ASR、工作流和 FFmpeg Job 的服务端执行器，使用 RabbitMQ 和数据库做持久化队列与恢复。
- `moras-media`：视频抠像、商品图抠图、音色转换、字幕、水印、相似度等媒体模型和后处理管线。

### 平台工程层

- `moras-schema`：所有数据库 Schema 和 Migration 的统一管理入口。
- `moras-composer`：Docker Compose、Helm、Argo CD Application、Secret、监控和告警配置的基础设施仓库。

## 以视频生产为例的主链路

```mermaid
sequenceDiagram
    actor User as 用户
    participant API as moras-api/edge
    participant Brain as moras-brain
    participant Agent as moras-agent
    participant Product as moras-product
    participant Video as moras-video
    participant Media as moras-media

    User->>API: 提交商品和视频需求
    API->>Brain: 创建一次 Agent 请求
    Brain->>Agent: 创建会话、任务和 Workflow Run
    Brain->>Product: 获取商品资料/推荐数据
    Product-->>Brain: 商品信息
    Brain->>Agent: 读取 Product Profile/分析缓存/技能/知识
    Brain->>Brain: 商品分析 → 视频策划 → 素材规划
    Brain->>Video: 提交图片/视频/TTS/ASR 任务
    Video-->>Brain: 异步任务状态与产物地址
    Brain->>Media: 必要时执行抠图、水印、音频或后处理
    Brain->>Agent: 保存阶段产物、Trace、检查点和最终结果
    Brain-->>API: 返回状态或最终视频
    API-->>User: 展示结果
```

## 关键边界

1. **推理与持久化分离**：Brain 负责决策，Agent 负责状态和事实。
2. **媒体生成与业务编排分离**：Video/Media 执行昂贵、异步的媒体工作，Brain 决定何时调用。
3. **DDL 与运行代码分离**：Schema 变更先进入 `moras-schema`，运行服务只消费结构。
4. **外部入口与内部服务分离**：`moras-agent` 不应直接暴露给终端用户，通常经 Edge/API 进入。
5. **环境声明与业务代码分离**：镜像版本、环境变量和 Secret 引用由 Composer/Helm/Argo CD 管理。

## 进一步阅读

- [[02-MORAS仓库职责与技术栈]]
- [[03-moras-brain的Multi-Agent与视频生产]]
- [[04-moras-agent产品能力与架构]]
