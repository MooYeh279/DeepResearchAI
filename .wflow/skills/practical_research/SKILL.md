---
name: practical_research
description: 实践信息检索与分析。多轮 Tavily 搜索 → 去重汇总 → 撰写实践研究报告。侧重开源工具、GitHub 项目、教程、技术栈选型、开发者最佳实践。
type: skill
version: 2.0.0
author: deep-research
tags: [practical, tools, github, tutorial, how-to, engineering, developer]
---

# practical_research

实践信息的深度检索与分析。三阶段流水线：**Search Pipeline**（脚本驱动）→ **Analysis**（AI 驱动）→ **Persist**（脚本驱动）。

## 完整工作流（7 步）

### Phase A — Search Pipeline

#### Step 1: 构造查询

根据研究主题构造 2-3 组递进式检索查询：

- **第 1 轮（工具发现）:** `"<topic> tool OR framework OR library open source site:github.com"`
- **第 2 轮（教程/实践）:** `"<topic> tutorial getting started best practices"`
- **第 3 轮（对比/选型）:** `"<tool_a> vs <tool_b> comparison benchmark 2024"`

> **域偏好:** `site:github.com` | `site:stackoverflow.com` | `site:pypi.org` | `site:npmjs.com` | `site:medium.com` | `site:dev.to` | `site:realpython.com`

#### Step 2: 执行多轮检索

```bash
python .wflow/scripts/tavily-search.py -n 10 -d advanced \
  "RAG framework tool open source site:github.com 2024" > practical_out/raw_r1.json

python .wflow/scripts/tavily-search.py -n 10 -d advanced \
  "how to deploy RAG in production tutorial best practices" > practical_out/raw_r2.json

python .wflow/scripts/tavily-search.py -n 10 -d advanced \
  "LangChain vs LlamaIndex vs Haystack comparison benchmark" > practical_out/raw_r3.json
```

#### Step 3: 合并 + 去重

```bash
python .wflow/scripts/dedup_db.py merge \
  -q "RAG framework tool open source site:github.com" \
     "how to deploy RAG in production tutorial" \
     "LangChain vs LlamaIndex vs Haystack comparison" \
  practical_out/raw_r1.json practical_out/raw_r2.json practical_out/raw_r3.json \
  | python .wflow/scripts/dedup_db.py filter --skill practical -i - \
  -o practical_out/deduped.json
```

### Phase B — Analysis

#### Step 4: 相关性筛选

阅读 `practical_out/deduped.json` 逐条判断并写入 `practical_out/result.json`：

```json
{
  "meta": {
    "skill": "practical",
    "topic": "RAG 检索增强生成框架",
    "search_date": "2025-06-16",
    "final_count": 14,
    "search_rounds": [
      {"round": 1, "query": "...", "results_count": 10},
      {"round": 2, "query": "...", "results_count": 10},
      {"round": 3, "query": "...", "results_count": 10}
    ]
  },
  "results": [
    {
      "title": "langchain-ai/langchain: Build context-aware reasoning applications",
      "url": "https://github.com/langchain-ai/langchain",
      "content": "LangChain is a framework for developing applications powered by LLMs...",
      "score": 0.94,
      "relevance": "high",
      "relevance_note": "最流行的 LLM 开发框架，直接相关",
      "_round": 1
    }
  ]
}
```

**筛选原则:** 保留含可执行代码/命令/教程、项目活跃（近 6 个月维护）、有文档；移除已归档/废弃项目、仅有概念无实操、过时教程（API 已变）。

> **验证清单:** 项目是否近 6 个月更新？依赖兼容性？许可证允许？有完整文档？

#### Step 5: 撰写报告

基于 `result.json` 撰写 `practical_out/practical_report.md`：

```markdown
# {Topic} — 实践研究报告

> 生成时间: {date} | 有效结果: {count} 条 | 检索: N 轮

## 1. 检索概览
（主题、日期、来源类型分布）

## 2. 核心发现
（3-5 条可落地的关键结论）

## 3. 工具/项目推荐

### 3.1 {工具名称}
| 维度 | 内容 |
|------|------|
| 项目地址 | {URL} |
| Stars/License | ★ 85K / MIT |
| 最近更新 | 2025-06-10 |
| 核心功能 | ... |
| 上手难度 | 低 / 中 / 高 |
| 社区活跃度 | 优秀 / 良好 / 一般 |
| 推荐理由 | ... |
| 注意事项 | ... |

## 4. 技术栈选型指南

| 方案 | Stars | 适合场景 | 优点 | 缺点 | 推荐度 |
|------|-------|----------|------|------|--------|
| A    | ★85K  | ...      | ...  | ...  | ★★★★★ |
| B    | ★12K  | ...      | ...  | ...  | ★★★☆☆ |

## 5. 教程与实践

（按难度递进：入门 → 进阶 → 生产部署，每级推荐 2-3 个资源）

## 6. 快速上手路线

```
1. 阅读 {入门教程} 了解基本概念
2. 克隆 {starter_template} 跑通 demo
3. 阅读 {best_practice_guide} 了解生产注意事项
4. 参考 {advanced_example} 实现自定义需求
```

## 7. 参考来源
| # | 标题 | URL | 来源类型 | 相关度 | 检索轮次 |
```

### Phase C — Persist

#### Step 6: 记录到数据库

```bash
python .wflow/scripts/dedup_db.py add --skill practical \
  -i practical_out/result.json -p
```

#### Step 7: 清理并呈现

```bash
rm -f practical_out/raw_r*.json practical_out/deduped.json
```

---

## 查询策略参考

### 实践关键词注入

| 意图 | 注入词 |
|------|--------|
| 寻找工具 | `tool`, `framework`, `library`, `open source` |
| 学习用法 | `tutorial`, `how to`, `getting started`, `guide` |
| 对比选型 | `vs`, `comparison`, `alternatives`, `benchmark` |
| 解决问题 | `error`, `fix`, `troubleshoot`, `debug` |
| 最佳实践 | `best practices`, `production`, `checklist`, `checklist` |
| 代码模板 | `starter`, `template`, `boilerplate`, `example` |

### 开发者源域限定

#### 开源项目

| 域 | 用途 |
|----|------|
| `site:github.com` | GitHub 仓库 |
| `site:gitlab.com` | GitLab 仓库 |
| `site:pypi.org` | Python 包 |
| `site:npmjs.com` | NPM 包 |

#### 开发者社区

| 域 | 用途 |
|----|------|
| `site:stackoverflow.com` | 问答、排错 |
| `site:dev.to` | 技术博客社区 |
| `site:reddit.com` | 社区讨论（`r/MachineLearning` 等） |
| `site:news.ycombinator.com` | 技术资讯 |

#### 教程站点

| 域 | 用途 |
|----|------|
| `site:medium.com` | 技术博客 |
| `site:towardsdatascience.com` | 数据科学教程 |
| `site:realpython.com` | Python 教程 |
| `site:digitalocean.com` | 运维教程 |

### 查询模式

| 目的 | 模板 |
|------|------|
| 工具发现 | `"<topic> tool OR framework open source site:github.com"` |
| 教程学习 | `"<topic> tutorial getting started step by step"` |
| 对比选型 | `"<tool_a> vs <tool_b> comparison benchmark"` |
| 问题排查 | `"<error_message> fix solution"` |
| 最佳实践 | `"<topic> best practices production deployment"` |
| 项目模板 | `"<topic> starter template OR boilerplate site:github.com"` |

## 环境变量

| 变量 | 必填 | 说明 |
|------|------|------|
| `TAVILY_API_KEY` | 是 | Tavily API 密钥 |

## 依赖

仅 Python 标准库。
