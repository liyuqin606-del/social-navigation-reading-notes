# 社会导航论文阅读笔记

2026-11-08 私有归档。包含三篇论文精读、数学与计算机前置知识、World Model 文献调研，以及后续理论设计。**阅读分析、待验证设计与实验结果分别标注。** 复现和实验另见 [social-navigation-reproduction](https://github.com/liyuqin606-del/social-navigation-reproduction)。

## 三篇精读

| 论文 | 阅读笔记 | 原文 |
|---|---|---|
| Falcon — From Cognition to Precognition | [方法、公式、前置知识与研究方向](research/falcon_notes.md) | [DOI](https://doi.org/10.1109/ICRA55743.2025.11127910) |
| NavThinker | [论文地图、方法链与负面证据](research/navthinker_notes.md) | [arXiv v2](https://arxiv.org/abs/2603.15359v2) |
| DPed-VLN | [语言先验、DPet、指标与实验分析](research/dped_notes.md) | [arXiv v1](https://arxiv.org/abs/2609.21504v1) |

Falcon 是点目标社会导航，DPed-VLN 是动态行人下的视觉语言导航；不能直接混用任务与成绩。

## 先读这些

1. [三篇导航论文：前置知识与研究路线](output/三篇导航论文_前置知识与研究路线.md) · [PDF](output/三篇导航论文_前置知识与研究路线.pdf)
2. [数学／物理／World Model 文献调研](output/导航优化_数学物理WorldModel_文献调研_2026-11-03.md) · [PDF](output/pdf/导航优化_数学物理WorldModel_文献调研_2026-11-03.pdf)
3. [权重编辑与导航研究建议](output/权重编辑_导航研究建议.md)
4. [近期问题与改进空间筛选](research/headroom_audit_20261108/REPORT.md)

## 专题和理论设计

- [数学](research/literature_2026-11-03/math_notes.md)、[物理与几何](research/literature_2026-11-03/physics_notes.md)、[世界模型](research/literature_2026-11-03/worldmodel_notes.md)、[权重编辑](research/literature_2026-11-03/editing_notes.md)、[综合分析](research/literature_2026-11-03/synthesis.md)。
- [Astra 数学建模](research/decision_world_model_20261104/ASTRA_MATHEMATICAL_MODELING.md)、[成熟算法迁移](research/decision_world_model_20261104/ASTRA_ALGORITHM_MIGRATION.md)、[显式时间与安全约束](research/decision_world_model_20261104/EXPLICIT_TIME_AND_SAFETY.md)。
- [时序方法理论](research/sota_temporal_20261105/THEORY.md)、[扩展理论](research/sota_temporal_20261105/THEORY_ADVANCED.md)。
- [SCET 审阅](research/scet_review_20261108/REVIEW.md)、[SCET-R 可恢复性设计](research/headroom_audit_20261108/SCET_RECOVERABILITY_DESIGN.md)。这些是研究提案，不能视为已验证的新方法或安全保证。
- [文献注册表及 BibTeX/RIS](research/headroom_audit_20261108/literature/)。检索范围、日期与核验限制保留在记录中。

## 归档说明

原文笔记按记录日期保留，不表示归档日所有公开代码、排行榜或论文发布状态均已重新核查。笔记中引用的本地实验附件可能只保留在原工作区；实验现状以复现仓库为准。

此处上传阅读笔记及自编指南，不上传原论文全文、抓取的全文缓存或完整译文。三篇已制作的完整译文仍保留本地。PDF 沿用原排版；没有为了上传重排或替换字体。

`EXPORT_PROVENANCE.json` 保存原文件来源与哈希，`SHA256SUMS` 用于检查发布快照。论文、参考文献和引用片段的权利归原作者；此仓库不代表论文作者的观点。
