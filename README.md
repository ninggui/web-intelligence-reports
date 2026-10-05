<img src="./assets/cover.png" alt="网络情报报告" width="100%">

<div align="center">

# 网络情报报告

**多角度搜索验证后出报告，品牌竞品舆情一键发飞书。**

![Status](https://img.shields.io/badge/status-production-green)
![Angles](https://img.shields.io/badge/search-%E4%B8%89%E8%A7%92%E5%BA%A6-blue)
![Verify](https://img.shields.io/badge/verify-%E5%8F%91%E5%B8%83%E5%89%8D%E5%BF%85%E9%AA%8C-green)
![Engine](https://img.shields.io/badge/engine-%E7%99%BE%E5%BA%A6%2BTavily-orange)
![License](https://img.shields.io/badge/license-MIT-blue)

[它解决什么问题](#它解决什么问题) - [为什么比手动强](#为什么比手动强) - [工作流](#工作流) - [实测参数](#实测参数) - [快速开始](#快速开始)

</div>

---

## 它解决什么问题

监控一个品牌或竞品，搜出来一堆软文和过时新闻，直接发出去容易翻车：假数据、过期信息、广告软文混进来。这套流程规定三角度同时搜、每条关键信息发布前二次验证标来源。

## 为什么比手动强

| 随手搜直接写 | 本仓库 |
|---|---|
| 只搜好话或只搜坏话 | 正面/负面/行业三角度同时搜 |
| 照抄搜索结果 | 每条关键信息二次验证、标原始链接 |
| 报告散乱无结构 | 按舆论类型分区 + 末尾舆情总结表 |
| 过期新闻混进来 | 代码层年份过滤 + AI 只取当月 |

## 工作流

```
多角度搜索(正/负/行业)
  → 验证真实性(二次搜索 + 标URL)
  → 编译报告(按舆论类型分区,时间倒序)
  → 发布飞书文档 / 群卡片
```

## 实测参数

- **阿维塔案例**：3 轮搜索 → 20 条信息 → 飞书文档，全部验证真实
- **早报模式**：18 条（6 板块 × 3），280±20 字/条，四要素 What/SoWhat/NowWhat/Source
- **搜索后端**：百度 AI 搜索 100 次/天主力 + Tavily 100 次/月辅助，勿单用 Tavily
- 参考实现见 `references/carnews-push-system.md`

## 快速开始

```bash
web_search("品牌名 评测 销量 2026")   # 正面/中性
web_search("品牌名 投诉 维权 问题")   # 负面
web_search("品牌名 行业 趋势 对比")   # 行业
# 三角度汇总 → 去重验证 → 写报告 → 发飞书文档
```

## License

MIT
