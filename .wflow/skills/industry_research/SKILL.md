---
name: industry_research
description: 业界与企业级信息检索与分析。多轮 Tavily 搜索 → 去重汇总 → 撰写行业研究报告。侧重市场报告、企业实践、竞争格局、趋势研判。
type: skill
version: 2.0.0
author: deep-research
tags: [industry, enterprise, market, business, analyst, report]
---

# industry_research

业界与企业级信息的深度检索与分析。三阶段流水线：**Search Pipeline**（脚本驱动）→ **Analysis**（AI 驱动）→ **Persist**（脚本驱动）。

## 完整工作流（7 步）

### Phase A — Search Pipeline

#### Step 1: 构造查询

根据研究主题构造 2-3 组递进式检索查询：

- **第 1 轮（市场概览）:** `"<topic> market size forecast 2024 2025"`
- **第 2 轮（企业实践）:** `"<topic> enterprise deployment production case study"`
- **第 3 轮（竞争/趋势）:** `"<topic> vendor comparison OR trends 2025"`

> **域偏好:** `site:gartner.com` | `site:mckinsey.com` | `site:idc.com` | `site:forrester.com` | `site:techcrunch.com` | `site:36kr.com` | `site:infoq.com`

#### Step 2: 执行多轮检索

```bash
python .wflow/scripts/tavily-search.py -n 10 -d advanced \
  "LLM enterprise adoption market size forecast 2024 2025" > industry_out/raw_r1.json

python .wflow/scripts/tavily-search.py -n 10 -d advanced \
  "LLM enterprise deployment production case study" > industry_out/raw_r2.json

python .wflow/scripts/tavily-search.py -n 10 -d advanced \
  "generative AI vendor comparison landscape trends 2025" > industry_out/raw_r3.json
```

#### Step 3: 合并 + 去重

```bash
python .wflow/scripts/dedup_db.py merge \
  -q "LLM enterprise adoption market size forecast" \
     "LLM enterprise deployment production case study" \
     "generative AI vendor comparison landscape trends 2025" \
  industry_out/raw_r1.json industry_out/raw_r2.json industry_out/raw_r3.json \
  | python .wflow/scripts/dedup_db.py filter --skill industry -i - \
  -o industry_out/deduped.json
```

### Phase B — Analysis

#### Step 4: 相关性筛选

阅读 `industry_out/deduped.json` 逐条判断并写入 `industry_out/result.json`：

```json
{
  "meta": {
    "skill": "industry",
    "topic": "大语言模型企业落地",
    "search_date": "2025-06-16",
    "final_count": 15,
    "search_rounds": [
      {"round": 1, "query": "...", "results_count": 10},
      {"round": 2, "query": "...", "results_count": 10},
      {"round": 3, "query": "...", "results_count": 10}
    ]
  },
  "results": [
    {
      "title": "Gartner: 80% of Enterprises Will Deploy GenAI by 2026",
      "url": "https://www.gartner.com/en/newsroom/...",
      "content": "By 2026, more than 80% of enterprises will have used...",
      "score": 0.93,
      "relevance": "high",
      "relevance_note": "权威分析机构，量化预测数据",
      "_round": 1
    }
  ]
}
```

**筛选原则:** 保留分析机构报告、厂商技术实践、量化数据、时效性高（6 个月内）；移除纯产品广告、不可靠来源。

> **可信度注意:** 厂商博客可能有推广倾向（可信度中），Gartner/IDC/麦肯锡等第三方分析可信度更高（可信度高）。

#### Step 5: 撰写报告

基于 `result.json` 撰写 `industry_out/industry_report.md`：

```markdown
# {Topic} — 行业研究报告

> 生成时间: {date} | 有效结果: {count} 条 | 检索: N 轮

## 1. 检索概览
（主题、日期、来源类型分布、可信度分层统计）

## 2. 核心发现
（3-5 条关键洞察，每条附数据支撑和来源）

## 3. 市场概览
（市场规模、增长率、主要驱动因素、区域分布）

## 4. 企业实践案例
（按行业/场景分组，每案例含: 企业 / 场景 / 方案 / 效果 / 来源可信度）

## 5. 竞争格局
| 厂商/产品 | 定位 | 优势 | 劣势 | 市场份额/影响力 |

## 6. 趋势研判
（短期 1-2 年 / 中期 3-5 年趋势，附研判依据）

## 7. 风险与挑战
（技术风险、市场风险、合规风险，每项附来源）

## 8. 参考来源
| # | 标题 | URL | 来源类型 | 可信度 | 检索轮次 |
```

### Phase C — Persist

#### Step 6: 记录到数据库

```bash
python .wflow/scripts/dedup_db.py add --skill industry \
  -i industry_out/result.json -p
```

#### Step 7: 清理并呈现

```bash
rm -f industry_out/raw_r*.json industry_out/deduped.json
```

---

## 查询策略参考

### 行业关键词注入

| 意图 | 注入词 |
|------|--------|
| 市场规模 | `market size`, `forecast`, `CAGR`, `TAM` |
| 企业落地 | `enterprise`, `production`, `deployment`, `case study` |
| 竞争格局 | `landscape`, `vendor comparison`, `market share`, `vs` |
| 投资趋势 | `funding`, `investment`, `valuation`, `startup` |
| 合规政策 | `regulation`, `compliance`, `policy`, `governance` |

### 权威来源域限定

#### 分析机构

| 域 | 侧重 |
|----|------|
| `site:gartner.com` | 技术成熟度、魔力象限 |
| `site:idc.com` | 市场份额、出货量数据 |
| `site:mckinsey.com` | 战略洞察、行业转型 |
| `site:forrester.com` | 技术评估、企业架构 |

#### 厂商技术博客

| 域 | 侧重 |
|----|------|
| `site:engineering.fb.com` | Meta 基础设施 |
| `site:netflixtechblog.com` | 微服务、流媒体 |
| `site:aws.amazon.com/blogs` | AWS 架构实践 |
| `site:cloud.google.com/blog` | Google Cloud 方案 |

#### 财经/科技媒体

| 域 | 侧重 |
|----|------|
| `site:techcrunch.com` | 创业融资 |
| `site:36kr.com` | 国内创投 |
| `site:infoq.com` | 技术架构资讯 |

### 查询模式

| 目的 | 模板 |
|------|------|
| 市场概览 | `"<topic> market forecast 2024 2025 site:gartner.com OR site:idc.com"` |
| 企业实践 | `"<topic> enterprise deployment case study best practices"` |
| 竞争分析 | `"<topic> vendor comparison landscape 2025"` |
| 趋势研判 | `"<topic> trends predictions 2025 site:mckinsey.com"` |
| 合规政策 | `"<topic> regulation compliance policy enterprise"` |

### 中英文双语检索

行业信息在中英文源差异大，建议双语检索：

```bash
# 国际视角
python .wflow/scripts/tavily-search.py -n 5 "LLM enterprise adoption survey 2024"

# 国内视角
python .wflow/scripts/tavily-search.py -n 5 "大模型 企业落地 应用现状 2024"
```

## 环境变量

| 变量 | 必填 | 说明 |
|------|------|------|
| `TAVILY_API_KEY` | 是 | Tavily API 密钥 |

## 依赖

仅 Python 标准库。
