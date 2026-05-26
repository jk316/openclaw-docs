# OpenClaw Control Center（可视化控制面板）

> 将 OpenClaw 从黑盒变成可见、可控、可理解的本地控制中心

## 基本信息

- **项目地址：** https://github.com/TianyiDataScience/openclaw-control-center
- **Star 数：** ~4k
- **语言：** TypeScript
- **本地位置：** `~/workspace/openclaw-control-center/`

## 安装步骤

```bash
git clone https://github.com/TianyiDataScience/openclaw-control-center.git
cd openclaw-control-center
npm install
cp .env.example .env
# 修改 .env 配置：时区、Local API Token
npm run dev:ui
```

## 启动方式

```bash
cd ~/workspace/openclaw-control-center
npm run dev:ui
```

访问 http://127.0.0.1:4310

## 主要功能

| 页面 | 用途 |
|------|------|
| **Overview** | 系统健康概览、关键告警、活跃 Agent |
| **Usage** | Token 用量、花费趋势、上下文压力 |
| **Staff** | Agent 实时状态 — 谁在工作、谁在排队 |
| **Collaboration** | 多 Agent 协作大厅，共享任务讨论、分派、交接、审查 |
| **Tasks** | 任务看板、审批流、执行链 |
| **Memory** | 记忆文件编辑和查看 |
| **Documents** | Markdown 文档编辑 |
| **Settings** | 连接状态、安全检查、更新状态 |

## 安全配置

- 默认只读模式（`READONLY_MODE=true`）
- 本地 Token 认证保护写操作（`LOCAL_TOKEN_AUTH_REQUIRED=true`）
- 审批功能默认关闭（`APPROVAL_ACTIONS_ENABLED=false`）
