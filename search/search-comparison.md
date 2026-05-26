# 联网搜索方案对比

> 内容来源：小红书经验贴综合整理

## 方案总览

| 方案 | 推荐场景 | 优点 | 缺点 | 费用 |
|------|---------|------|------|------|
| **Tavily** | 日常搜索 | OpenClaw 自带集成，直接配 apiKey 即可 | 有免费额度限制 | 免费额度+ |
| **Serper** | 白嫖首选 | 2500 次免费，不用绑卡，外文搜索强，覆盖广 | 用完需换邮箱重注 | 免费 2500次 |
| **Perplexity** | 体验最好 | 又快又稳，AI 搜索代表 | 需付费 Pro | 付费 |
| **Bird** | X/Twitter 定向搜索 | OpenClaw 自带，专门搜 X | 需要配 cookies | 免费 |
| **Minimax** | 中文/B站搜索 | 能覆盖 B 站，中文场景不错 | 速度慢，search tool 未集成到模型 | 模型付费 |

## 各方案详解

### Tavily（当前在用）

当前 OpenClaw 默认使用的搜索引擎，已在配置中集成：
```json
{
  "tools": { "web": { "search": { "provider": "tavily", "enabled": true } } }
}
```

### Serper（Google 搜索）

**配置方式：** 让 agent 帮忙搭一个 Serper 搜索 skill
```
帮我搭一个 serper dev 的搜索 skill，需要 api key 搭时候告诉我
```
- 注册：https://serper.dev
- 免费 2500 次额度，无需绑卡
- 用完后换邮箱注册继续白嫖
- 外文搜索又新又全
- 连 X/Twitter 内容都能搜

### Perplexity

- AI 搜索的代表
- 如果有 Lenny Bundle Pro 赠送的额度可以用
- 又快又稳，体验最好
- 钱包 ok 就选它

### Bird（X 搜索）

- OpenClaw 自带的搜索工具
- 专门用来搜索 X/Twitter 内容
- 需要按指南配置 cookies
- X 定向资讯搜索很稳

### Minimax

- Minimax 模型本身有 search tool 功能
- 但未集成到模型里，需要单独封装成一个 skill
- 中文搜索效果好
- 可覆盖 B 站内容
- 最大缺点：速度慢

## 最佳实践

1. 钱包 ok → **Perplexity**，又快又稳
2. 白嫖党 → **Serper**，免费额度大，覆盖面广，X 都能搜
3. X 定向搜索 → **Bird**，稳
4. 中文搜索 → **Minimax**，可覆盖 B 站，但慢
5. 把这些策略告诉 agent，让它记住并按需调度不同的搜索工具
