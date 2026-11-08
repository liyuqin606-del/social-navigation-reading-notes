# 数学、控制与风险路线调研

截止：2026-11-03。对象：Falcon四动作社会导航、NavThinker未来表征、DPed-VLN动态行人任务；目标是可迁移机制，不是直接合并不同平台的排行榜数字。重点2024–2026，保留2019 CBF作为基础锚点。

交付：`math_records.json`含11篇独立论文（10篇arXiv身份+正文；1篇PMLR正式版本）。其中1篇基础锚点、10篇直接/相邻研究。11篇均已核对完整作者元数据与至少相关正文段落。作者名保留arXiv原始“姓, 名”格式。源文本、元数据和PMLR PDF保存在`math_sources/`。未独立验证作者所有实验数字，未复现实验，也未宣称覆盖整个领域。

## 结论先行

最值得立即读的是M01 ECP、M03 AECP、M04 ICP、M08 DR-MPC、M10 BIG-CBF。前三篇表明“轨迹不确定性+CVaR+社会导航”已经很拥挤；后两篇说明“控制器混合/想象+安全过滤”也不是空白。可探索的问题应缩小为：**已有视觉四动作策略发生局部修复后，如何以有限反馈预算重新校准危险动作，并在正常状态保持行为、在动态障碍环境避免冻结。**这是调研推断，尚未证明该精确问题具有论文级新颖性。

|编号|原文|可借鉴机制|主要阻力|
|---|---|---|---|
|M01|[Egocentric CP](https://arxiv.org/html/2504.00447v1)|按照候选动作造成的距离风险校准，有限动作序列MPC|决策相关分数自身已有先例；长序列组合爆炸|
|M02|[GenSafeNav](https://arxiv.org/html/2508.05634v1)|不确定性输入+预测侵入代价+约束RL|需策略重训，观测/环境不等同视觉Habitat|
|M03|[AECP](https://arxiv.org/html/2610.07474v1)|残差椭球形状与尺度分离，CVaR乘子|已有强重叠；部署半径截断会破坏名义覆盖|
|M04|[ICP](https://arxiv.org/html/2502.06221v1)|校准分布随机器人计划变化|依赖交互模拟器；交换性与有限样本分位数仍要核查|
|M05|[ACP-SBC](https://arxiv.org/html/2407.03569v2)|校准屏障表达式误差，不只是位置误差|动力学/正则性/离散执行之间有假设缺口|
|M06|[DRCC-MPC](https://arxiv.org/html/2403.05081v1)|机会约束经分布鲁棒CVaR保守近似|真分布是否在歧义集合内不能假设成立|
|M07|[Safe POMDP ACP](https://arxiv.org/html/2404.15557v2)|动态安全shield与信念状态规划|粒子信念近似可损害保证，搜索成本增加|
|M08|[DR-MPC](https://arxiv.org/html/2410.10646v2)|候选避人动作与路径跟踪控制混合|连续动作混合不能直接混合离散动作标签|
|M09|[CBF基础](https://arxiv.org/html/1903.11199v1)|安全集合不变性和最小干预|安全不等于可到达；需初始安全与可行输入|
|M10|[BIG-CBF](https://arxiv.org/html/2609.14343v1)|行为想象与执行共享不确定性|全向连续机器人与视觉离散动作差别大|
|M11|[生成先验+鲁棒CP](https://proceedings.mlr.press/v331/rahaman26a.html)|按环境条件生成校准数据，同时约束生成失配|不能用世界模型自己生成的未来无条件自证安全|

## 数学上的三处易错点

1. **覆盖对象。**单个行人单步90%覆盖不推出整回合90%不碰撞。多人、多步需要联合事件定义与风险分配；同回合帧强相关，随机帧切分会泄露。自适应CP的时间平均失覆盖控制也不等于每个时刻/每个条件状态的覆盖保证。M03正文明确承认部署截断后的欠覆盖。M07正文明确承认近似信念可能损害保证。
2. **闭环分布。**策略改变后，访问状态、人类反应和预测误差可能一起改变。M04已经处理机器人计划依赖的校准，不能把“策略依赖”简单重提为首创。M11需要可观测环境参数和受控生成分布失配；它将交互式、非平稳分布移位列为后续方向。合成残差可补数据，但不能替代真实held-out校准。
3. **安全与进展。**CVaR罚项不是硬约束；CBF不变性不是到达保证；只把STOP概率提高可能更早终止回合。对Falcon应逐个检查已有动作及整个执行扫掠路径；无安全动作要如实计失败，不能悄悄加入WAIT或后退。M10的六种行为也不是Falcon已有四动作，迁移必须重定义行为实现。

## 问题状态矩阵

|问题|支持证据|已有解决/反证|剩余条件|判断|
|---|---|---|---|---|
|人群OOD时点预测不可靠|M02、M03|DtACI、椭球与风险约束已显著研究|视觉漏检、任意静态障碍、截断后欠覆盖、交互移位|部分解决，不能以“加入不确定性”作为新颖点|
|轨迹误差降低不保证安全动作排序|M01、M05|ECP及屏障误差校准直接针对控制相关误差|视觉潜变量到真实几何风险、稀有危险状态、决策后分布|条件性解决，可针对具体失配做实验|
|编辑策略后校准失效|M04、M11提供分布与动作依赖理由|计划依赖校准与生成条件校准已有方案|少反馈、不可见行人、四动作原策略保留等组合未在本次证据中直接验证|候选问题，未证明全领域空白|
|安全过滤导致冻结|M09说明安全/活性分离，M10明确处理停滞|行为想象、混合控制已经实现|Falcon动作受限且STOP终止；噪声风险模型可无可行动作|条件性解决，必须同时报超时/任务进展|
|世界模型可自我生成安全证明|M11明确处理生成模型偏差|鲁棒CP可在误差/参数假设下恢复保证|真实分布与生成分布偏差难验证；非平稳交互|不能无条件成立|

## 推荐最小验证顺序（研究建议，未执行）

第一步从已获许可的Falcon开发回合记录四动作logit、隐藏特征、失败前短历史；不用测试集选择编辑阈值。先判断错误发生在感知/未来预测还是读出。所有额外真状态仅作为oracle诊断，单列结果。

第二步冻结骨干，设一个短时域风险读出或已有预测器；比较恒速预测+常数安全距离、欧氏误差分数、M01决策相关分数、M03形状校准。保持预测器/反馈样本/动作枚举次数相同。记录覆盖、碰撞、最小间距、SR/SPL、超时、干预率和延迟。

第三步只在有闭环价值时拟合最小动作头修复。对照外部logit修正、bias-only、ridge全头、同范数随机改动、同预算微调。再比较原策略校准、编辑后重校准、动作分层校准。风险降低若完全由停滞或提前STOP造成，否定导航改进。

M08完整重训与M02/M03约束RL为较高投入基线；M11小型残差条件模型可后置，优先与经验重采样/条件高斯比较。没有证据支持先训练大型视频世界模型。

## 检索记录与完整性边界

所有查询在2026-11-03执行，未设置截止日之后条目入选。使用web搜索发现线索，最终证据来自arXiv官方HTML/摘要页、IEEE论文入口、作者实验室页、PMLR正式论文PDF。arXiv官网metadata通过HTTP读取citation字段，版本从Submission history核查。当前入口未见撤稿提示，但没有穷尽独立争议检索。

|批次|完整查询|来源|返回/处理说明|
|---|---|---|---|
|Q1|`conformal prediction safe robot navigation adaptive conformal 2024 2025`; `social navigation distributionally robust model predictive control CVaR 2024 2025`; `social navigation control barrier functions prediction 2025 2026`; `decision focused world model value equivalence safe reinforcement learning navigation`|web搜索|接口合并各查询结果，不提供数据库总命中数或逐查询数量；输出过长，不能据此声明完整候选数|
|Q2|`"Risk-Sensitive Crowd Navigation with Adaptive Ellipsoidal Conformal Prediction"`; `"Interaction-aware Conformal Prediction"`; `"Integrating Predictive Motion Uncertainties"`|web搜索|3个唯一目标全部核验入选；聚合站/自动AI审稿仅作发现，未作为证据|
|Q3|`"DR-MPC" arxiv Han`; `site.prismmodelchecker.org/papers/ral-conformal.pdf`; `"Predictive Control With Adaptive Conformal Prediction"`|web搜索|DR-MPC与POMDP核验入选；同名腿式DR-MPC排除|
|Q4|`"Safe POMDP Online Planning" arxiv`; `"Control Barrier Functions: Theory and Applications" Ames Xu Grizzle Tabuada`; `"Planning With Conformal Prediction" safe robot 2023 Lindemann`|web搜索|沿后向/相邻引用发现PMLR 2026生成先验论文，正文核验入选；作者名以官方页面纠正，不使用查询词推断作者|
|Q5|`https://arxiv.org/abs/2609.14343`及对应HTML|另一研究分支提供线索，官方原文核验|BIG-CBF全文方法段核验后入选1篇；不使用COMPASS转述作为证据|
|Q6|10个入选arXiv ID的`/abs/{id}`及`/html/{id}`；PMLR `/v331/rahaman26a.html`与其PDF|官方HTTP全文/metadata|10/10 arXiv成功；1/1 PMLR PDF成功；共11篇独立证据记录|

未入选/待后续候选：`Time-aware Motion Planning in Dynamic Environments with Conformal Prediction`（2511.18170，未做正文核验）；`Heterogeneous Predictor-based Risk-Aware Planning...`（2507.11920，未做正文核验）；`Safety-Critical Control under Uncertainty via Adaptive Conformal Quantile Prediction Intervals`（2609.23280，仅发现）；`Safe POMDP Online Planning via Shielding`（2309.10216，已有同团队更新动态ACP版本，未重复占证据条目）；`DR-MPC: Fast and Feasible Dynamics-Relaxed...`（2609.20035，同名腿式控制，非本任务核心）；手部协作机器人UA-PCBF（场景不直接）；TU Delft硕士论文Robust CP adaptive motion planning（未做全文核验）。这些候选不计入11篇证据集。

访问失败：批量arXiv插件`get_abstract`/`download_paper`调用长时间未完成后主动终止，未获得明确错误码，不能断言429；改从官方HTTP取全文/元数据成功。web打开PMLR GitHub raw PDF返回Unsupported content-type application/octet-stream，随后直接下载同一官方PDF并用PyMuPDF读取23页成功。检索接口未报告数据库总结果数，因此不能声称系统综述式召回完整性。

## 对综合报告的独立公式QA

6.1闭式ridge正确，但需明确lambda>=0、eta>0；eta>0单独不足以约束负lambda。P Hp=0对应投影到null(Hp^T)，当Hp满行秩仅零空间。6.3的CVaR表达需0<beta<1及损失可积；离散CBF建议改用kappa∈(0,1]，写初始安全并将动态行人状态并入状态或使用b_t。预测屏障满足不等于现实扫掠路径安全。C路线还应区分长期平均失覆盖与每时刻条件覆盖。上述是数学审查，不是新的实验或新定理。
