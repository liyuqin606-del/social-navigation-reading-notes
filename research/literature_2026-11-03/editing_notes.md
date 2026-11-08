# 权重编辑检索记录与证据边界

截止 2026-11-03。目标：把权重空间方法转成可检验的视觉社会导航局部修复；未运行模型或实验。

## 查询记录

来源均为 web 搜索及返回的原始论文网页。搜索返回的是混合摘要，不是数据库完整结果集，无法可靠恢复各查询总命中数；因此总命中数记为 unavailable，以下只报告实际进入核验的候选数，不制造筛选漏斗。

|查询组（完整检索式）|核验候选数|处理|
|---|---:|---|
|`"AlphaEdit" null-space constrained knowledge editing paper`; `"Behavior Uncloning" MoRE policy 2026`; `"What do VLM-Based Vision-Language Navigation Models Rely on"`|5|AlphaEdit、复现研究、MoRE、VLN steering 纳入；FedAlphaEdit仅发现，联邦合并非当前单策略优先范围|
|`"Locating and Editing Factual Associations in GPT" paper`; `"ReFT: Representation Finetuning for Language Models" authors`|2|两篇均回到正式论文/作者全文，纳入|
|`"LoRA: Low-Rank Adaptation of Large Language Models" arxiv`|1|纳入为必须比较的便宜适配基线|
|直接打开前序审查种子 `https://arxiv.org/html/2411.04408v1`|1|SARP 纳入，正文 §III–V 核验|
|`"From Cognition to Precognition" arxiv`; `"NavThinker" arxiv`; `"DPed-VLN" arxiv`|3 种子|元数据与项目已有用户全文对齐，不计新相关工作|

以上本线明确跟踪 9 个扩展候选，8 个纳入、1 个只作发现线索。它不是整场搜索所有命中量。后向追踪阅读 ROME/ReFT/AlphaEdit 的方法与参考文献；前向通过 2026 AlphaEdit复现、MoRE和VLN steering补足。未取得完整引文图谱，不声称穷尽所有前向引用。

## 访问失败（均为 2026-11-03）

- arXiv 插件 `export_citations` 批量请求 2202.05262、2410.02355、2606.26783、2411.04408v1、2606.29201v1、2609.24576v1、2404.03592、2106.09685：HTTP 429，返回 retry_after_seconds=60。未将失败返回视为成功导出，改用官方原文作者栏与正式会议页面。
- OpenReview `https://openreview.net/pdf?id=HvSytvg3Jh` 返回浏览器验证页；未绕过验证。
- ICLR AlphaEdit 正式 PDF 返回工具容量限制（12,824,655 bytes）；通过 arXiv v4 全文核验方法、作者变更，会议身份从 ICLR 官方结果核对。

## 版本冲突

- AlphaEdit：搜索初版少 Jie Shi，正式版及 arXiv v4 有该作者；采用后者8人列表，不拼混作者。
- MoRE：arXiv HTML头标 v1 / 2026-06-28，但排版正文写 August 24, 2026；两个字段并存，不推断不存在的 v2。
- LoRA：这里引用已读 arXiv v2/2021；不为凑正式出版信息擅自写未核验的会议信息。
- 全部原始页面未见撤稿标注；未进行完整出版后勘误、争议数据库审计。不得写为“无任何争议/撤稿”。

## 本线问题状态

1. 导航策略能否永久修复：`conditionally_solved`，SARP/MoRE已有实例；视觉四动作与小预算跨场景保留仍需验证。
2. 少量参数能否保证行为局部：`insufficient_evidence`，ROME/AlphaEdit只提供其条件下机制；有限保留矩阵不覆盖闭环新分布。
3. VLN能否用内部表征引导恢复：`conditionally_solved`，E06有证据，但实机人工触发不可改称自动恢复。
4. 无梯度编辑是否独立优于便宜优化：`insufficient_evidence`，本项目尚无闭环数据；闭式解、随机搜索与LLM搜索必须同反馈预算。

所有“transfer/pilot”字段都是本次分析提出的迁移假设，不是被引用论文已验证的导航结论。
