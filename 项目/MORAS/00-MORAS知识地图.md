---
title: MORAS 知识地图
aliases:
  - MORAS 项目总览
tags:
  - project/moras
  - map-of-content
  - ai-agent
created: 2026-09-17
updated: 2026-09-17
status: active
---

# MORAS 知识地图

> [!summary]
> MORAS 是一套面向商品理解、选品、Multi-Agent 协作与视频生产的服务体系。当前最值得掌握的主链路是：入口服务接收需求，`moras-brain` 负责推理和调度，`moras-agent` 保存配置、会话、任务、产物和学习数据，`moras-video` / `moras-media` 执行媒体任务，`moras-schema` 统一管理数据库结构，`moras-composer` 管理运行环境与部署声明。

## 阅读路径

1. [[01-MORAS系统全景与调用链]]：先建立服务边界和端到端调用链。
2. [[02-MORAS仓库职责与技术栈]]：按仓库快速定位代码归属。
3. [[03-moras-brain的Multi-Agent与视频生产]]：理解智能调度发生在哪里。
4. [[04-moras-agent产品能力与架构]]：理解本轮重点学习项目。
5. [[05-moras-composer与moras-finder专题]]：理解部署配置与选品数据来源。
6. [[06-本机开发环境与运行手册]]：在新电脑上复现已经跑通的环境。
7. [[07-moras-agent安全开发全生命周期]]：完成一次不影响 `main` 和线上的开发练习。
8. [[08-权限边界与线上只读排障]]：明确当前可以做什么、不能做什么。
9. [[09-周边项目与待确认事项]]：记录 `lark-agent`、`moras-forge`、`moras-moneyking` 及信息缺口。

## 当前结论

- `moras-brain` 是 Multi-Agent 运行时，负责模型调用、Agent Profile 加载、工具调用和任务委派。
- `moras-agent` 已经不只是一个 CRUD 数据服务，更接近 AI 运行控制面和生产数据中枢。
- `moras-composer` 不是业务服务，它管理 Docker Compose、Helm、Argo CD、监控和环境配置注入。
- `moras-finder` 负责 TikTok Shop 商品采集、趋势评分、跨库同步和搜索索引，是选品数据上游。
- `moras-video` 负责异步图片、视频、TTS、ASR、工作流和 FFmpeg 任务。
- `moras-media` 负责视频、音频、字幕、抠图、水印和模型后处理流程。
- `moras-schema` 是数据库 DDL 的统一事实来源。

## 本轮会话已完成事项

- 登录公司私有 Gitea，并将 MORAS 相关仓库拉到本机。
- 梳理了 `moras-brain`、`moras-agent`、`moras-api` 及其关键依赖。
- 深入查看了 `moras-finder`、`moras-composer`、`moras-forge`、`moras-moneyking` 和 `lark-agent`。
- 选择 `moras-agent` 作为第一个全生命周期学习项目。
- 在隔离 Git worktree 和独立分支中准备开发环境。
- 安装 Go、MySQL、Python 等工具，使用 `moras-schema` 初始化本地库表。
- 成功编译、测试并运行 `moras-agent`，验证健康检查、鉴权和数据库访问。
- 登录 Argo CD；当前约定仅进行线上查询和只读排障，不操作发布。
- 完成飞书 CLI / `lark-agent` 的授权与消息链路验证；当前不把它纳入 CI/CD。

## 本地关键目录

| 内容 | 路径 |
|---|---|
| MORAS 仓库 | `/Users/zhao/Documents/Workspace/` |
| `moras-agent` 学习 worktree | `/Users/zhao/Documents/Workspace/.worktrees/moras-agent-version-endpoint` |
| `moras-schema` 学习 worktree | `/Users/zhao/Documents/Workspace/.worktrees/moras-schema-learning` |
| 本地隔离 Python | `/Users/zhao/Documents/Workspace/.local-moras-learning/` |
| MySQL LaunchAgent | `/Users/zhao/Library/LaunchAgents/ai.moras.learning-mysql.plist` |
| MySQL 数据目录 | `/Users/zhao/Library/Application Support/MorasLearning/mysql` |

## 信息安全约定

- 笔记不保存数据库密码、OAuth Token、API Key 或内部访问密钥值。
- `.env` 只记录用途和位置，不复制内容。
- 线上查询保持只读；任何部署、同步、回滚或数据写入都需要单独确认范围。
