# 新能源汽车早报推送系统 — 完整参考

> 实施日期：2026-08-08 | 容器：service:2.0

## 系统架构

```
app_main_fixed.py (定时调度 08:30/16:30)
  → processor_ai.py (Tavily搜索 + AI增强 + 飞书推送)
    → config.json (模型/搜索/内容配置)
```

## AI模型配置

- **提供商**: 硅基流动 SiliconFlow (`api.siliconflow.cn/v1`)
- **模型**: `deepseek-ai/DeepSeek-V3.2`（中文质量最优）
- **备选**: `Qwen/Qwen3.5-35B-A3B`（MoE省钱）
- **温度**: 0.3
- **max_tokens**: 3500

## Tavily多角度搜索关键词

每板块3组（2组中文 + 1组英文 + 可选站点定向）：

| 板块 | 中文关键词 | 英文关键词 | 站点定向 |
|------|-----------|-----------|---------|
| 🚨 风险预警 | 新能源汽车 召回 自燃 事故 投诉 | EV recall fire accident safety | site:12365auto.com |
| ⚖️ 政策监管 | 工信部 新能源 政策 法规 标准 | China EV policy regulation | - |
| 🔗 供应链 | 动力电池 宁德时代 产能 价格 锂电 | EV battery supply chain lithium | - |
| ⚔️ 友商动态 | 蔚来 小鹏 理想 零跑 极氪 销量 | Nio Xpeng Li Auto BYD sales | - |
| 🚀 技术趋势 | 固态电池 钠离子 超快充 800V 智驾 | solid state battery autonomous | - |
| 📈 资本市场 | 新能源 融资 IPO 股价 财报 | EV startup funding IPO China | - |

## 质量评分

- 高价值关键词（发布/宣布/投产/量产/突破/召回/自燃）: +8
- 数据指标（亿元/%/GWh/万辆）: +4
- 官方来源（工信部/发改委/乘联会）: +6
- 权威域名（12365auto.com/工信部）: +15
- 内容>500字: +10

## AI提示词（核心）

见飞书文档：https://my.feishu.cn/docx/BeM2dCScsoYcNSxIaGJcyNafn5b

要点：
- What/So What/Now What/Source 四要素
- 280±20字
- 风险等级标注（🔴高/🟡中/🟢低）
- 时效性铁律：仅24h内新闻，超48h跳过
- 编号格式：01/18~18/18

## 飞书推送格式

使用机器人Webhook `interactive` 卡片 + `markdown` 元素：

```json
{
  "msg_type": "interactive",
  "card": {
    "header": {"title": {"tag": "plain_text", "content": "🚗 新能源汽车行业早报"}, "template": "blue"},
    "elements": [{"tag": "markdown", "content": "..."}]
  }
}
```

## 搜索架构（v10 最终版）

### 双引擎策略

| 层 | 引擎 | 配额 | 每轮调用 | 用途 |
|:---:|------|:---:|:---:|------|
| 1 | **百度AI搜索** | 100次/天 | 2×6=12次 | 中文主力 |
| 2 | **Tavily** | 100次/月 | 1×6=6次 | 英文辅助 |

每天2次推送 = 百度24次 + Tavily 12次。Tavily 360次/月已超免费额度——**必须限Tavily到1query/板块。**

百度AI Key：`bce-v3/ALTAK-<REPLACE_WITH_YOUR_KEY>`
API：`https://qianfan.baidubce.com/v2/ai_search/chat/completions`

### 时效性三层过滤（10轮迭代验证）

1. **代码层**：正则 `20\d{2}年` 匹配标题，<当前年份→丢弃（仅标题，不搜正文）
2. **AI层**：仅生成当月新闻，无日期静默跳过，不使用"⏱待核实"
3. **AI层**：日期允许到月（`⏱ 2026-08`），不强制到日——搜索API不返回精确日期
4. **AI层**：禁止输出"跳过""不符合""素材日期不符"等元文本——只输出内容或"无当月新闻"

## 常见问题与修复

| 问题 | 根因 | 修复 |
|------|------|------|
| MoonShot 403 | 模型已停用 | 切 SiliconFlow DeepSeek-V3.2 |
| RSS源不可用 | 中文汽车RSS大面积失效 | 全切搜索API |
| 编号溢出 21/20 | 6×4=24但模板说20 | 改为6×3=18，模板同步 |
| 双重⏱⏱ | 模板⏱+AI日期字段冲突 | 统一为单⏱+月级日期 |
| 旧闻混入 | 搜索API不限时 | 代码标题年过滤+AI当月过滤 |
| 搜索量不足 | 单一Tavily源 | 接入百度AI搜索 |
| AI输出元文本 | 提示词未禁止 | 添加"静默跳过，不输出说明文字" |
| 域名白名单0结果 | Tavily结果域名分散 | 改信任加分制，不拦截 |
| Tavily超额 | 36次/天=1080次/月 | 限1query/板块，百度主力 |

## Docker部署命令

```bash
# 部署代码
docker cp config.json service:/app/config.json
docker cp processor_ai.py service:/app/processor_ai.py

# 后台运行（长任务）
docker exec service python3 /app/processor_ai.py &

# 查看日志
docker exec service tail -f /app/logs/app_main.log
```
