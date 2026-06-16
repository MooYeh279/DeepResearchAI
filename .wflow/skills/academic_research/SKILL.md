---
name: academic_research
description: 学术论文检索与分析。多轮 Tavily 搜索 → 去重汇总 → 撰写学术研究报告。侧重 arXiv、会议论文、文献综述等学术源。
type: skill
version: 2.0.0
author: deep-research
tags: [academic, paper, arxiv, research, literature, scholar]
---

# academic_research

学术论文与研究文献的深度检索与分析。三阶段流水线：**Search Pipeline**（脚本驱动）→ **Analysis**（AI 驱动）→ **Persist**（脚本驱动）。

## 完整工作流（7 步）

### Phase A — Search Pipeline

#### Step 1: 构造查询

根据研究主题构造 2-3 组递进式检索查询：

- **第 1 轮（广域扫描）:** `"<topic> survey OR review site:arxiv.org"`
- **第 2 轮（聚焦深挖）:** `"<specific_method> <topic> 2024 2025 site:arxiv.org OR site:openreview.net"`
- **第 3 轮（交叉验证）:** `"<topic> benchmark OR evaluation OR empirical study"`

> **域偏好:** `site:arxiv.org` | `site:openreview.net` | `site:aclanthology.org` | `site:semanticscholar.org` | `site:papers.nips.cc`

#### Step 2: 执行多轮检索

```bash
python .wflow/scripts/tavily-search.py -n 10 -d advanced \
  "LLM reasoning survey site:arxiv.org" > academic_out/raw_r1.json

python .wflow/scripts/tavily-search.py -n 10 -d advanced \
  "chain-of-thought reasoning 2024 2025 site:openreview.net" > academic_out/raw_r2.json

python .wflow/scripts/tavily-search.py -n 10 -d advanced \
  "LLM reasoning benchmark empirical study" > academic_out/raw_r3.json
```

#### Step 3: 合并 + 去重

```bash
python .wflow/scripts/dedup_db.py merge \
  -q "LLM reasoning survey site:arxiv.org" \
     "chain-of-thought reasoning 2024 2025" \
     "LLM reasoning benchmark empirical study" \
  academic_out/raw_r1.json academic_out/raw_r2.json academic_out/raw_r3.json \
  | python .wflow/scripts/dedup_db.py filter --skill academic -i - \
  -o academic_out/deduped.json
```

### Phase B — Analysis

#### Step 4: 相关性筛选

阅读 `academic_out/deduped.json` 逐条判断并写入 `academic_out/result.json`：

```json
{
  "meta": {
    "skill": "academic",
    "topic": "大语言模型推理能力",
    "search_date": "2025-06-16",
    "final_count": 18,
    "search_rounds": [
      {"round": 1, "query": "...", "results_count": 10},
      {"round": 2, "query": "...", "results_count": 10},
      {"round": 3, "query": "...", "results_count": 10}
    ]
  },
  "results": [
    {
      "title": "Chain-of-Thought Prompting Elicits Reasoning in LLMs",
      "url": "https://arxiv.org/abs/2201.11903",
      "content": "We explore how generating a chain of thought...",
      "score": 0.95,
      "relevance": "high",
      "relevance_note": "开创性工作，直接相关",
      "_round": 2
    }
  ]
}
```

**筛选原则:** 保留学术源（arxiv/会议/期刊）、直接相关、含方法/实验数据；移除纯新闻、仅提及关键字、过时（3 年前且无后续引用）。

#### Step 5: 撰写报告

基于 `result.json` 撰写 `academic_out/academic_report.md`：

```markdown
# {Topic} — 学术研究报告

> 生成时间: {date} | 有效结果: {count} 条 | 检索: N 轮

## 1. 检索概览
（主题、日期、来源分布、去重统计）

## 2. 核心发现
（3-5 条宏观发现，每条 2-3 句话 + 证据）

## 3. 论文综述
（按研究方向分组，每篇含: 核心贡献 / 方法 / 关键结果 / 局限）

## 4. 研究趋势与流派
（领域发展脉络、主要技术路线对比）

## 5. 方法论对比
| 方法/路线 | 代表论文 | 优点 | 局限 | 适用场景 |

## 6. 建议阅读清单
| # | 论文 | URL | 重要度 | 理由 |

## 7. 参考来源
| # | 标题 | URL | 相关度 | 检索轮次 |
```

### Phase C — Persist

#### Step 6: 记录到数据库

```bash
python .wflow/scripts/dedup_db.py add --skill academic \
  -i academic_out/result.json -p
```

#### Step 7: 清理并呈现

```bash
rm -f academic_out/raw_r*.json academic_out/deduped.json
```

---

## 查询策略参考

### 学术术语替换

| 口语/中文 | 学术术语 |
|-----------|---------|
| "怎么让 AI 推理" | "LLM reasoning capability" |
| "大模型微调" | "parameter-efficient fine-tuning LLM" |
| "知识问答" | "knowledge-grounded question answering" |

### 常用学术域限定

| 域 | 适用场景 |
|----|---------|
| `site:arxiv.org` | 预印本（最广覆盖） |
| `site:openreview.net` | ICLR/NeurIPS/ICML 审稿论文 |
| `site:aclanthology.org` | NLP 领域（ACL/EMNLP/NAACL） |
| `site:semanticscholar.org` | 跨领域学术索引 |
| `site:papers.nips.cc` | NeurIPS 正式论文集 |

### 查询模式

| 目的 | 模板 |
|------|------|
| 文献综述 | `"<topic> survey OR review"` |
| 方法追踪 | `"<method> improved OR extended 2024 2025"` |
| 对比评估 | `"<topic> benchmark OR empirical study"` |
| 前沿探索 | `"<topic> novel approach 2025 site:arxiv.org"` |

## 环境变量

| 变量 | 必填 | 说明 |
|------|------|------|
| `TAVILY_API_KEY` | 是 | Tavily API 密钥 |

## 依赖

仅 Python 标准库。
