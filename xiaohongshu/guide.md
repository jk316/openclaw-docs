# 🦐 小红书使用指南

本机已部署 xiaohongshu-mcp（v2.0）服务，通过 HTTP API 调用，无需每次重复扫码。
以下整理了所有常用操作及具体用法。

---

## 登录状态

> ✅ **已登录**（用户名: xiaohongshu-mcp）
> Cookie 有效期约 30 天，过期需重新扫码。

如失效，需重新扫码登录：

```bash
# 1. 启动服务
~/.local/bin/xiaohongshu-mcp -headless -port ":18060" &

# 2. 获取二维码（会生成到桌面）
# 二维码会输出 base64 图片数据
# 用小红书 App 扫码登录后，Cookies 自动保存
```

---

## MCP 服务管理

### 启动服务

```bash
~/.local/bin/xiaohongshu-mcp -headless -port ":18060" &
```

服务监听在 `http://localhost:18060/mcp`，共 13 个 MCP 工具。

### 检查服务是否运行

```bash
curl -s http://localhost:18060/mcp -X POST \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1.0"}}}'
```

### 停止服务

```bash
pkill -f xiaohongshu-mcp
```

---

## API 调用规范

每次调用需要完整的三步流程：

```
1. initialize  → 获取 Mcp-Session-Id（从 Response Header 中获取）
2. notifications/initialized → 通知服务端准备就绪
3. tools/call → 调用具体工具（带上 Session-Id）
```

---

## 可用工具清单（13个）

| 工具 | 用途 | 是否需要登录 |
|------|------|:----------:|
| `check_login_status` | 检查登录状态 | ❌ |
| `get_login_qrcode` | 获取登录二维码 | ❌ |
| `delete_cookies` | 删除 cookies | ❌ |
| `search_feeds` | 🔍 搜索笔记 | ✅ |
| `list_feeds` | 🏠 首页推荐 | ✅ |
| `get_feed_detail` | 📄 帖子详情+评论 | ✅ |
| `post_comment_to_feed` | 💬 发表评论 | ✅ |
| `reply_comment_in_feed` | ↩️ 回复评论 | ✅ |
| `like_feed` | ❤️ 点赞/取消 | ✅ |
| `favorite_feed` | ⭐ 收藏/取消 | ✅ |
| `user_profile` | 👤 用户主页+笔记 | ✅ |
| `publish_content` | 📝 发布图文笔记 | ✅ |
| `publish_with_video` | 🎬 发布视频笔记 | ✅ |

---

## 常用操作速查

### 1️⃣ 搜索笔记

```python
# 关键词：必填
# filters：可选筛选条件
{
    "keyword": "openclaw可视化",
    "filters": {
        "sort_by": "综合",        # 综合 | 最新 | 最多点赞 | 最多评论 | 最多收藏
        "note_type": "不限",      # 不限 | 视频 | 图文
        "publish_time": "不限",   # 不限 | 一天内 | 一周内 | 半年内
        "search_scope": "不限",   # 不限 | 已看过 | 未看过 | 已关注
        "location": "不限"        # 不限 | 同城 | 附近
    }
}
```

返回数据包含每个笔记的：
- `id` → feed_id
- `xsecToken` → xsec_token（**这两个很重要，后续操作都需要**）
- `noteCard` → 标题、作者、点赞/评论/收藏数、封面图

### 2️⃣ 获取帖子详情 + 评论

```python
{
    "feed_id": "69b7599b000000001d01c044",
    "xsec_token": "ABFHBtwcMdVZ4iKGopHCB2FhOEkCHKeZ2EfFjXnAfoZrM=",
    "load_all_comments": true,   # 是否加载全部评论
    "limit": 20,                 # 评论加载上限
    "click_more_replies": false, # 是否展开二级回复
    "scroll_speed": "normal"     # slow | normal | fast
}
```

返回：正文内容（`desc`）、图片列表（`imageList`）、评论列表（`comments`，含 `comment_id` 和 `user_id`）

### 3️⃣ 点赞/收藏

```python
# 点赞
{"feed_id": "...", "xsec_token": "..."}

# 取消点赞
{"feed_id": "...", "xsec_token": "...", "unlike": true}

# 收藏
{"feed_id": "...", "xsec_token": "..."}

# 取消收藏
{"feed_id": "...", "xsec_token": "...", "unfavorite": true}
```

### 4️⃣ 发表评论 / 回复评论

```python
# 发表评论
{"feed_id": "...", "xsec_token": "...", "content": "写得真好！"}

# 回复某条评论（comment_id 和 user_id 从帖子详情获取）
{"feed_id": "...", "xsec_token": "...", "content": "谢谢！", "comment_id": "...", "user_id": "..."}
```

### 5️⃣ 查看用户主页

```python
# user_id 从搜索结果中的 noteCard.user.userId 获取
# xsec_token 用同一篇笔记的 xsecToken
{"user_id": "63a10c7b0000000026004a19", "xsec_token": "..."}
```

### 6️⃣ 首页推荐

```python
# 无参数，直接调用
{}
```

### 7️⃣ 发布笔记

```python
# 图文
{
    "title": "标题（≤20字）",
    "content": "正文（≤1000字）",
    "images": ["/path/to/image.jpg"],
    "tags": ["美食", "旅行"]
}

# 视频
{
    "title": "标题",
    "content": "正文",
    "video": "/path/to/video.mp4"
}
```

---

## 核心数据结构

```
search_feeds / list_feeds / user_profile
        │
        ▼
  返回 feeds 数组
  每个 feed 包含:
  ├── id              → 用作 feed_id（后续操作的钥匙🔑）
  ├── xsecToken       → 用作 xsec_token（后续操作的钥匙🔑）
  ├── modelType       → note | video
  └── noteCard
        ├── displayTitle → 标题
        ├── user
        │   ├── userId   → 用户 ID（查看主页用）
        │   └── nickname → 昵称
        └── interactInfo
              ├── likedCount      → 点赞数
              ├── commentCount    → 评论数
              ├── collectedCount  → 收藏数
              └── sharedCount     → 分享数
```

**重要规则：** `feed_id` 和 `xsec_token` 必须配对使用，不可跨笔记混用，不可自行构造。

---

## 实用 Python 调用模板

```python
import http.client, json

def xhs_call(tool_name: str, arguments: dict = {}) -> dict:
    """
    调用小红书 MCP 工具的通用函数。
    自动处理 Initialize → Notification → Call 三阶段流程。
    """
    conn = http.client.HTTPConnection("localhost", 18060)
    headers = {"Content-Type": "application/json", "Accept": "application/json, text/event-stream"}

    # 1. Initialize
    conn.request("POST", "/mcp", json.dumps({
        "jsonrpc":"2.0","id":1,"method":"initialize",
        "params":{"protocolVersion":"2024-11-05","capabilities":{},
                  "clientInfo":{"name":"openclaw","version":"1.0"}}
    }), headers)
    r = conn.getresponse()
    sid = r.getheader("Mcp-Session-Id")
    r.read()

    # 2. Initialized notification
    h = {**headers, "Mcp-Session-Id": sid}
    conn.request("POST", "/mcp", json.dumps({"jsonrpc":"2.0","method":"notifications/initialized"}), h)
    r = conn.getresponse(); r.read()

    # 3. Call tool
    conn.request("POST", "/mcp", json.dumps({
        "jsonrpc":"2.0","id":2,"method":"tools/call",
        "params":{"name":tool_name,"arguments":arguments}
    }), h)
    r = conn.getresponse()
    data = json.loads(r.read())
    conn.close()

    return data

# ===== 使用示例 =====

# 搜索
result = xhs_call("search_feeds", {"keyword": "AI工具"})
feeds_text = result["result"]["content"][0]["text"]
print(feeds_text)

# 帖子详情
detail = xhs_call("get_feed_detail", {
    "feed_id": "...",
    "xsec_token": "...",
    "load_all_comments": True,
    "limit": 10
})
print(detail["result"]["content"][0]["text"])
```

---

## 注意事项

1. **Session 有效期**：每次调用需重新获取 Session，不能复用
2. **Cookie 有效期**：约 30 天，过期后需要重新扫码
3. **频率控制**：避免短时间大量并发请求
4. **发布限制**：标题 ≤20 字符，正文 ≤1000 字符，每日 ≤50 条
5. **headless 模式**：启动时加 `-headless`，不弹浏览器窗口
