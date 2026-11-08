# World Model 文献核验与迁移笔记

检索日/截止日：2026-11-03。主集合 14 篇，另保留 3 篇已读正文的补充记录（共 17 篇唯一 arXiv ID）。原始摘要页元数据保存在 `wm_sources/metadata.json`；逐条方法、正文定位、边界和最小实验见 `worldmodel_records.json`。这些是有界跨方向调研，不是穷尽系统综述，也没有本项目新增闭环实验。

## 结论与优先级

1. 先读 **Social-WM → CLWM → Objective Is the Bottleneck → Value Equivalence/TD-MPC2**。这条链分别回答安全可执行性、动作干预可分性、代价读出是否误导规划、哪些信息真正值得建模。前两种“新想法”已经有直接公开工作；仅组合 latent WM、逆动力学和反事实损失不能主张原创。
2. 再读 **MWM → DINO-WM → LeWM → One-Step Next-Latent Prediction**。MWM 已明确冻结主干只调动作 AdaLN；DINO-WM 提供冻结表征实践；LeWM 给低参数模型和防塌缩设计；One-Step 论文说明防塌缩与一步误差都不是多步控制正确性的充分条件。
3. **V-JEPA2/2.1、DreamerV3** 学范式，不优先从头训练。V-JEPA2.1 是当前检索到应补入的更新，但其导航表7为开放环轨迹评价，不等同社会导航成功/碰撞；大模型和蒸馏模型能力需分别核验。
4. 补充表保留 NavWM、One-Step NWM、Execution Gap：前二计算较大，后者适合执行层审计。并非无价值，当前优先级低于上面的因果/目标诊断。

## 问题状态矩阵

|问题|支持与已有方案|反证/剩余条件|截至检索日判定|
|---|---|---|---|
|预测更准是否动作更好|Value Equivalence；TD-MPC2 联合回报与表征目标|Objective Bottleneck 在 TwoRoom 展示表征含信息而距离不单调；单环境诊断不可直接泛化|已明确非充分关系，社会导航中的具体失败仍需实验|
|动作条件预测是否足够|NavThinker 基础上，CLWM 明确加入同锚点反事实难负例|CLWM 的因果措辞不代表结构可识别；需干预结果标签；未测大预训练编码器|通用想法已有，局部视觉的人群响应与风险尾部仍待证据|
|可执行动作是否等于安全|Social-WM 用逆动力学与命令残差筛选|训练 keep-out gate 是安全标签来源；连续动作不同；持续阻挡仍超时|条件性方案，不能当四动作无条件安全保证|
|只改少量WM权重是否新|MWM 已做动作 AdaLN 后训练|视觉一致性提高不保证行动排序/闭环安全；小参数不等于低数据成本|“只调动作层”已有，需更细故障与保留控制|
|一步MSE、防塌缩能否保证长时模型|LeWM SIGReg 避免常量解；One-Step 给条件均值/非线性反例|分布、历史可观测性和误差传播不能靠边缘统计消除|数学上需额外条件；不能笼统否定JEPA|
|更大视频模型是否适合即时社交导航|V-JEPA2.1 改善稠密特征/开放环规划|原规划10.6秒；V-JEPA2机械臂16秒/动作；不同GPU不能换算3090实时|优先特征probe与小头迁移，而非原样部署|

## 推荐的可证伪小实验（本次推断）

**实验 W1：排序比重建更重要吗？** 冻结基线表征、固定候选动作与规划预算，比较 latent L2、线性几何读出、进度/碰撞代价头、动作对比目标。拆开“真实下一状态编码”和“模型想象下一状态”，检查学习代价是否只在前者有效。基线必须包括原actor与外部logit/代价校正，不能仅与一个弱MSE头比较。真正继续条件是独立场景中排序与闭环联合成功均改善；不能靠早停或超时换少碰撞。

**实验 W2：动作分支到底学到了什么？** 在可精确恢复状态的模拟器中，对同一锚点分叉四动作，固定外生随机源，记录自车位姿、行人世界轨迹与风险标签。先补偿相机运动，再分别看“世界静态”“行人固定回放”“行人互惠响应”三组。CLWM式可分性是对照：若仅区分画面移动，不能叫交互建模。不要固定响应性行人的未来轨迹来做所谓因果控制，这会删除要研究的反应。

**实验 W3：怎样在保持常态行为下修复？** 由 W1/W2 确定真正错误层后，才比较动作调制层、未来特征读出与原actor头。保持同参数量/同范数/同数据预算，加入随机层和外部校正；只做AdaLN后训练与MWM高度重叠，只做反事实对比与CLWM高度重叠。可研究的更窄问题是风险标签校准、保留正常状态、跨行人反应策略迁移，但目前不宣称新颖。

## 数学上建议抓住的三个式子

- 价值等价关注 `T_model^pi v = T_true^pi v` 对指定 `(pi,v)` 集成立；把集合换成进度、碰撞、个人空间的价值基底是本次建议，不是已有安全定理。有限策略族不能控制任意分布外动作，期望值一致也不控制尾风险。
- 对线性噪声系统，K步误差协方差为 `Σ_K = sum_(j=0)^(K-1) A^j Σ (A^j)^T`。一步残差再小，长时随机性仍累积；非线性时 `f(f(z))` 一般不等于 `E[z_(t+2)|z_t=z]`。
- 区分候选排序损失与状态MSE，例如 `max(0, margin + c_hat(a_safe)-c_hat(a_risk))`。其中标签必须来自独立开发数据，且safe/risk为特定时域和阈值；该表达是实验设计示意，不是新算法声明。

## 检索与计数

详见 `worldmodel_queries.json`：5组共20个精确检索式；4组为同日重放以补记返回条目数。批次返回条目为40、25、34、21、21，总计141个搜索结果条目，包含重复、博客和非相关工作，**不能写成141篇候选论文**。每组工具未给全网总数或逐query独立数。主集合14篇+补充3篇，是正文核验后的保留条目数；不是这141条的完整PRISMA排除统计。窗口优先2023–2026，Value Equivalence 2020为经典锚点。

arXiv 插件查询（日期截至2026-11-03，sort=date，max_results=20）：

- `ti:"world model" AND (abs:navigation OR abs:social)`
- `ti:"DINO-WM" OR ti:"TD-MPC2" OR ti:"Mastering Diverse Domains"`

两条均返回 HTTP 429 / rate_limited，无有效论文结果，转用官方 arXiv HTML、摘要元数据、NeurIPS、作者项目页和 Frontiers 原文。发现阶段网页只用于线索，主记录均至少核验相关方法正文。后向沿 Social-WM 到 LeWM、DINO-WM，沿 CLWM 到价值等价；前向通过标题/失败查询查到 MWM、V-JEPA2.1、CLWM、Objective Bottleneck。没有完整引用图谱，不能声称所有后续工作已覆盖。

## 已筛但未纳入主表

- **NavWM、One-Step NWM、Execution Gap**：均已读相关正文与元数据，另置补充JSON；优先级/算力原因，不是证据不合格。
- **DreamerNav**：已读 Frontiers §3.5、§4.2；使用深度、动态占据图和全局A*路径，不能与纯局部RGB同信息直接比较。作者报告4090上24.79h，非当前一小时首轮任务。正式出处 DOI `10.3389/frobt.2025.1655171`，2025-09-26。保留为邻域线索而非计入主14。
- **Hidden Failure Modes in Latent World-Model Planning from Offline Data**：官方OpenReview检索可发现身份与节选，但打开PDF遇浏览器验证，全文未通过；列 discovery_only，不依赖其实验数字支持论断。URL：https://openreview.net/pdf/2970593988c2eb8f2bd6612fdaa898f0dbd7b107.pdf
- **World-Model-Grounded LLM Planning for AUV and ASV Navigation Near Offshore Wind Farms**（2608.19661）：动力学邻域，未读正文、不适配当前社会视觉四动作主线，排除。
- 最后一轮也发现潜在相关的 **Correcting a learned physical invariant improves world-model rollouts**（2608.23526）、**Beyond Euclidean Proximity**、**WaveGo**、**One-Step...**等。只把其中已转官方正文核验的 One-Step 纳入；其余不补造元数据，不拿二手总结作证据。若进一步研究守恒latent修正/自适应执行，应优先追查这些重叠。

## 访问失败、元数据与覆盖边界

- arXiv插件429：损失结构化按日期检索/导出；用官方摘要HTML提取作者和日期。17份摘要页与JSON在wm_sources可追溯。
- Nature DreamerV3全文及PDF跳转身份服务，工具返回500/non-retryable；正式版书目信息由官方搜索记录核验，方法依据arXiv v2，两个版本不重复计数。
- PMC DreamerNav触发浏览器检查；Frontiers出版社全文可读取，未使用绕过机制。
- OpenReview Hidden Failure PDF触发verification；降级发现线索，不声称读完。
- V-JEPA2摘要页作者元数据将“Mojtaba Komeili”错误拆为两条，且“Mido/Mahmoud Assran”为摘要/正文名差异；作者列表以正文名单为准并在记录注记。
- DINO-WM、Social-WM、LeWM、NWM、V-JEPA2.1、CLWM 的最新摘要修订日已记录；对应最新v2/v3方法或关键实验重新核验。标题页出现自动编译日期不作为首发日期；首发日期取citation_date。
- 检查官方版本/摘要页未见撤稿标识，并执行针对NWM/DINO的反向更正搜索；未进行Crossref/RetractionWatch全库审计，不写“保证无更正”。
- 代码/权重仅记录正文或项目链接，不等于成功安装、可用权重已下载或已独立复现；GPU耗时只在原文硬件和任务定义下引用。
