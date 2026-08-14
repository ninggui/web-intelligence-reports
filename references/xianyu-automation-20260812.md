# 闲鱼自动化研究（2026-08-12，研究前置协议首个案例）

## 需求 → 方案映射

| 需求 | 方案 | 可行性 |
|------|------|--------|
| 借用 AI 搜索筛选商品 | goofish-cli `search items`（Playwright 浏览器路径）；官方 AI智搜需 APP 内操作 | 🟡 部分可行 |
| 收藏商品 | goofish-cli 暂无收藏命令 | 🔴 待验证（可用自建"监控清单"替代） |
| 自动回复/上架/自动发货 | goofish-cli `message send`/`item publish` + OpenClaw Docker | 🟢 完全可行 |
| 跟踪上新+提醒 | ai-goofish-monitor / 自建 cron 轮询 search → 飞书提醒 | 🟢 完全可行 |

## 核心方案

### goofish-cli（首选，专为 AI Agent 设计）
- GitHub: fancyboi999/goofish-cli，Apache 2.0
- 安装: `pip install goofish-cli` / `uv pip install goofish-cli`
- 登录: 浏览器导出 cookies JSON → `goofish auth login <json>` 或 `--qr` 扫码
- 17 命令: item get/publish/delete、search items、message list-chats/send/watch、media upload、category recommend、auth status 等
- **原生 MCP 支持**: `uvx goofish-cli` 自动注册 MCP tool → 可接入 Hermes
- 内置风控护栏: 令牌桶限流（1写/分钟）+ RGV587 自动熔断
- 5 Skill: goofish-overview / publish-item / reply-buyer / risk-guard / shop-diagnosis
- 登录态 cookie 有时效（h5_token），过期需重导
- CLI 路径: hermes 在 `/opt/hermes/.venv/bin/hermes`（不在默认 PATH）

### OpenClaw（自动回复+自动发货）
- Docker: `docker pull careyke/openclaw` + `-e FTOKEN="<f-token>" -e MODE="xianyu"`
- f-token: Chrome 登录 2.taobao.com → F12 → Application → Cookies
- 关键词自动回复 + 付款后自动发网盘链接（虚拟商品）
- 2核2G 云服务器即可

### xianyu-mcp-server / XianyuAutoAgent
- xianyu-mcp-server: 闲鱼能力封装 MCP 工具，接 Trae/Claude Desktop/Cursor
- XianyuAutoAgent: 多专家协同智能客服（议价/技术/客服专家，三层意图路由），COOKIES_STR 网页端 Cookie

### 监控/秒拍
- ai-goofish-monitor（Usagi-org）: Playwright+AI 多任务监控+后台 UI
- 闲鱼监控助手: 关键词+价格区间+地区 → 微信/短信提醒
- Python 秒拍: AutoJS（安卓 root）/ Selenium + 代理池 + 随机 UA

## 风控红线（封号高危）

| 行为 | 风险 |
|------|------|
| 站外导流（发微信/手机号） | **封号重灾区**——买家发外部联系方式后继续对话即触发 |
| 纯数字取件码/快递单号 | 触发风控，需带文字标注"取件码""快递公司" |
| 主动核对收货地址 | 多说多错，触发封号 |
| 虚拟商品违规（版权/违禁/灰色） | 下架+扣分+限流（版权/违禁/代做代写/同行举报四类） |
| 高频自动化 | 无护栏工具易被检测 |

## 落地建议（用户场景）

**短期**: 安装 goofish-cli → 导出 cookies → 接入 Hermes MCP → 验证 search items（需求1）→ message 自动回复（需求3前半）
**中期**: 自建 cron 轮询 search items → 新商品检测 → 飞书提醒（需求4）→ 验证收藏（需求2）
**长期**: OpenClaw 容器化自动发货（需求3后半，注意虚拟商品风控）
**不推荐**: 秒拍/抢单脚本（封号风险极高）

## 研究流程沉淀（研究前置协议执行样板）

1. 本地检查是否研究过（.learnings/ + 文档库表）→ 全新领域
2. 6 组关键词并行搜索（API自动化/AI搜索/MCP/爬虫监控/自动发货/自动拍下）
3. web_extract 432 兜底 → 写 fetch_article.py（urllib+UA+正则）抓 GitHub raw README
4. 产出: 本地 .learnings/闲鱼自动化研究_2026-08-12.md + 飞书文档（docs +create --doc-format markdown）+ 文档库表登记（base +record-upsert）

## 来源
- github.com/fancyboi999/goofish-cli（README 实测）
- github.com/topics/xianyu（60个闲鱼开源项目）
- 腾讯云开发者社区: OpenClaw 部署教程/自动发货方案
- CSDN: XianyuAutoAgent API 解析
- 微博: 闲鱼严查封号保命规则
- 什么值得买: 闲鱼虚拟商品下架原因
