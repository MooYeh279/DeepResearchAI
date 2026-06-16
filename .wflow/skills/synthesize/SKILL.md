---
name: synthesize
description: 综合归纳三份研究报告（学术/行业/实践）为一份结构化的综合报告，含交叉洞察、角色化建议和可执行路线图。
type: skill
version: 2.0.0
author: deep-research
tags: [synthesize, report, merge, analysis, recommendations, deep-research]
---

# synthesize

综合归纳与报告生成。三阶段流水线：**Collect**（读取上游报告）→ **Synthesize**（AI 驱动交叉分析）→ **Persist**（保存综合报告）。

## 前置条件

本 skill 在工作流中处于最后阶段，依赖以下上游节点全部通过审核：

- `academic_research` — 学术研究报告 (`academic_out/academic_report.md`)
- `industry_research` — 行业研究报告 (`industry_out/industry_report.md`)
- `practical_research` — 实践研究报告 (`practical_out/practical_report.md`)

## 完整工作流（6 步）

### Phase A — Collect

#### Step 1: 确认上游输出

验证三份报告文件存在且可读：

```bash
ls -la academic_out/academic_report.md industry_out/industry_report.md practical_out/practical_report.md
```

如果任何文件缺失，中止并报告缺失的报告。

#### Step 2: 读取三份报告

使用 Read 工具完整读取三份报告：
- `academic_out/academic_report.md`
- `industry_out/industry_report.md`
- `practical_out/practical_report.md`

提取各报告的核心结构：关键发现、方法论、数据、来源。

### Phase B — Synthesize

#### Step 3: 执行交叉分析

在撰写综合报告前，先完成以下交叉分析：

**3.1 一致性验证 (Convergence)**
识别三个维度中相互印证的发现：
- 学术论文中的方法是否在行业案例中得到应用？
- 行业趋势是否被实践工具生态所支持？
- 实践中的最佳实践是否有学术理论支撑？

输出：3-5 条"三维印证"发现。

**3.2 矛盾识别 (Divergence)**
识别三个维度之间的张力和矛盾：
- 学术前沿 vs 行业落地之间的时间差（学术领先/行业滞后）
- 行业宣传 vs 实践现实的差距（过度营销 vs 实际效果）
- 不同维度对同一问题的对立观点

输出：2-3 条"矛盾/张力"发现。

**3.3 缺口分析 (Gap Analysis)**
识别三个维度共同缺失的领域：
- 是否有重要方向未被任何维度覆盖？
- 是否有新兴趋势在报告中尚未体现？

输出：1-2 条"研究缺口"说明。

#### Step 4: 撰写综合报告

创建 `result/` 目录并撰写 `result/{topic_slug}_research_report.md`：

```bash
mkdir -p result
```

---

## 综合报告模板

```markdown
# {研究主题} — 深度研究综合报告

> **生成日期:** {YYYY-MM-DD}
> **研究范围:** 学术前沿 · 行业格局 · 实践落地
> **数据来源:** 学术论文 {N} 篇 | 行业来源 {M} 条 | 实践工具 {K} 项

---

## 执行摘要

> {200-300 字，综合三个维度的核心发现}
>
> **关键结论:**
> 1. {结论一 — 最重要的跨维度洞察}
> 2. {结论二}
> 3. {结论三}

---

## 1. 学术前沿

> 来源: 学术研究报告

### 1.1 研究全景

{概括学术领域的研究现状、主要方向和活跃度}

### 1.2 关键论文与方法论

| # | 论文/方向 | 核心贡献 | 方法论 | 局限 | 来源 |
|---|----------|---------|--------|------|------|
| 1 | {paper} | {contribution} | {method} | {limitation} | {ref} |

### 1.3 技术路线对比

| 路线 | 代表工作 | 优势 | 劣势 | 成熟度 |
|------|---------|------|------|--------|
| {approach_a} | {paper} | ... | ... | 高/中/低 |

### 1.4 学术趋势研判

{学术领域的发展方向、新兴热点、潜在突破点}

---

## 2. 行业洞察

> 来源: 行业研究报告

### 2.1 市场概览

{市场规模、增长率、主要驱动力、区域分布}

### 2.2 竞争格局

| 厂商/产品 | 定位 | 核心优势 | 主要不足 | 市场阶段 |
|----------|------|---------|---------|---------|
| {vendor} | {positioning} | {strength} | {weakness} | 成熟/成长/新兴 |

### 2.3 企业实践精选

| 企业 | 行业 | 应用场景 | 方案要点 | 效果 | 可信度 |
|------|------|---------|---------|------|--------|
| {company} | {industry} | {scenario} | {approach} | {result} | 高/中 |

### 2.4 行业趋势研判

{短期(1-2年) / 中期(3-5年) 趋势，附研判依据}

---

## 3. 实践指南

> 来源: 实践研究报告

### 3.1 工具生态总览

| 工具/项目 | Stars | 定位 | 上手难度 | 社区活跃度 | 推荐度 |
|----------|-------|------|---------|-----------|--------|
| {tool} | ★{N}K | {positioning} | 低/中/高 | 优秀/良好/一般 | ★★★★★ |

### 3.2 技术选型建议

**场景一: {场景名称}**
- 推荐方案: {tool_a} + {tool_b}
- 理由: {rationale}
- 注意事项: {caveats}

**场景二: {场景名称}**
- 推荐方案: {tool_c}
- 理由: {rationale}
- 注意事项: {caveats}

### 3.3 快速上手路线

```
第 1 步: {入门动作} — 预计 {N} 小时
第 2 步: {进阶动作} — 预计 {N} 小时
第 3 步: {实战动作} — 预计 {N} 小时
第 4 步: {生产部署} — 预计 {N} 小时
```

---

## 4. 交叉洞察

> 综合学术、行业、实践三个维度的交叉分析

### 4.1 三维印证 (Convergence)

{三个维度相互印证的发现，3-5 条}

| # | 洞察 | 学术证据 | 行业证据 | 实践证据 | 可信度 |
|---|------|---------|---------|---------|--------|
| 1 | {insight} | {academic_evidence} | {industry_evidence} | {practical_evidence} | 高/中 |

### 4.2 矛盾与张力 (Divergence)

{三个维度之间的矛盾和张力，2-3 条}

| # | 矛盾描述 | 维度 A 观点 | 维度 B 观点 | 分析 |
|---|---------|-----------|-----------|------|
| 1 | {tension} | {view_a} | {view_b} | {analysis} |

### 4.3 研究缺口 (Gap Analysis)

{当前研究覆盖不足的领域}

- **缺口 1:** {描述} — 建议后续关注 {direction}
- **缺口 2:** {描述} — 建议后续关注 {direction}

---

## 5. 综合建议

### 面向研究员

{学术研究方向的建议，含具体可开展的课题}

1. {recommendation_1}
2. {recommendation_2}
3. {recommendation_3}

### 面向技术决策者

{技术选型和战略规划的建议}

1. {recommendation_1}
2. {recommendation_2}
3. {recommendation_3}

### 面向开发者

{学习和实践的建议}

1. {recommendation_1}
2. {recommendation_2}
3. {recommendation_3}

---

## 6. 参考来源汇总

### 学术来源
| # | 标题 | URL | 类型 | 重要度 |
|---|------|-----|------|--------|
| 1 | {title} | {url} | 论文/综述/会议 | 高/中 |

### 行业来源
| # | 标题 | URL | 类型 | 可信度 |
|---|------|-----|------|--------|
| 1 | {title} | {url} | 分析报告/企业博客/新闻 | 高/中/低 |

### 实践来源
| # | 标题 | URL | 类型 | 相关度 |
|---|------|-----|------|--------|
| 1 | {title} | {url} | 项目/教程/问答 | 高/中 |

---

> **报告生成:** {YYYY-MM-DD HH:MM} | **生成引擎:** deep-research v3.0 / synthesize skill
> **免责声明:** 本报告由 AI 辅助生成，基于公开可获取的信息源。所有观点仅供研究参考，不构成专业建议。
```

---

### Phase C — Persist

#### Step 5: 验证并保存

```bash
# 确认文件已写入
wc -c result/*_research_report.md
```

验证要点：
- 所有 6 个主要章节均已覆盖
- 交叉洞察至少包含 2 条印证和 1 条矛盾
- 角色化建议分别针对三类角色
- 参考来源汇总包含来自三份原始报告的来源

#### Step 6: 输出结构化结果

```json
{
  "report_path": "result/{topic_slug}_research_report.md",
  "summary": "{200-300 字的执行摘要文本}",
  "word_count": {整数}
}
```

---

## 质量标准

### 综合报告 Checklist

- [ ] 执行摘要精炼（200-300 字），独立可读
- [ ] 学术/行业/实践三个章节均有实质性内容（非简单复制粘贴）
- [ ] 交叉洞察包含: 印证 ≥ 3 条 + 矛盾 ≥ 1 条 + 缺口 ≥ 1 条
- [ ] 综合建议覆盖: 研究员 / 技术决策者 / 开发者 三个角色
- [ ] 参考来源汇总分类清晰，包含来自三份原始报告的关键来源
- [ ] 报告语言与 `inputs.language` 一致（zh 输出中文，en 输出英文）
- [ ] 文件中无占位符 `{xxx}` 残留（已全部替换为实际内容）

### 常见问题避免

| 问题 | 正确做法 |
|------|---------|
| 纯拼接三份报告 | 提炼、重组、交叉对照，写出新洞察 |
| 交叉洞察空洞 | 具体引用三个维度中的证据，点明联系和矛盾 |
| 建议泛泛而谈 | 面向具体角色给出可执行的建议，附理由 |
| 忽略矛盾 | 主动识别学术前沿与行业落地之间的时间差、过度营销等问题 |
| 来源丢失 | 保留三份原始报告的关键来源链接 |

---

## topic_slug 生成规则

将研究主题转换为合法的文件名 slug：

```python
import re

def topic_to_slug(topic: str) -> str:
    """将中文/英文主题转为文件名 slug"""
    # 移除特殊字符，保留中英文、数字、连字符
    slug = re.sub(r'[^\w\s-]', '', topic)
    # 空白替换为连字符
    slug = re.sub(r'[\s_]+', '-', slug)
    # 去除首尾连字符，转小写
    slug = slug.strip('-').lower()
    # 限制长度
    return slug[:80]
```

示例: `"大语言模型在代码审查中的应用"` → `"大语言模型在代码审查中的应用"` (中文保留) 或 `"llm-code-review-application"` (英文 slug)

---

## 依赖

- 上游三份研究报告（由 academic_research / industry_research / practical_research 产出）
- 无外部 API 依赖
- 仅需 Read / Write / Bash 工具
