# 🦐 OpenClaw 使用经验沉淀

> 整理日期：2026-05-26

## 一、基础命令

| 命令 | 作用 |
|------|------|
| `/stop` | 停止当前正在执行的任务 |
| `/status` | 查看当前会话状态、模型、上下文使用量 |
| `/compact` | 压缩上下文，减少 token 消耗（长会话必用） |
| `/new` | 开启全新会话（当上下文太乱或卡死时使用） |

## 二、模型选择

OpenClaw 可通过配置切换底层模型：

- **DeepSeek Chat**（当前默认）：性价比高，速度快
- 可在 `openclaw.json` 或通过 `/model` 命令切换

## 三、技能系统

OpenClaw 通过 Skills 扩展能力，技能目录通常位于 `~/.agents/skills/` 或 `~/.openclaw/lib/node_modules/openclaw/skills/`。

### 常用技能

- **股票分析**（stock-analysis）：基于 yfinance 的股票/加密货币分析
- **小红书**（xiaohongshu）：小红书内容搜索、爬取、热点追踪
- **飞书文档**（feishu-doc）：飞书文档读写操作
- **GitHub**（github）：GitHub issues、PR、CI 操作
- **浏览器**（agent-browser）：Headless 浏览器自动化
- **iMessage**（imsg）：macOS 短信/iMessage 操作
- **天气**（weather）：天气查询
- **Obsidian**（obsidian）：Obsidian 笔记操作
- **任务流**（taskflow）：多步骤长任务编排

### 技能发现

使用 `find-skills` 技能可以在 [clawhub.ai](https://clawhub.ai) 上搜索可用技能。

## 四、文件系统

- **工作目录**：`~/.openclaw/workspace/` — 所有文件操作都在此目录
- **记忆系统**：
  - `MEMORY.md` — 长期记忆，跨会话持久化
  - `memory/YYYY-MM-DD.md` — 每日日志
- **配置文件**：`~/.openclaw/openclaw.json` — 主配置
- **技能文件**：`SKILL.md` — 每个技能的说明文档

## 五、飞书集成

飞书作为主要消息通道，支持：

### 文档操作
- 通过飞书开放 API 直接操作（RESTful）
- 使用应用 token 认证（`tenant_access_token`）
- 支持创建文档、写入表格、删除表格等操作
- 表格 API 有限制：一次性创建大尺寸表格可能失败

### 权限说明
- `docx:document` 权限：可创建/读写文档
- `drive:drive` 权限：可上传文件到云盘（需额外申请）

### 注意事项
- curl 比 Python requests 在飞书 API 调用中更稳定（避免 JSON 编码问题）
- 文档操作失败后需重新创建文档（旧文档结构可能损坏）
- Excel 区域操作 API 有行数限制

## 六、股票数据获取

### 数据源
- **yfinance**：免费无限制，支持全部历史日线数据（1980至今）
- **Finnhub**：免费 API 仅支持 `/quote`（实时报价）和基本信息，`/candle`（K线）需要付费套餐

### 实际案例
- 获取 NASDAQ 100 全部 101 只股票的历史数据（749,421 条记录）
  - 使用 yfinance，耗时约 4 分钟
  - 输出格式：每只股票一个 JSON 文件 + 汇总 CSV + 摘要 JSON
  - 总大小约 156MB（JSON）+ 50MB（CSV）
- 通过 163 邮箱 SMTP 发送附件到 QQ 邮箱
  - 需要 163 邮箱的授权码（非登录密码）
  - 授权码可能过期，需定期更新

## 七、常用技巧

### 长会话管理
1. 定期使用 `/compact` 压缩上下文
2. 任务完成后使用 `/new` 开启新会话
3. 重要信息写入 `memory/` 目录下的日志文件

### 数据存储
1. 重要数据存本地文件（JSON/CSV）
2. 可通过邮件发送分享
3. 飞书文档适合展示摘要，大量数据需用附件

### 故障恢复
1. 任务卡死 → `/stop` 停止
2. 上下文太乱 → `/compact` 压缩或 `/new` 开新会话
3. 飞书 API 报错 → 检查 token 是否过期，重新获取

## 八、联网搜索方案对比

OpenClaw 支持多种联网搜索工具，实测对比总结如下：

| 方案 | 推荐场景 | 优点 | 缺点 | 费用 |
|------|---------|------|------|------|
| **Tavily**（当前在用） | 日常搜索 | 自带在 OpenClaw 中，可直接配置 apiKey | 有免费额度限制 | 免费额度+
| **Serper** | 白嫖首选 | 2500 次免费，不用绑卡，外文搜索强，覆盖广（含 X） | 用完需换邮箱重注 | 免费 2500次
| **Perplexity** | 体验最好 | 又快又稳，AI 搜索代表 | 需付费 Pro | 付费
| **Bird** | X/Twitter 定向搜索 | OpenClaw 自带，专门搜 X | 需要配 cookies | 免费
| **Minimax** | 中文/B站搜索 | 能覆盖 B 站，中文场景不错 | 速度慢，search tool 未集成到模型 | 模型付费

### 配置方式

**Serper 配置示例：**
```
口喷让 agent 帮忙搭一个 Serper 的搜索 skill，需要 api key 搭的时候告诉你
```
- 注册地址：https://serper.dev（免费 2500 次，无需绑卡）
- 用完后换邮箱注册可继续白嫖

**Bird（X 搜索）配置：**
- OpenClaw 自带，按指南配置 cookies 即可使用
- 适合 X/Twitter 上的定向资讯追踪

**Minimax 配置：**
- Minimax 本身有 search tool 功能但未集成到模型中
- 需要单独封装成一个 skill 使用
- 中文搜索效果好，能覆盖 B 站

### 经验总结
1. 钱包 ok → **Perplexity**，又快又稳
2. 白嫖党 → **Serper**，免费额度大，覆盖面广
3. X 定向搜索 → **Bird**，稳
4. 中文搜索 → **Minimax**，可覆盖 B 站，但慢
5. 最佳实践：把这些策略告诉 agent，让它按需调度不同的搜索工具

## 九、小红书平台技巧

- 小红书链接可通过 xhslink.com 短链接分享
- 通过 web_fetch 可读取小红书笔记内容
- 小红书搜索使用专用 `xiaohongshu` 技能

## 十、OpenClaw Control Center（可视化控制面板）

- **项目地址：** https://github.com/TianyiDataScience/openclaw-control-center
- **本地安装位置：** `/Users/zxkkk/.openclaw/workspace/openclaw-control-center/`
- **功能：** 将 OpenClaw 从黑盒变成可视化、可控的控制中心
- **启动命令：** `cd ~/.openclaw/workspace/openclaw-control-center && npm run dev:ui`
- **访问地址：** http://127.0.0.1:4310
- **主要页面：** Overview（健康概览）、Usage（用量/花费）、Staff（Agent状态）、Collaboration（协作大厅）、Tasks（任务与审批）、Memory/Documents（记忆和文档编辑）、Settings（安全/连接）
- **默认配置：** 只读模式、本地 Token 认证、时区 Asia/Shanghai

## 十一、编程能力

- 可直接在本地执行 Python/Shell 脚本
- 支持 pip 安装第三方库（如 yfinance）
- 邮件发送通过 smtplib 实现
- 数据文件可本地处理和转换格式
