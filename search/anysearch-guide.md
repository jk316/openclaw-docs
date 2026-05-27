# 🔍 AnySearch — 通用联网搜索 Skill

> 适用场景：网页搜索、垂直领域搜索（23个领域）、批量并行搜索、URL 内容提取
> 状态：✅ 已安装到本机 OpenClaw

---

## 概述

AnySearch 是一个统一的实时搜索服务，支持：
- **通用网页搜索** — 类似 Google/Brave 的普通搜索
- **垂直领域搜索** — 23个专业领域（股票、金融、学术、代码、专利等），按领域有特定 query 格式
- **批量并行搜索** — 一次同时搜 2-5 个独立查询
- **URL 内容提取** — 抓取网页正文（转 Markdown，上限 5 万字）

底层协议：JSON-RPC 2.0，无需 MCP 服务端安装。

## 安装

### 方法 1：ClawHub 下载（推荐）

访问 https://clawhub.ai/plugins/anysearch → 点 Download 按钮 → 解压到 `~/.openclaw/workspace/skills/anysearch/`

### 方法 2：openclaw onboard

```
openclaw onboard
```
按引导选择安装 anysearch。

## 安装后验证

```bash
cd ~/.openclaw/workspace/skills/anysearch/
python3 scripts/anysearch_cli.py doc   # 查看完整接口文档
python3 scripts/anysearch_cli.py search "测试查询" --max_results 3
```

## 使用方式

### 1. 通用搜索

```bash
python3 scripts/anysearch_cli.py search "<查询词>" [选项]
```

选项：
| 参数 | 说明 | 默认 |
|------|------|------|
| `--max_results, -m` | 结果数 1-100 | 10 |
| `--content_types, -t` | 内容类型: web/news/code/doc/academic/data/image/video/audio | web |
| `--freshness, -f` | 时效: day/week/month/year | — |
| `--zone, -z` | 区域: cn / intl | 不限制 |

示例：
```bash
# 通用查询
python3 scripts/anysearch_cli.py search "OpenClaw agent开发教程" -m 5

# 最近一周新闻
python3 scripts/anysearch_cli.py search "AI芯片" -f week -t news -m 5

# 中文内容
python3 scripts/anysearch_cli.py search "langchain教程" -z cn -m 5
```

### 2. 垂直领域搜索

先查可用领域：

```bash
python3 scripts/anysearch_cli.py list_domains --domain finance
```

再用领域特定的格式搜：

```bash
# 查美股实时行情
python3 scripts/anysearch_cli.py search "TSLA" --domain finance --sub_domain finance.us_stock

# 查 A 股
python3 scripts/anysearch_cli.py search "600519" --domain finance --sub_domain finance.cn_stock --zone cn

# 查学术论文
python3 scripts/anysearch_cli.py list_domains --domain academic
python3 scripts/anysearch_cli.py search "transformer attention" --domain academic --sub_domain academic.paper
```

### 3. 批量搜索

```bash
python3 scripts/anysearch_cli.py batch_search --query "ARK Invest" --query "Cathie Wood TSLA" -m 3
```

### 4. URL 内容提取

```bash
python3 scripts/anysearch_cli.py extract "https://example.com/article"
```

## 可用领域一览

通过 `list_domains` 命令动态查询，目前支持 23 个领域：

| 领域 | 领域代码 | 示例子领域 |
|------|---------|-----------|
| 科技 | tech | tech.news, tech.product |
| 时尚 | fashion | fashion.trend, fashion.brand |
| 旅游 | travel | travel.destination, travel.flight |
| 电商 | ecommerce | ecommerce.product |
| 游戏 | gaming | gaming.news, gaming.review |
| 电影 | film | film.review, film.box_office |
| 音乐 | music | music.artist, music.album |
| 金融 | finance | **finance.us_stock**, finance.cn_stock, finance.forex, finance.news |
| 学术 | academic | academic.paper, academic.author |
| 法律 | legal | legal.case, legal.statute |
| 商业 | business | business.company, business.news |
| 知识产权 | ip | **ip.patent**（按专利号查） |
| 安全 | security | security.cve |
| 教育 | education | education.course |
| 健康 | health | health.disease, health.drug |
| 宗教 | religion | — |
| 地理 | geo | geo.place |
| 环境 | environment | — |
| 能源 | energy | — |
| 通用 UGC | ugc | ugc.reddit, ugc.quora, ugc.zhihu |

## API Key 配置

AnySearch 支持匿名使用（有限额），也可以配置 API Key 获得更高额度。

```bash
# 方式 1：在 skill 目录下创建 .env
echo "ANYSEARCH_API_KEY=your_key" > ~/.openclaw/workspace/skills/anysearch/.env

# 方式 2：环境变量
export ANYSEARCH_API_KEY=your_key
```

申请地址：https://anysearch.com/console/api-keys

## 常见问题

### Q: 和 Tavily 有什么区别？
Tavily 是 OpenClaw 内置的默认搜索工具，有免费额度但限制较多。AnySearch 的优势在于：
- 23 个垂直领域搜索
- 批量并行搜索
- URL 全文提取
- 匿名可用

### Q: 适合什么场景？
- 需要美股/A 股实时行情 → 垂直搜索 `finance.us_stock`
- 需要搜专利/论文 → 垂直搜索 `ip.patent` / `academic.paper`
- 需要批量查多个不相关的内容 → `batch_search`
- 需要抓取网页正文 → `extract`

### Q: 速度如何？
通常 1-3 秒返回结果，垂直搜索稍快。

---

> 最后更新: 2026-05-28
> 当前版本: 1.0.3（SKILL.md 标注 v2.0.0）
