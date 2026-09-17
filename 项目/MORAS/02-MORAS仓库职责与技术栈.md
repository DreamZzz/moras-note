---
title: MORAS 仓库职责与技术栈
tags:
  - project/moras
  - repositories
  - tech-stack
created: 2026-09-17
updated: 2026-09-17
---

# MORAS 仓库职责与技术栈

## 本地仓库速查

| 仓库 | 核心职责 | 主要技术 | 典型依赖/下游 |
|---|---|---|---|
| `moras-brain` | Multi-Agent 推理、Profile/Skill 加载、工具调用、任务委派 | Python、FastAPI、多模型 Provider | `moras-agent`、`moras-product`、`moras-video` |
| `moras-agent` | Agent 运行数据、会话、任务、产物、Trace、技能、知识、Workflow | Go、Gin、GORM、MySQL | 被 Brain、Edge、Console 等调用 |
| `moras-api` | 面向用户的 AI 视频后端、认证、文件和任务 API | TypeScript、Node.js、Express、MySQL、OSS、FFmpeg | 媒体 Provider、`moras-media` 等 |
| `moras-edge` | OAuth、角色控制、内部平台网关和反向代理 | Go | Keeper、Brain、Agent、Product、Video |
| `moras-product` | 商品推荐、检索与内部商品 API | Go、MySQL、Meilisearch | Finder 数据、Video 已售验证等 |
| `moras-video` | 图片/视频/TTS/ASR/FFmpeg 异步任务 | Go、Gin、GORM、RabbitMQ、MySQL、FFmpeg | 外部生成 Provider、`ai_video` |
| `moras-media` | 抠图、抠像、字幕、音频、水印和媒体模型后处理 | Python、Flask、Celery、RabbitMQ、FFmpeg、GPU 模型 | `moras-api` 等业务服务 |
| `moras-finder` | TikTok Shop 商品采集、评分、ETL 和搜索同步 | Python、Flask、Playwright、MySQL、Meilisearch | FastMoss、`moras-product` |
| `moras-schema` | 集中管理 MySQL/PostgreSQL DDL、Migration 和 Seed | Python、SQL | 所有持久化服务 |
| `moras-composer` | 本地/测试/生产编排、Helm、Argo CD、Secret、监控告警 | Docker Compose、Kubernetes、Helm、Argo CD、SOPS | 全平台服务 |
| `moras-moneyking` | 本地目录存在，但当前检出不完整，不能仅凭本地代码确认职责 | 待重新拉取 | Composer 中存在对应 Helm/Argo CD 声明 |

## 技术选型判断

### 合理的部分

- **Go 用于 Agent、Edge、Product、Video**：这些服务以 API、状态机、数据库和并发任务为主，Go 的低资源占用、静态编译和并发模型适合这种场景。
- **Python 用于 Brain、Finder、Media**：模型 SDK、浏览器采集、数据处理和媒体算法生态主要集中在 Python，开发效率更高。
- **Node.js 用于产品 API**：`moras-api` 连接 Web 产品、JWT、文件上传和多种外部接口，TypeScript 有利于快速迭代和前后端契约协作。
- **MySQL 作为业务事实库**：会话、任务、版本、审核和生产状态都需要事务、一致性和索引查询，关系型数据库合适。
- **RabbitMQ 用于长耗时媒体任务**：生成和后处理无法依赖单个 HTTP 请求生命周期，持久化队列是合理选择。
- **Meilisearch 用于商品检索**：把全文与筛选检索从业务数据库中拆出，适合商品搜索和推荐候选集获取。

### 主要风险

- 多语言、多框架会提高本地联调和运维成本，需要稳定的接口契约与统一观测。
- `moras-agent` 的职责持续扩张，已经同时承担数据服务、运行账本、配置中心和学习治理；新增能力时应先判断是否仍属于 Agent 运行域。
- `moras-api`、`moras-video`、`moras-media` 都包含部分媒体能力，产品职责需要持续保持清晰，避免重复实现。
- 本地全栈启动依赖 Docker、RabbitMQ、Meilisearch、FFmpeg 和多个数据库；日常单仓开发应优先使用最小依赖集。

## 判断代码归属的简单规则

- “模型该怎么思考、调用什么工具” → `moras-brain`
- “这次会话/任务/产物是否存在、执行到哪一步” → `moras-agent`
- “商品从哪里来、怎样评分和同步” → `moras-finder`
- “怎样搜索或推荐商品” → `moras-product`
- “怎样提交和恢复视频生成任务” → `moras-video`
- “怎样对媒体做算法处理或后处理” → `moras-media`
- “数据库表如何变化” → `moras-schema`
- “服务如何获得配置并部署到环境” → `moras-composer`

## 本地位置

所有核心仓库位于 `/Users/zhao/Documents/Workspace/`。开始开发前不要直接在这些仓库的 `main` 上修改，应按 [[07-moras-agent安全开发全生命周期]] 创建隔离 worktree。
