# 小红书品牌官方号语料抓取 SOP（2026-08-13 实测验证）

> 用途：抓取品牌官方小红书号的笔记标题+正文，作为品牌语言风格研究/品牌深度研究的语料来源。
> 环境：xiaohongshu-mcp 容器（18060），MCP 调用经 docker exec curl 转发。
> 案例：蔚来官方号 62条笔记全量 + 7篇全文（skill nio-brand-voice v2.2 落地）

## 为什么需要（背景）

品牌语言风格研究需要官方号一手文案（与发布会演讲互补）。直接 `search_feeds "品牌名"` 返回的全是个人用户/营销号，官方号被推荐流淹没。必须用本 SOP 定位官方号。

## 步骤

### 1. 定位官方号（多关键词 + 按点赞排序）

```python
# 单关键词搜索会被个人用户淹没，必须多角度：
search_feeds("品牌名", {"sort_by": "最多点赞"})   # 官方内容通常高赞
search_feeds("品牌名NIO/NIO Day/换电站", ...)      # 品牌专有词/子品牌/业务词
# 子品牌账号（如乐道/萤火虫）也是官方体系
```

官方号识别特征：昵称=品牌名（如"蔚来"）、简介=使命缩写、粉丝量大、IP在北京/上海、发每日播报/里程碑类内容。

### 2. 提取官方号笔记的 userId + xsecToken

```python
# 官方号笔记的原始字段（注意不是 noteCard.user 里的，是顶层）：
fd["id"]              # 笔记ID（完整24位，不可截断）
fd["xsecToken"]       # 访问令牌（会轮换）
nc["user"]["userId"]  # 注意：24位完整ID！截断20位做匹配会失败
```

**坑1**：`userId` 匹配时若截断（如取前20位），匹配永远失败。必须用完整24位ID。

### 3. 用 user_profile 拉取全部笔记列表

```python
user_profile(user_id="<完整24位ID>", xsec_token="<任一笔记token>", tab="note")
# 返回：userBasicInfo + interactions（粉丝数/赞藏）+ feeds（全部笔记标题+新token）
# 蔚来官方号：18.4万粉/73.5万赞藏/62条笔记，一次拉全
```

### 4. 保存最新 token 映射，批量拉正文

```python
# 第一次 user_profile 返回的 token 会在后续 get_feed_detail 时部分失效
# 正确做法：重新 user_profile 刷新全部 token → 存 {note_id: token} 映射
# 然后逐条 get_feed_detail(feed_id, xsec_token) 拉正文
```

**坑2**：xsecToken 会轮换。同一 token 隔一段时间后 get_feed_detail 返回 118字节错误。批量前先重新 user_profile 刷新 token 表。

**坑3**：批量拉取会超时（每条 curl --max-time 90 + 间隔3秒，12条≈4分钟）。用 background 执行或分批，单批≤10条。

### 5. 正文解析

```python
# get_feed_detail 返回 JSON，正文在：
data["data"]["note"]["desc"]    # 正文（含emoji/换行/话题标签）
data["data"]["note"]["title"]   # 标题
data["data"]["note"]["interactInfo"]  # 互动数据
```

## 复用提示

- 换品牌时替换关键词与官方号识别特征即可
- 语料更新（如半年后再抓）→ 重新执行本 SOP，增量并入 skill 语料库
- 所有 MCP 调用脚本模式：写 payload.json → docker cp 进容器 → docker exec curl → 存响应文件
