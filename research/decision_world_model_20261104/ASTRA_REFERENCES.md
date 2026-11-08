# Astra 数学建模：引文与本地证据核验记录

核验日期：2026-11-04。仅记录本轮实际访问的原始来源。网页身份核验不等于全文逐式审查。主报告内所有小命题/反例均为针对本项目的显式推导，不以引用替代证明。

| ID | 身份（作者/年份/标题） | 原始链接 | 本轮阅读范围及用途 |
|---|---|---|---|
| R1 | Leslie Pack Kaelbling, Michael L. Littman, Anthony R. Cassandra; 1998; Planning and Acting in Partially Observable Stochastic Domains; Artificial Intelligence 101:99–134 | https://www.cassandra.org/arc/papers/aij98.pdf ; https://www.sciencedirect.com/science/article/pii/S000437029800023X | 作者PDF检索返回及出版页身份/摘要；经典POMDP起点，不提供Falcon表征充分性 |
| R2 | Robert Givan, Thomas Dean, Matthew Greig; 2003; Equivalence Notions and Model Minimization in Markov Decision Processes; Artificial Intelligence 147:163–223 | https://cs.brown.edu/people/tdean/publications/archive/GivanetalAIJ-03.pdf | 作者原文搜索返回作者/题名/摘要；双模拟与模型最小化概念，未复核全部定理 |
| R3 | Christopher Grimm, André Barreto, Satinder Singh, David Silver; NeurIPS 2020; The Value Equivalence Principle for Model-Based Reinforcement Learning | https://arxiv.org/abs/2011.03506 | 原始摘要/作者/日期；价值等价针对指定策略与函数集的Bellman更新 |
| R4 | André Barreto, Will Dabney, Rémi Munos, Jonathan J. Hunt, Tom Schaul, Hado van Hasselt, David Silver; NeurIPS 2017; Successor Features for Transfer in Reinforcement Learning | https://arxiv.org/abs/1606.05312 | 原始摘要/作者；预印本2016，修订2018；线性奖励、固定动力学与策略的价值分解 |
| R5 | Nicklas Hansen, Hao Su, Xiaolong Wang; ICLR 2024; TD-MPC2: Scalable, Robust World Models for Continuous Control | https://arxiv.org/abs/2310.16828 | 原始摘要/作者/会议；decoder-free latent planning的先例，不引用结果为本项目保证 |
| R6 | Zeying Gong, Tianshuai Hu, Ronghe Qiu, Junwei Liang; ICRA 2025; From Cognition to Precognition: A Future-Aware Framework for Social Navigation | https://arxiv.org/abs/2409.13244 ; https://arxiv.org/html/2409.13244v2 | 身份、全文任务/指标相关段落；PointGoal、深度、社会导航、PSC阈值。执行行为仍以本地实现为准 |
| R7 | Zhe Huang, Tianchen Ji, Heling Zhang, Fatemeh Cheraghi Pouria, Katherine Driggs-Campbell, Roy Dong; WAFR 2024（预印本2025）; Interaction-aware Conformal Prediction for Crowd Navigation | https://arxiv.org/abs/2502.06221 ; https://arxiv.org/html/2502.06221v1 | 身份、摘要及HTML检索；决策相关人体不确定性与校准框架。未把其安全结果移植到本模型 |
| R8 | Michael Janner, Justin Fu, Marvin Zhang, Sergey Levine; NeurIPS 2019; When to Trust Your Model: Model-Based Policy Optimization | https://papers.nips.cc/paper_files/paper/2019/hash/5faf461eff3099671ad63c6f3f094f7f-Abstract.html ; https://arxiv.org/abs/1906.08253 | 原始会议页搜索返回与预印本记录；短rollout与模型偏差动机，不套用单调改进保证 |
| R9 | Stephane Ross, Geoffrey Gordon, Drew Bagnell; AISTATS 2011; A Reduction of Imitation Learning and Structured Prediction to No-Regret Online Learning | https://proceedings.mlr.press/v15/ross11a.html | 原始会议页作者/题名/摘要；闭环分布变化，当前未实现DAgger |
| R10 | Tianshuai Hu et al.; 2026; NavThinker: Action-Conditioned World Models for Coupled Prediction and Planning in Social Navigation | https://arxiv.org/abs/2603.15359 | v2 2026-03-18身份及摘要机制；与on-policy训练耦合，非本项目冻结外挂复现 |
| R11 | Zhihao Zheng, Mooi Choo Chuah; 2026; Social-WM: Safety-Aware Latent World Models for Robot Social Navigation | https://arxiv.org/abs/2609.40177 | v2 2026-10-01身份及摘要机制；可执行动作与潜预测，未复核全部实验 |

## 本轮本地只读核验

- `research/decision_world_model_20261104/DESIGN.md`、`proposal.json`、`transition_contract.json`：完整读取。设计参数、未解决项、允许输入、禁止推理字段、终止/删失及独立数据约束。
- `weightlab/README.md`：读开头65行。当前仅操作者驱动CPU头回放，不是自动LLM或世界模型闭环。
- `baselines/repos/Falcon/falcon/additional_metric.py:81–108`：HumanCollision累计接触并终止；154–191附近PSC使用距离配置且阈值1m。
- `baselines/repos/Falcon/habitat-baselines/habitat_baselines/config/social_nav_v2/falcon_hm3d.yaml:54` 起：六个人体动作类型为OracleNavRandCoordAction_Obstacle。
- `baselines/repos/Falcon/falcon/additional_action.py:497,602,761,946,1091`：动作继承、人体避障循环排除agent0、可选机器人距离响应分支及默认关闭值。证据为静态代码；未运行最终配置解析与动态反事实回放。

## 历史记忆使用

做了轻量关键词定位，MEMORY.md:83–93用于提醒任务可比性和“局部验证不等于完整复现”的边界。本报告的工程状态随后从当前设计及README核对，不将旧记忆中的评测数字作为当前结果。未修改任何记忆文件。
