# 社会导航优化：数学、物理与 World Model 文献调研

本次整理 47 篇去重相关工作，另列 3 篇种子论文；相关正文均已定位阅读，不等于全部论文逐字精读。

检索截止：2026-11-03。对象：Falcon、NavThinker、DPet / DPed-VLN。目标：找到能够改善决策、风险和跨场景泛化的具体机制，并适配当前有限算力。文中实验方案是待验证假设；没有新增导航跑分，也没有把论文声称的优势当作本项目已复现结果。

## 1. 先给判断

最值得推进的主线是：**诊断世界模型在哪些状态会误导动作，把这些错误转化为风险或保留约束，再用小规模策略修复验证闭环收益。** 用物理与几何区分视角运动、真实人群运动、相互反应及左右绕行的稳定性；用统计工具检查预测是否可信；用线性代数控制编辑对正常状态的影响。

这是一条研究组织思路，尚不是已确认新颖的算法。单独的“World Model + 社会导航”“共形预测 + CVaR”“Hamiltonian + 人群运动”“永久修改策略行为”均已有直接先例。当前更有价值的问题是：这些方法进入**局部视觉观测、四动作、动态行人和固定小反馈预算**后，哪些假设失效，能否以可解释且便宜的改动修复。

优先顺序分成两种，避免把可实现性当成创新性。工程试验优先 Falcon 的动作头，因为有清晰推理路径；科学问题优先动作相关的预测错误及跨域失效，因为单纯低秩编辑本身很难构成贡献。NavThinker 的未来特征读出最贴近第二个问题，但需先补齐匹配论文的检查点与配置。

## 2. 三篇原文实际暴露了什么

|对象|已核验现象|由此提出的问题，不是已证实原因|
|---|---|---|
|Falcon|轨迹预测主要承担训练辅助作用；当前公开 actor 路径不调用独立辅助预测头|特征中是否已有避让信息，却在动作读出阶段没有正确利用？|
|NavThinker|论文表 I：Social-HM3D SR 59.46，对照 Falcon 56.26；Social-MP3D SR 47.33，反低于 Falcon 51.67|收益依赖训练分布吗？预测失准、保守决策、场景感知或配置差异分别贡献多少？|
|DPet|表 6 去掉 SCP：SR 38.55→36.59，HCR 36.70→30.29|奖励提升任务完成时是否增加碰撞暴露？需要约束和联合成功指标，而非继续盲调惩罚系数|
|DPed-VLN|普通指令与含行人线索指令的可用信息不同；动态行人造成明显退化|真实在线信息、离线轨迹先验和错误先验应如何分开？|

来源：[NavThinker 表 I/III](https://arxiv.org/html/2603.15359v2)、[DPed-VLN 表 4/6](https://arxiv.org/html/2609.21504v1)、[Falcon 固定版本推理代码](https://github.com/Zeying-Gong/Falcon/blob/d15ef512c4b1451f9daeaf6d4d7ecc702ed57430/habitat-baselines/habitat_baselines/rl/ppo/policy.py)。这些数字属于各自论文，不能横跨任务直接排名。差异也没有在本项目中做过多种子显著性验证。

本地材料审查还发现：NavThinker 默认配置与论文架构有差异，且 policy/world-model checkpoints 待发布；DPet 正式仓库尚未提供完整实现和匹配检查点。实验候选可以先设计，但不能把任选开发配置训练称为论文复现。

## 3. 文献地图应该怎样读

|路线|真正要学的工具|它解决的具体问题|最容易误用的地方|
|---|---|---|---|
|数学与控制|约束 MDP、MPC、CVaR、控制屏障、共形校准|把“尽量安全”变为可检查的风险目标；处理尾部失败|将逐步平均风险当回合碰撞率；把有限样本覆盖当闭环永久保证|
|物理、几何、交互|速度障碍、社会力、相对运动、博弈、同调路径类、Koopman|区分运动学限制、让行意图与左右绕行；减少震荡和死锁|把人当守恒系统；忽略他人是否会对机器人反应|
|World Model|潜在转移、价值等价、短视滚动、动作条件预测、逆动力学|优先预测影响动作排序和碰撞的未来，而非所有像素|视觉预测更漂亮却更会“利用模型错误”；盲目拉长想象时域|
|参数与表征编辑|干预定位、保留协方差、零空间、低秩、凸二次优化|少量样本修复局部读出错误，并量化对正常状态的扰动|少参数等于局部行为；只改辅助分支；不比较等价 logit 修正|

各路线都需要相同底座：POMDP 中的历史/信念状态、动作实际执行语义、世界坐标与机器人坐标、闭环分布偏移，以及公平的配对评估。

## 4. 必须先排查的直接竞争

**Social-WM** 已把名义动作和实际可执行动作的差异用于候选筛选；使用冻结视觉表征和潜在预测。它比一般视频生成论文更接近我们的任务。但正文采用连续三维动作，采集数据带安全限制，必须审计这些设定及标签来源，不能将其与四动作方法直接做数字排名。[原文 §III–IV](https://arxiv.org/html/2609.40177v2)

**Risk-Sensitive Crowd Navigation with Adaptive Ellipsoidal Conformal Prediction** 已将自适应椭球预测与尾部风险控制用于人群导航。因此，提出“把不确定性交给 CVaR”只能算已有方法迁移；需进一步证明视觉部分可观测、策略变化后校准失效等具体缺口。[原文](https://arxiv.org/html/2610.07474v1)

**SARP、MoRE、VLN steering** 分别覆盖预测器辅助安全参数修复、行为模式写入策略权重、以及导航激活干预。差异不能只写成“我们编辑权重”；应落到反馈预算、自动触发、保留约束和分布外闭环效果。VLN steering 的实机恢复包含人工触发，需要与自动系统分开。[SARP §IV](https://arxiv.org/html/2411.04408v1)、[MoRE §3](https://arxiv.org/html/2606.29201v1)、[VLN steering §V-I](https://arxiv.org/html/2609.24576v1)

**NaviDIFF、SHINE、DenseKoopman** 已分别覆盖物理结构的人群规划、绕行类别与决策一致性、以及 Koopman 行人预测。这些适合作为有依据的对照和模块，不适合仅换名字后声称原创。它们所依赖的人群可见性、输入真值、连续动作和预测数据集需要逐项对齐。详见后附文献条目的正文定位。

还应把三篇新近反证放进必读组。**CLWM** 已研究部分可观测下的信念状态、反事实动作可分性；不能再把“区分不同动作的未来”单独当新贡献。**The Objective Is the Bottleneck** 在 TwoRoom 的具体案例中发现规划代价而非预测器是主要瓶颈，并报告学习代价在想象分布漂移下的失败；其结论不能直接外推到社会导航。**One-Step Next-Latent Prediction Is Not a World Model** 从条件均值与转移分布的区别解释为何一步误差不足以支撑滚动。它不证明所有潜在世界模型无效。[CLWM v2](https://arxiv.org/html/2609.05834v2)、[Objective §4–9](https://arxiv.org/html/2608.12959v1)、[One-Step §3](https://arxiv.org/html/2609.36227v1)

## 5. 六条候选路线

### A. 有保留约束的动作头修复：最先做的可证伪试验

先在 Falcon 开发集找一种频繁故障，例如左右震荡、未到目标便 STOP、可避让时持续前进。只在故障发生前仍有正确选择的状态收集编辑样本，配上外观相近但应保留原行为的正常样本。

当前代码推导 actor 是 4×512 权重加 4 维偏置，共 2052 个标量；必须以实际 checkpoint 和动作映射为准。冻结表征后，问题可以降为小型线性代数/凸优化。计算头部更新可能很轻，但获取闭环数据、图像编码、仿真渲染仍有成本。

**候选增量**是小预算下的故障定位、保留集构造与跨场景可靠性，而非秩一形式本身。必须对照 bias-only、全头 ridge/微调、同范数随机编辑、相同功能的外部 logit 修正。不能靠提高 STOP 来模拟等待；当前 STOP 会结束回合。

否定条件：特征不可分、错误在感知而非读出、正常成功明显受损、闭环不改善，或不优于便宜校准。此时停止扩大编辑，先检查实现再调整研究假设。

### B. 按动作后果校准世界模型：最值得深入的科学问题

不只问“下一帧预测准不准”，而问“预测是否把危险动作排在了安全动作前”。比较相同状态下各候选动作的短时风险、可通行性和任务进展。将模型误差分为不改变动作排序的误差、会颠倒排序的误差，以及对稀少危险状态的误差。

首轮冻结视觉主干，只训练小风险头/价值差头，或对 NavThinker 的未来特征融合做校准。先保留一步和短时域，不先训练大视频生成器。参考 Value Equivalence、TD-MPC2、DINO-WM 与新近导航 World Model；Social-WM 已覆盖可执行性，因此不能把逆动力学差异直接重提为新点。

同样，“代价比预测重要”与“动作反事实可分性重要”也分别已有 Objective 和 CLWM 的证据。这里应首先验证这些已知失败机制在本项目是否出现，再讨论社会交互、视觉输入和小预算约束下是否还有方法上的增量。

候选研究问题：**在按预测误差分层或匹配、固定数据与候选计算预算下，对动作排序有针对性的校准能否更好保持未见场景的成功-安全权衡？** 这是待查证假设。若只有预测指标变化、排序不改善，或排序改善无法转成闭环收益，就否定该机制链。

### C. 策略变化后的风险校准：数学价值高，先限定保证范围

采用多模态/椭球预测集合或风险分数，校准其覆盖，再让四动作决策考虑高风险尾部。关键并非简单加阈值，而是编辑策略后访问了新状态，原有校准样本不再同分布。可研究保留校准与在线再校准的成本及失败情况。

同时覆盖多人、多时间步的风险集合，与单行人单步覆盖是不同对象。校准单位优先完整轨迹/回合，避免把同一回合中高度相关的帧随机分到训练与校准两侧。对闭环策略与自适应预测，仅在相应论文假设下讨论覆盖；不承诺普遍有限样本安全。自适应共形预测的长期平均失覆盖控制，也不是每时刻的条件安全保证。

必须对照常数安全距离、恒速预测、固定风险阈值和已有动作依赖共形预测。若所谓收益仅来自停滞/超时或更早结束回合，就不能称导航更好。

### D. 把自车视角变化与行人反应拆开：物理与因果的交点

机器人转向时，即使行人静止，图像与机器人局部坐标也会大幅变化。因此“动作条件预测对动作敏感”不等于学到了人对机器人的反应。先做几何补偿，再检查真实行人运动残差是否依赖机器人动作。

构造同一场景状态和相同外生随机数下的配对动作干预，比较：固定行人回放、对机器人不可见的行人、会互惠避让的行人、非合作/延迟反应行人。交互情形应固定随机源而非强制固定行人未来轨迹，否则恰好删除了要研究的反应。

CLWM 已用同锚点动作分支训练，但这不等于识别了结构因果模型；本路线还需单独识别自车运动补偿后的人响应残差与行人策略变化。

仿真真值可用于诊断或监督标签，部署模型仍须只用与基线一致的视觉/自身位姿信息。若补偿后动作差异几乎消失，则世界模型可能主要学会了视角变化；若只在 ORCA 生效，则需要说明其行为模型依赖，而不是宣称真实人机因果理解。

### E. 保持绕行决策一致：低成本但重叠明显

需要检验的一类失败是每一步重新选择左右绕行，导致反复横跳；目前尚无本项目日志证明它是主要失败来源。先用相对位置、短轨迹或估计路径类别识别切换，增加有限滞回或一致性成本；必须允许原路径被堵时及时换边。

SHINE 已明确研究路径类别和决策一致性；2026 年 10 月的 COMPASS 预印本又提出通过侧承诺与安全优先释放，所以“模式记忆 + 风险解除”本身也不新。COMPASS 尚未同行评审，当前结果是离线决策核心，不能视作已证明闭环碰撞改善。可检验的剩余问题是：在无地图、局部视觉和冻结循环策略中，能否用已有隐藏状态实现同等机制；若加入外部记忆/模块，要如实记录系统变化。[COMPASS 正文](https://www.preprints.org/manuscript/202610.0120)

与固定转向偏置、前一动作平滑、固定保持时长、同调类规划、没有行人的对照场景比较。若只是始终偏左、碰巧匹配特定布局，或降低抖动同时增加死锁，则否定泛化解释。

### F. 语言先验与当前观测冲突：DPed-VLN 后续路线

将同一回合指令分为普通目标、正确行人线索、过期线索、左右对调线索、无关行人线索，控制长度和表达风格。研究语言对风险预测或未来特征的影响是否能随观测冲突而减弱。

这一路线依赖 DPet 正式实现或选定可复现的 VLM 基线。先做信息条件审计，不把离线专家轨迹形成的先验当成部署时天然可用信息。世界模型无法自动补上指令生成的信息泄漏，也不能用更强输入掩盖较弱决策。

## 6. 四个可计算的数学抓手

以下为本次分析提出的简化形式，旨在帮助设计验证，不是对任一论文公式的照抄，也不是新定理声明。

### 6.1 固定隐藏状态下的保留约束

设原 actor 为 z = W h + b。Hf 的列是需修复状态，Hp 的列是需保留状态，R 是希望在 Hf 上施加的 logit 残差。可先求解：

`min_D ||D Hf - R||_F^2 + lambda ||D Hp||_F^2 + eta ||D||_F^2`，其中 lambda >= 0、eta > 0。

唯一闭式解为：

`D = R Hf^T (Hf Hf^T + lambda Hp Hp^T + eta I)^(-1)`。

实现用线性方程求解，不显式求逆。R 必须来自已定义的安全动作/进展目标，不能事后用终评答案挑出来；也可把明确动作间 margin 写成凸二次规划约束。拟合只对缓存的 h 成立，闭环后状态会变，必须重新验证。

若 P 投影到 null(Hp^T)，使 P Hp = 0，则 (W + D P) Hp = W Hp。但当 Hp 满行秩时，精确零空间只包含零向量；使用近似低特征值子空间便只有近似保留，阈值须报告。这里可借鉴 ROME/AlphaEdit，不能把矩阵恒等式升级为所有未见场景的行为保证。

### 6.2 动作头编辑的等价性

`W' = W + alpha u v^T` 精确等价于 `z' = z + alpha u (v^T h)`。

只要输入 h 相同，两者动作分布相同。因此“写进权重”并不比这个外部 logit 修正更强。若外部模块使用了新观测、非线性门控或记忆，那是额外能力，不能再称相同对照。偏置实现的 v^T h - tau 也不是 ReLU 门；负值会反向改变偏好。

### 6.3 风险与任务进展分开

可把短期候选选择写成 `argmax_a [Q_goal(h,a) - lambda Risk(h,a)]`，或者最大化任务价值同时满足风险上界。两者的意义不同：惩罚系数本身不保证约束满足。

对可积损失 L，取 0 < beta < 1，尾部风险可用 `CVaR_beta(L) = min_tau {tau + E[max(L-tau,0)]/(1-beta)}`。先定义 L 是碰撞指示、最小间距损失还是全回合累积损失；三者不能互换。对极少事件，用回合级风险和置信区间，不能只看均值。

对于几何安全函数 b(x)，离散屏障条件示意为 `b(x_next) >= (1-kappa) b(x)`，其中 0 < kappa <= 1。名义不变性还要求起点 b(x0) >= 0，之后每一步都满足约束；动态行人必须纳入联合状态，或使用时变函数 b_t。四动作可逐个检查；不存在可行候选时须记录失败，不能暗中增加等待/后退。还要检查动作执行期间的扫掠路径，终点无碰撞不足以保证中途无碰撞。预测误差、遮挡和人反应未知时，模型上的条件不是真实系统无条件保证。

### 6.4 几何补偿和交互残差

用 `p_human_world = R_robot p_human_robot + r_robot` 把行人位置还原到共同坐标系，再比较不同动作下的未来位置。先排除机器人自身姿态变化，再分析剩余运动差异。

这里的位置必须是同步、标定后的米制三维/地平面位置，不能把 RGB 像素直接代入。未来时刻要用各动作分支对应的机器人姿态与平移；只用当前姿态补偿，仍可能混入自车运动。

可将人运动写成“恒速/惯性基线 + 学习交互残差 + 不确定性”。这是结构先验，不意味着动量/能量守恒；目标切换、避让、刹停与感知都在系统外提供驱动。Koopman 或 port-Hamiltonian 可用于可解释表征与局部结构，但不要把它们的条件性稳定结论当成社会行为正确性证明。

## 7. 当前算力下怎样安排验证

|阶段|要回答的问题|所需材料与计算|继续条件|
|---|---|---|---|
|0. 对齐材料|我们运行的究竟是哪一个基线？|匹配代码/config/checkpoint、场景与资产；不申请 GPU 空等|动作、传感器、终止条件和指标可追溯|
|1. 故障诊断|错误出在感知、预测、读出还是执行？|少量固定回合、隐藏状态和动作日志；缓存编码|至少一种可重复故障有足够样本|
|2. 离线机制|目标方向可分吗？保留矩阵是否允许修改？|CPU/SVD/小头拟合；必要时单卡编码|独立开发场景改善，保留集无明显破坏|
|3. 小闭环试验|离线改善能否改变实际回合？|配对场景与行人 seed，记录所有候选预算|不靠早停/超时换碰撞下降|
|4. 正式比较|能否跨场景、跨行人策略保持收益？|冻结方案后独立评估，多个种子和场景分层统计|胜过强而便宜的对照，置信区间支持结论|

可先设一个开发试验上限，例如只比较 5 类方法、每类至多 4 个强度值，使用共同开发回合；这是资源规划示例，不是功效足够的样本量承诺。正式样本量应由观测方差、最小有意义差值和场景数量决定。所有搜索/失败候选都计入仿真步数、模型前向量与编辑器成本。

首轮最小对照集：原策略、bias-only、全头轻量拟合、同范数随机编辑、保留约束编辑。若研究 World Model，则再加入恒速预测/固定安全距离、预测误差训练、动作排序训练及匹配推理次数的短时域方案。不要同时改动作空间、输入权限、语言、奖励和模型，否则无法归因。

TACC 3090 适合先缓存表征并拟合小模块；并无证据证明本文列出的全部方法都能在一小时内复现。基础视频模型预训练、完整 Dreamer/大型 V-JEPA 训练或大规模扩散规划训练不属于首轮建议。原文使用小模型，也不等于其数据收集和训练总成本低。

## 8. 该学哪些前置知识，学到什么程度

|顺序|知识|掌握标准|对应研究能力|
|---|---|---|---|
|1|线性代数：SVD、秩、零空间、投影、二次型、ridge|能推导 6.1；解释满秩为何无非零精确保留编辑|判断编辑能做什么与不能做什么|
|2|概率与统计：条件分布、分位数、校准、CVaR、bootstrap|区分帧/回合/场景分母；解释交换性和策略分布变化|不夸大安全覆盖与小样本效果|
|3|控制与几何：坐标变换、运动学、MPC、屏障函数|能检查短动作扫掠路径、相对速度与可行集|将预测输出接到真实可执行动作|
|3b|动力系统：离散积分、线性化、特征值、耗散与稳定|解释稳定不等于无碰撞，系统也可能稳定在死锁|审查物理约束的实际作用|
|4|强化学习：POMDP、PPO、actor-critic、奖励与约束|解释辅助头不直接改动作；区别训练奖励与部署决策|沿推理图选择干预位置|
|5|World Model：潜在状态、teacher forcing、滚动误差、价值等价|举出像素误差大但动作正确、像素误差小却动作危险的例子|设计决策相关目标与短视规划|
|6|人群交互：ORCA、社会力、博弈、路径类别|解释互惠假设、非合作行人与对称死锁|设计模型失配和反事实试验|
|7|因果干预与适配：激活替换、LoRA、保留损失|分清相关可读出、干预有效和永久权重修改|把“已有知识”变成可检验机制|

学习顺序不要求先通读所有文献。先吃透三篇原文及一个故障，沿上表补知识，再读对应直接竞争论文；通用大 World Model 主要用于理解设计原则。

## 9. 评估中必须保留的边界

SR、SPL、HCR/H-Coll、PSC、超时、回合时长应共同报告。另直接统计“成功且无碰撞”的回合比例，不能用 SR × (1-HCR) 代替联合事件。可对安全指标施加预先定义的非劣阈值，再比较成功率；不要看完终评结果才挑对自己有利的指标。

场景是重要统计单位。训练、编辑、校准、模型选择、终评场景应分开；同一轨迹相邻帧不能随机拆分后声称泛化。比较采用配对 episode/seed，区间按场景聚类，统计失败类型与计算时间。仿真真值监督、oracle 控制器和部署可见观测必须明确标注。

在 ORCA 行人环境上改善不意味着对真实人安全。需要测试行人不互惠、急停、遮挡、群体移动、不同速度/密度与反应时延；且这些压力测试不能随意替代官方基准定义。原基准成绩和扩展诊断分开记录。

本次调研给出的优先级属于“值得花少量资源证伪”的判断，不是已选择投稿 IDEA，也不承诺接近 SOTA。第一项真正有说服力的证据，是在材料和协议对齐后，某个明确故障被稳定减少，并且正常行为与跨域表现没有靠额外信息或预算换来。

## 10. 检索范围与不能据此下的结论

本次是围绕三篇种子开展的广泛定向调研，采用精确题名、邻近方法、反向失败检索与有限前后向追踪；不是覆盖所有数据库的系统综述。主线覆盖风险控制、物理交互、世界模型与编辑。一般高能/量子物理、无任务依据的抽象理论拼接没有优先纳入。来源以 arXiv、会议原文、出版社或作者机构全文为主，搜索摘要只作发现线索。

近期预印本尚需独立复现，原始页面未见撤稿标注不等于完成全库勘误审计。arXiv 插件曾返回 429，OpenReview 部分页面要求浏览器验证，部分大 PDF 超出网页工具容量；已用可取得的官方全文补足相关条目。完整查询、访问限制、元数据冲突和去重记录见同目录材料。未取得全平台稳定命中总数，不报告虚假的 PRISMA 式全量筛选漏斗。

正式版与预印本按研究家族去重，不把 v1/v2、HTML/PDF、不同镜像重复计数。只对实际读取的正文段落作机制判断；不把代码链接存在视为成功运行，不将作者设备的时间移植为 3090 实测，不把题名中“安全”“世界模型”“物理”自动视作已满足相关性质。


## 11. 按任务选读的文献卡片

编号仅用于本报告定位；预印本年份采用首发年份，所读修订版本单列；正式发表关系见注册表。正式论文使用卷期或会议年份，在线日期差异另列。以下迁移建议及优先级均为本次分析推断。完整作者在 BibTeX/RIS 与注册表中，卡片只缩写展示。

### 数学、控制与风险

**M01 | Egocentric Conformal Prediction for Safe and Efficient Navigation in Dynamic Cluttered Environments**

2025；Shin, Jaeuk；Lee, Jungjin；Yang, Insoon。版本：v1。[原文](https://arxiv.org/abs/2504.00447)

方法：ECP用机器人候选位置与实际/预测障碍集的距离差定义非一致性分数，有限动作序列版避免在整个连续状态空间更新校准器。

边界：安全集合包含关系要求相同校准数据及相同校准水平；连续空间离散化昂贵，有限序列仍随预测长度组合增长。目标点覆盖不能直接替代连续动作扫掠体避碰。

迁移假设：最适合Falcon四动作短视界：校准动作造成的危险距离误差，而非统一惩罚全轨迹ADE。NavThinker可在少量候选动作未来上使用相同思路。

最小验证：冻结策略/预测器，比较欧氏误差分数与决策距离分数；同样校准样本、风险预算和动作枚举长度，测碰撞、进展、超时、干预率。

正文定位：[§4.1–4.2; Proposition 1; Algorithm 1](https://arxiv.org/html/2504.00447v1)。

**M02 | Towards Generalizable Safety in Crowd Navigation via Conformal Uncertainty Handling**

2025；Yao, Jianpeng；Zhang, Xiaopan；Xia, Yu 等。版本：v1。[原文](https://arxiv.org/abs/2508.05634)

方法：GenSafeNav把DtACI轨迹不确定性同时加入策略输入和预测侵入代价，使用奖励/代价双critic与PPO-Lagrangian。

边界：文中限制包括任意形状静态障碍、行人漏检、尚未消除的OOD退化。主要环境为二维CrowdNav，不能将其SR直接与Habitat/Falcon比较。

迁移假设：是“预测不确定性+安全RL”的直接先例；可提供风险教师与强baseline，但完整移植需改变观测并重训策略。

最小验证：先做冻结策略上的外部风险读出；另设有限预算头部微调对照。必须把校准覆盖与回合避碰分别统计。

正文定位：[§3.4–3.6; §6 Limitations; Appendix M](https://arxiv.org/html/2508.05634v1)。

**M03 | Risk-Sensitive Crowd Navigation with Adaptive Ellipsoidal Conformal Prediction**

2026；Li, Ruihan A.；Guo, Ziyao；Li, Yingying。版本：v1。[原文](https://arxiv.org/abs/2610.07474)

方法：AECP以因果可用残差更新单位行列式椭球形状，源时刻形状评价延迟残差；DtACI独立校准尺度，episode CVaR调节PPO安全乘子。

边界：作者明确：部署半径截断后不再保证名义覆盖；OOD实测存在欠覆盖。二维连续速度、GST输入及重训PPO不等于Falcon视觉离散动作设置。

迁移假设：直接否定“椭球+CVaR社会导航”作为新点。可研究有限动作、部分观测条件下形状—尺度—校准失配及只改策略头的损失。

最小验证：比较圆形/椭球×均值代价/CVaR的完整2×2；冻结和重训分开，报告未截断与部署截断覆盖、同场景配对差异。

正文定位：[§III-B Eqs.1–7; §III-C; §IV Coverage Evaluation](https://arxiv.org/html/2610.07474v1)。

**M04 | Interaction-aware Conformal Prediction for Crowd Navigation**

2025；Huang, Zhe；Ji, Tianchen；Zhang, Heling 等。版本：v1。[原文](https://arxiv.org/abs/2502.06221)

方法：ICP交替机器人MPC与根据计划生成的人类ORCA模拟轨迹，重新构造动作依赖校准集；预测GST权重固定。

边界：校准依赖交互模拟器及误差交换性假设；不能把ORCA合成分布当真实人类分布。非凸MPC一般无全局可行性保证，文本分位数有限样本索引需实现时独立核查。

迁移假设：NavThinker对动作条件化未来建模之后仍需动作条件化风险校准；策略编辑改变动作与访问状态分布，旧校准集可能失效。

最小验证：固定场景种子，原策略校准vs编辑后重校准vs动作分层校准；避免用测试轨迹校准，比较覆盖/碰撞及分层样本稀疏程度。

正文定位：[§3 Theorem 3.1; §4.2–4.3 Eqs.3–7](https://arxiv.org/html/2502.06221v1)。

**M05 | Safety-Critical Control with Uncertainty Quantification using Adaptive Conformal Prediction**

2024；Zhou, Hao；Zhang, Yanze；Luo, Wenhao。版本：v2。[原文](https://arxiv.org/abs/2407.03569)

方法：ACP-SBC直接校准预测/真实安全屏障表达式的误差，采用误差分位数收紧CBF约束并放入MPC；验证单积分器和独轮车噪声场景。

边界：需要动力学、屏障正则性、有效误差界与可行控制输入；连续时间屏障不自动覆盖离散转动/前进动作或动作执行过程。

迁移假设：提供“校准控制相关误差”的理论材料。Falcon应做动作级安全掩码/短轨迹最小间距，而非生搬连续QP。

最小验证：先用模拟器真状态作oracle筛选诊断，再用允许观测估计状态；两种成绩分列，检测空安全动作集合与恢复策略。

正文定位：[§III-A Eqs.10–14, Theorem 1 and Proposition 1; §IV](https://arxiv.org/html/2407.03569v2)。

**M06 | Integrating Predictive Motion Uncertainties with Distributionally Robust Risk-Aware Control for Safe Robot Navigation in Crowds**

2024；Ryu, Kanghyun；Mehr, Negar。版本：v1。[原文](https://arxiv.org/abs/2403.05081)

方法：以碰撞概率机会约束表达风险，使用分布鲁棒CVaR保守近似，通过采样优化把预测不确定性纳入实时MPC。

边界：保证以真实分布处于歧义集合为前提；分布/矩估计误差与动作条件化分布漂移需检查。保守约束可能换来停滞，原实现依赖预测器和求解器。

迁移假设：对NavThinker低维未来成本建模有直接价值；Falcon少动作枚举可避免大型连续优化，代价是无法照搬原闭环保证。

最小验证：对同一批未来样本比较均值、上分位数、CVaR动作排序；记录尾部样本数并增加风险预算扫参，保持SPL/超时对照。

正文定位：[§III-A/B; §IV DRCC-MPC formulation](https://arxiv.org/html/2403.05081v1)。

**M07 | Safe POMDP Online Planning among Dynamic Agents via Adaptive Conformal Prediction**

2024；Sheng, Shili；Yu, Pian；Parker, David 等。版本：v2。[原文](https://arxiv.org/abs/2404.15557)

方法：以ACP预测区域构造动态安全shield，限制POMCP可选动作，并在有限POMDP信念支持中计算winning regions。

边界：正文承认粒子近似信念可能损害保证，定理依赖足够准确的信念估计；更大粒子/搜索数带来计算代价。视觉隐状态并非已知有限POMDP。

迁移假设：支持把未观测人群/遮挡风险纳入belief，而不只预测已检测行人；可作离散动作安全筛选的概念baseline。

最小验证：先做短历史/不同遮挡率的风险读出；报告漏检状态下校准覆盖，oracle可观测与真实视觉观测绝不混分。

正文定位：[§II; Algorithm 1–2; Theorem 1 and Complexity](https://arxiv.org/html/2404.15557v2)。

**M08 | DR-MPC: Deep Residual Model Predictive Control for Real-world Social Navigation**

2024；Han, James R.；Thomas, Hugues；Zhang, Jian 等。版本：v2。[原文](https://arxiv.org/abs/2410.10646)

方法：DR-MPC产生六个避人候选动作，学习混合系数将其与MPC路径跟踪动作组合，并用OOD检测/启发式安全组件辅助真实环境学习。

边界：连续速度加权混合不适用于离散动作标签；文中指出奖励手调、人圆形近似误检及实验人群熟悉机器人等限制。

迁移假设：是小数据策略修复的控制参考；可将安全教师的动作分布蒸馏到Falcon头部，而不是重训视觉骨干。

最小验证：对比外部混合/屏蔽、头部拟合与同预算PPO微调；测干预频率、正常场景保留、窄门与拥挤交叉口。

正文定位：[§IV DR-MPC Policy Architecture; §VII-D Limitations](https://arxiv.org/html/2410.10646v2)。

**M09 | Control Barrier Functions: Theory and Applications**

2019；Ames, Aaron D.；Coogan, Samuel；Egerstedt, Magnus 等。版本：v1。[原文](https://arxiv.org/abs/1903.11199)

方法：CBF将安全表示为集合正向不变性，并用约束优化最小修改名义控制器；安全与到达目标的活性是不同要求。

边界：理论依赖已建模动力学、正则性、初始安全与可行控制。不存在“加一个安全惩罚就得到CBF保证”的结论。

迁移假设：属于前置知识锚点，用于理解为何安全过滤可能冻结，以及为何Falcon离散动作需要单独分析。

最小验证：用二维玩具系统检查屏障、离散时间与输入界；不把玩具系统通过当作Habitat安全结果。

正文定位：[§II–III control barrier definitions and optimization controllers](https://arxiv.org/html/1903.11199v1)。

**M10 | BIG-CBF: Behavior-Imagination-Guided Control Barrier Function with Shared Uncertainty for Mobile Robot Navigation**

2026；Li, Shibo；Wang, Zhongcheng；Cao, Jiahe 等。版本：v1。[原文](https://arxiv.org/abs/2609.14343)

方法：BIG-CBF低频比较六种闭环行为，高频重算控制并通过硬CBF；规划和执行共享延迟、预测及执行误差来源。

边界：原机器人为全向连续控制，采样命令级滤波与假设误差模型不可直接转换为视觉四动作的永远安全保证。15次硬件运行不是全分布证明。

迁移假设：是“世界想象+安全过滤减少冻结”的直接竞争。NavThinker可研究想象风险与执行风险的一致性，Falcon则需离散可行动作版本。

最小验证：有限动作/动作片段rollout，共享vs不共享风险模型；同时比较纯CBF/纯策略/带想象筛选，统计停滞与切换频率。

正文定位：[§III-A Eq.7–13; §III-B; §III-C Eqs.25–26](https://arxiv.org/html/2609.14343v1)。

**M11 | When Environments Shift: Safe Planning with Generative Priors and Robust Conformal Prediction**

2026；Kaizer Rahaman；Jyotirmoy V. Deshmukh；Ashish R. Hota 等。版本：PMLR 331:601–623。[原文](https://proceedings.mlr.press/v331/rahaman26a.html)

方法：用可观测环境参数条件化扩散模型，生成轨迹或误差分数校准样本，并通过鲁棒CP计入生成模型失配后嵌入MPC。

边界：要求有意义的可观测环境参数和受控模型分布失配；实验为ORCA，正文将交互式及非平稳移位列为未来方向。不能用世界模型自己生成的数据无条件自证正确。

迁移假设：与NavThinker世界模型直接相接：预测器可输出校准数据，但必须另估模型偏差；只训练低维残差分布比训练视频世界模型省算力。

最小验证：先用经验残差重采样/简单条件高斯对照，再决定是否需要小型条件扩散；在held-out场景检验失配界与覆盖，禁用测试真值训练。

正文定位：[§2–4; §6; Appendix 7.3](https://raw.githubusercontent.com/mlresearch/v331/main/assets/rahaman26a/rahaman26a.pdf)。

### 物理、几何与人群交互

**P01 | Social force model for pedestrian dynamics**

1995；Dirk Helbing；Péter Molnár。版本：v1。[原文](https://doi.org/10.1103/PhysRevE.51.4282)

方法：目标驱动项、人与边界斥力及吸引项构成随机动力系统。

边界：经典行为近似，不能推出真实行人遵守封闭系统能量守恒或固定交互参数。

迁移假设：可作为Falcon/NavThinker的人运动外推基线或预测残差先验；不能靠单独编辑训练辅助头改变未接入该头的策略。

最小验证：CPU比较匀速、SFM和有限加速度模型的风险排序；核对时间单位、朝向和观察噪声，再决定是否训练残差。

正文定位：[II; III, Eqs.1-11](https://arxiv.org/html/cond-mat/9805244v1)。

版本备注：正式1995；arXiv上传1998，按同一论文计一次。

**P02 | Reciprocal n-Body Collision Avoidance**

2011；Jur van den Berg；Stephen J. Guy；Ming Lin 等。版本：see source record。[原文](https://gamma-web.iacs.umd.edu/ORCA/)

方法：速度障碍的可行半平面交集，线性规划选择接近期望速度的动作。

边界：连续速度、多智能体互惠假设不等价于单机器人面对不可反应行人；局部无碰撞不代表全局到达或无死锁。

迁移假设：适合作为几何避碰teacher/强控制基线；Falcon四离散动作需逐动作rollout，不能直接声称保留原连续控制保证。

最小验证：固定相同观测和离散动作，比较完整责任VO、半责任ORCA与原actor；报告停滞与碰撞，禁止用GT人状态对比只看图像的agent。

正文定位：[§1; §4.2, Eq.6; §5](https://gamma-web.iacs.umd.edu/ORCA/publications/ORCA.pdf)。

**P03 | DenseKoopman: A Plug-and-Play Framework for Dense Pedestrian Trajectory Prediction**

2024；Xianbang Li；Yilong Ren；Han Jiang 等。版本：see source record。[原文](https://www.ijcai.org/proceedings/2024/113)

方法：Koopman提升空间内预测，解码还原轨迹。

边界：预测指标不是闭环导航；线性提升不是任意非线性系统存在精确有限维线性化的证明；像素结果不可直接与米制比较。

迁移假设：候选：冻结NavThinker表示，用低维、带动作输入的线性残差预测风险特征；从离线轨迹学习仍须识别动作覆盖与分布移位。

最小验证：比较同维同参数线性ridge、MLP残差与Koopman约束，检测多步谱半径/漂移、风险排序和闭环收益。

正文定位：[§3.1-3.4; §4.1](https://www.ijcai.org/proceedings/2024/0113.pdf)。

**P04 | Human-Robot Cooperative Distribution Coupling for Hamiltonian-Constrained Social Navigation**

2024；Weizheng Wang；Chao Yu；Yu Wang 等。版本：v2。[原文](https://arxiv.org/abs/2409.13573)

方法：端口Hamiltonian动态结构、扩散式交互不确定性与RL。

边界：论文所用结构及输入条件下的被动性不能自动成为真实视觉社会导航的碰撞保证；不可见机器人实验不证明双向适应。

迁移假设：是“物理+扩散+社交导航”的直接先例，不能将此组合当新意；可借鉴耗散结构限制latent rollout发散。

最小验证：先用同参数自由残差与耗散残差对照，统计多步误差/风险失真/停止率；若只压平预测造成过度停滞则放弃。

正文定位：[§IV-A, Eqs.3-7; §V-A](https://arxiv.org/html/2409.13573v2)。

**P05 | AToM: Adaptive Theory-of-Mind-Based Human Motion Prediction in Long-Term Human-Robot Interactions**

2025；Yuwen Liao；Muqing Cao；Xinhang Xu 等。版本：v2。[原文](https://arxiv.org/abs/2502.05792)

方法：微分博弈预测人对机器人的内部信念，UKF在线更新行为参数。

边界：已知目标、效用模型和反复互动是重要条件；不能宣称ILQ近似求解在任意非凸问题得到全局Nash均衡；匿名短episode的认知适应不可由此直接推出。

迁移假设：建议只学低维交互类型/风险偏好latent，不先重训视觉encoder；必须区分轨迹回放行人与会对机器人反应的人。

最小验证：两智能体交会CPU沙盒：固定偏好/变化偏好；比较无适应、滑窗回归、UKF，观察后验识别与deadlock，后续才接导航。

正文定位：[§III-B, Eqs.3-5; Algorithm1](https://arxiv.org/html/2502.05792v2)。

**P06 | TPK: Trustworthy Trajectory Prediction Integrating Prior Knowledge For Interpretability and Kinematic Feasibility**

2025；Marius Baden；Ahmed Abouelazm；Christian Hubschneider 等。版本：v4。[原文](https://arxiv.org/abs/2505.06743)

方法：交互先验引导注意力，类别专属运动学层把控制量转成轨迹。

边界：AV2主要为车辆、含带噪标签；运动学可行不等于社会正确或碰撞安全；全模型17.7M参数，不应称直接免训练补丁。

迁移假设：可为NavThinker轨迹decoder加受限加速度层，或将违反运动界限作为不确定性提示；界限需按数据采样/行人生成器重新标定。

最小验证：固定预测器，对比原轨迹、平滑、加速度投影与软惩罚；同时测ADE、碰撞风险漏报和真实导航，防止平滑掩盖急转。

正文定位：[§III-B; §IV; §V, TablesIII-V](https://arxiv.org/html/2505.06743v4)。

**P07 | STFlow: Data-Coupled Flow Matching for Geometric Trajectory Simulation**

2025；Kiet Bennema ten Brinke；Koen Minartz；Vlado Menkovski。版本：v3。[原文](https://arxiv.org/abs/2505.18647)

方法：使用观测条件随机游走作生成先验，学习到目标轨迹的flow。

边界：不是HM3D社会导航验证；best-of-k与均值指标表现不同；不能把“可单卡”写成一小时可复现。

迁移假设：借鉴惯性/连续性先验做小轨迹残差或采样候选，暂不复现完整生成模型；不直接支持编辑视频WM权重即可提高SR。

最小验证：在相同小MLP/flow与训练预算下比较高斯、匀速随机游走、SFM先验，按预测时延和风险排序筛选。

正文定位：[§3.2, Eq.2-3; §4.1.3; AppendixA.4](https://arxiv.org/html/2505.18647v3)。

版本备注：v1标题Flow Matching for Geometric Trajectory Simulation；采用v3新标题，一篇计一次。

**P08 | A Game of Social Forces: Integrating Non-cooperative Game Theory with Social Force Model for a Socially-acceptable Mobile Robot Navigation**

2025；Giada Galati；Andrea Usai；Giacomo Vignolo 等。版本：see source record。[原文](https://doi.org/10.1007/s10846-025-02318-0)

方法：SFM生成候选轨迹，非合作博弈选择；用神经网络近似慢参数估计。

边界：理性效用/候选集合假设；多agent复杂度；几何指标提升不能替代人感知改善。

迁移假设：适合把四种候选“轨迹/意图”转成离散重排序teacher；其四候选轨迹不等于Falcon四瞬时动作。

最小验证：仅最相关一名行人，2×2或4×4博弈；比较同代价的独立最优、minimax和博弈解，限定延迟预算。

正文定位：[pp.3,6 (§4.1); §6-7](https://link.springer.com/content/pdf/10.1007/s10846-025-02318-0.pdf)。

**P09 | SHINE: Social Homology Identification for Navigation in Crowded Environments**

2026；Diego Martinez-Baselga；Oscar de Groot；Luzia Knoedler 等。版本：v3。[原文](https://doi.org/10.1177/02783649251344639)

方法：同调类表示左右绕行/先后通过，学习类代价，再由MPC执行。

边界：已有拓扑模式选择和一致性约束，单加左右偏好不新；需要轨迹/障碍推断，过强坚持旧模式会妨碍紧急避碰。

迁移假设：优先研究低维“模式一致性+风险触发释放”接在Falcon动作重排序或NavThinker未来特征上，不重训大WM。

最小验证：对向/交叉/窄门场景比较原actor、简单动作平滑、模式记忆、风险解除记忆；测左右切换、停滞、SR/SPL/碰撞。

正文定位：[§4.3; §5.1-5.3](https://arxiv.org/html/2404.16705v3)。

版本备注：2024首版arXiv；2025在线正式发表/作者代码名IJRR2025；2026卷45(1):60-79，文献年份采用卷期2026。

**P10 | Multi-robot navigation in social mini-games: definitions, taxonomy, and algorithms**

2026；Rohan Chandra；Shubham Singh；Wenhao Luo 等。版本：see source record。[原文](https://doi.org/10.1007/s10514-026-10251-w)

方法：约束空间强耦合互动的定义、方法分类和评价框架。

边界：综述不是新控制器的性能证明；多机器人协议不能直接假设人类服从。

迁移假设：用于三baseline失败分层：安全但停住/反复让行/强迫别人让行分别记录；DPed-VLN还要保持指令目标正确。

最小验证：构造三类固定种子微场景，报告安全、到达、停滞、对其他agent干扰及观察权限；这是任何权重编辑前置诊断。

正文定位：[§2.1-2.5; §3-4](https://link.springer.com/article/10.1007/s10514-026-10251-w)。

版本备注：正式版4作者；arXiv早期索引列6作者，采用出版社正式版，不混拼作者表；版本关系由作者项目页支持。

**P11 | COMPASS: Temporally Consistent Topological Passing Decisions for Socially Aware Robot Navigation**

2026；Jungmo Kang。版本：v1。[原文](https://www.preprints.org/manuscript/202610.0120)

方法：以泄漏证据累积器管理通过侧承诺，安全仲裁优先于模式惯性。

边界：未同行评审；track-ID类生命周期仅部分实现；离线反复选择次数减少不能推为机器人碰撞减少；默认简单dwell也是强对照。

迁移假设：否定“模式记忆+风险释放本身就是新贡献”的笼统说法。可研究受感知不确定性影响的模式记忆是否可安全蒸馏到冻结导航policy，但必须测可表示性及闭环。

最小验证：模式选择baseline至少含SHINE式前类代价、固定dwell、COMPASS式累积器；比较同等延迟/误报约束下的噪声抖动、遮挡ID切换、风险突变和导航成功。

正文定位：[§2; §3.1-3.5; §4-5](https://www.preprints.org/manuscript/202610.0120)。

### World Model 与决策表征

**W01 | The Value Equivalence Principle for Model-Based Reinforcement Learning**

2020；Grimm, Christopher；Barreto, André；Singh, Satinder 等。版本：v1。[原文](https://arxiv.org/abs/2011.03506)

方法：针对策略族Π与价值函数族V，要求模型产生相同Bellman更新；不必拟合全部转移细节。

边界：理论依赖指定函数和策略集合；平均回报等价不自动保证罕见碰撞尾风险、分布外行为或POMDP信念正确。

迁移假设：推断：NavThinker预测空间应保留进度与碰撞代价的决策差异，加入多成本Bellman目标，而非只压低latent MSE。

最小验证：冻结表征训练小动力学与多成本头；同一候选集比较MSE训练和价值/风险加权训练，测行动排序及闭环风险；若MSE下降而排序不改善则停止。

正文定位：[§3 Definitions 1–2, §3.1–3.2](https://proceedings.neurips.cc/paper/2020/file/3bb585ea00014b0e3ebe4c6dd165a358-Paper.pdf)。

资源备注：理论与小头实验；无新基础模型预训练需求

**W02 | Mastering Diverse Domains through World Models**

2023；Hafner, Danijar；Pasukonis, Jurgis；Ba, Jimmy 等。版本：v2。[原文](https://arxiv.org/abs/2301.04104)

方法：循环随机状态空间模型学习动态，actor/critic在想象轨迹中训练；在线采样actor而非每步搜索。

边界：这是整个RL算法，不是给Falcon加一个预测头即可复现；需要交互数据和训练，可能利用模型误差。

迁移假设：推断：值得借鉴信念记忆与短想象+价值尾项；不优先替换现有已训练Falcon全栈。

最小验证：保持原actor，用小头预测未来风险与进度；先检验冻结记忆是否能预测遮挡后行人，再考虑想象更新。

正文定位：[Learning algorithm; World model learning; Critic learning](https://arxiv.org/html/2301.04104v2)。

资源备注：原文规模不能直接当单3090预算；本次只建议小头诊断。

**W03 | TD-MPC2: Scalable, Robust World Models for Continuous Control**

2023；Hansen, Nicklas；Su, Hao；Wang, Xiaolong。版本：v2。[原文](https://arxiv.org/abs/2310.16828)

方法：联合学习潜在动态、奖励和价值，以策略先验引导短视MPC并用终端Q补足时域。

边界：主体实验为连续控制；潜在Q在规划分布偏移下可能高估，不能据其104任务表现推断社会导航优势。

迁移假设：推断：适合四离散动作的小候选树+终端价值；比大视频生成更符合低预算。

最小验证：1/2/4步固定候选预算比较纯即时风险、累积风险、终端价值；报告错误排序与耗时，防止延长时域只放大偏差。

正文定位：[§3.1–3.2; Appendix I](https://arxiv.org/html/2310.16828v2)。

资源备注：选择小模型/缓存表征；317M多任务版本非本项目直接复现目标。

**W04 | DINO-WM: World Models on Pre-trained Visual Features enable Zero-shot Planning**

2024；Zhou, Gaoyue；Pan, Hengkai；LeCun, Yann 等。版本：v2。[原文](https://arxiv.org/abs/2411.04983)

方法：冻结DINOv2空间patch特征，学习动作条件未来特征；无需像素解码，CEM寻找达到图像目标的动作。

边界：基准不等同拥挤人群导航；图像目标的latent近不保证可达，离线动作覆盖和规划器影响结果。

迁移假设：推断：冻结小编码器并缓存特征是可复用实现骨架；需要另学对碰撞/进度敏感的代价。

最小验证：用相同缓存特征比较DINO patch/池化特征在距离、碰撞、候选排序上的线性probe；仅特征MSE提升不继续。

正文定位：[§3.1–3.2; Appendix A.5](https://arxiv.org/html/2411.04983v2)。

资源备注：避免整套大模型训练；作者项目 https://dino-wm.github.io/，仓库执行/权重未核。

**W05 | V-JEPA 2: Self-Supervised Video Models Enable Understanding, Prediction and Planning**

2025；Assran, Mahmoud；Bardes, Adrien；Fan, David 等。版本：v1。[原文](https://arxiv.org/abs/2506.09985)

方法：在视频自监督编码器之上训练动作条件预测器，以teacher forcing和短rollout损失支持latent MPC。

边界：不能把动作无关视频表征等同可控因果模型；相机布局敏感、长时滚动误差、原MPC每动作16秒。

迁移假设：推断：借用短rollout损失，不直接移植大规模机械臂模型到实时行人导航。

最小验证：固定导航编码器，只训练动作适配层；比较teacher forcing与2步自回归潜在rollout，再测试动作打乱是否显著恶化。

正文定位：[§3.1; §4.3; Table 3](https://arxiv.org/html/2506.09985v1)。

资源备注：原Table3：单RTX4090、800样本×10迭代、时域1，16秒/动作；不是3090实时方案。

**W06 | Navigation World Models**

2024；Bar, Amir；Zhou, Gaoyue；Tran, Danny 等。版本：v2。[原文](https://arxiv.org/abs/2412.03572)

方法：动作条件扩散视频模型可生成/筛选导航候选，使用连续平移、偏航与时间条件。

边界：生成逼真不保证动作执行准确或行人响应正确；大候选搜索代价高，不能拿视频指标当闭环安全。

迁移假设：推断：用它定义动作敏感性与动态交互的反例测试，而非从头训练十亿参数模型。

最小验证：固定图像与行人初态，仅改机器人动作，检查预测是否区分停止/前进；再固定机器人动作改变行人时间相位。

正文定位：[§3.1–3.3; §5 Limitations](https://arxiv.org/html/2412.03572v2)。

资源备注：模型可至1B；本次无完整训练预算核算，建议仅方法借鉴。

**W07 | Social-WM: Safety-Aware Latent World Models for Robot Social Navigation**

2026；Zheng, Zhihao；Chuah, Mooi Choo。版本：v2。[原文](https://arxiv.org/abs/2609.40177)

方法：冻结DINOv2-small；latent未来+可实现动作逆动力学，按命令与预计实际位移差筛选候选。

边界：名义动作差不等于普遍安全证书；安全标签受gate构造影响，与Falcon四动作协议需重核。

迁移假设：推断：最直接重叠，不能把可执行性筛选当新贡献；可研究校准与持续阻挡后的重规划。

最小验证：同传感器/四动作/场景审计后，比较恒定阈值与风险校准；分别审计采集期keep-out gate与部署期学习筛选；若另加真值测试gate，单独消融，匹配SR同时考察碰撞/超时。

正文定位：[§III-A–F; §IV-B,H–J](https://arxiv.org/html/2609.40177v2)。

资源备注：§IV-I：58M总参数，A5000上CVAE规划51ms/步；本机/3090尚未测量。

**W08 | MWM: Mobile World Models for Action-Conditioned Consistent Prediction**

2026；Yan, Han；Xiang, Zishang；Zhang, Zeyu 等。版本：v1。[原文](https://arxiv.org/abs/2603.07799)

方法：通过自身滚动上下文改善动作条件一致性；冻结CDiT主干，仅更新动作注入AdaLN，并做推理一致蒸馏。

边界：只改动作调制层已是已有工作；视频生成速度在RTX PRO6000测量，未证明3090实时；SCAND离线误差不是社会安全。

迁移假设：推断：若选编辑WM权重，动作嵌入/AdaLN优先于盲目编辑任意MLP，但需保留语义与离线动作覆盖。

最小验证：针对NavThinker实际动作条件模块，做相同参数量随机层/动作层/输出logit校正控制；只以闭环改善决定去留。

正文定位：[§III-B.2–C; Tables III–IV](https://arxiv.org/html/2603.07799v1)。

资源备注：表III：MWM DDIM5平均rollout2.3s，NWM DDIM25为9.6s；硬件RTX PRO6000。

**W09 | LeWorldModel: Stable End-to-End Joint-Embedding Predictive Architecture from Pixels**

2026；Maes, Lucas；Le Lidec, Quentin；Scieur, Damien 等。版本：v3。[原文](https://arxiv.org/abs/2603.19312)

方法：小型端到端JEPA以next-latent MSE+SIGReg防塌缩；动作通过AdaLN注入，CEM规划。

边界：防塌缩不等于社会几何足够；论文指出低维低多样数据困难、离线覆盖和短时域限制。

迁移假设：推断：适合作低成本玩具反例与目标设计试验；不据其物理probe直接声称学会人的意图。

最小验证：先TwoRoom或小导航数据对比MSE+防塌缩与风险目标；检查可达性误排序和转移，随后再接Falcon。

正文定位：[§3.1–3.2; §6 Limitations; Appendix D](https://arxiv.org/html/2603.19312v3)。

资源备注：v1 Appendix D报告单L40S；v3架构约5M编码器+10M预测器；不能保证3090同耗时。

**W10 | TD-M(PC)$^2$: Improving Temporal Difference MPC Through Policy Constraint**

2025；Lin, Haotian；Wang, Pengcheng；Schneider, Jeff 等。版本：v1。[原文](https://arxiv.org/abs/2502.03550)

方法：用行为分布log-likelihood约束策略更新，限制规划器与学习先验分布失配导致的价值高估。

边界：不是Falcon PPO直接可插入项；过强早期约束抑制探索，需有行为分布估计与价值学习。

迁移假设：推断：给编辑/重排序加KL或动作支持约束，避免WM从数据外虚假未来套利。

最小验证：在候选重新打分时加入对原actor的KL预算；比较同预算无约束、常数温度与低秩编辑；报告分布外失败。

正文定位：[§4; §5.1–5.2, Eq.10](https://arxiv.org/html/2502.03550v1)。

资源备注：作者称最小实现改动；迁移至本任务仍需设计，非已验证免算力提升。

**W11 | V-JEPA 2.1: Unlocking Dense Features in Video Self-Supervised Learning**

2026；Mur-Labadia, Lorenzo；Muckley, Matthew；Bar, Amir 等。版本：v3。[原文](https://arxiv.org/abs/2603.14482)

方法：对可见和掩码token均监督、跨中间层监督改善稠密空间表征；冻结特征训练动作条件导航模型。

边界：不能将ATE或10倍加速当碰撞改善；报告规划仍10.6秒，且1B/2B与蒸馏版能力不能混用。

迁移假设：推断：优先用已发布小特征做离线probe比较，验证碰撞几何/动作差异保留后才替换编码器。

最小验证：相同头、相同帧和预算比较DINOv2、DA-V2与可取得的V-JEPA蒸馏特征；只报告实际核验的权重与配置。

正文定位：[§2.3; §3.4 Table 7](https://arxiv.org/html/2603.14482v3)。

资源备注：§3.4：480候选、8去噪步、约10.6秒规划；论文称发布80M/300M蒸馏编码器，实际文件未下载核验。

**W12 | Learning Counterfactual World Models for Embodied Reasoning under Partial Observability**

2026；Zhou, Todd Y.；Zhang, Daniel。版本：v2。[原文](https://arxiv.org/abs/2609.05834)

方法：在循环信念状态上构造同锚点动作干预，用结果标签、知觉混淆和价值分歧挖掘难负例，学习反事实可分性。

边界：需要干预结果标签；实验为从头训练的模型，未验证大规模预训练编码器或真实人类社会行为；相关性不是因果证明。

迁移假设：推断：直接覆盖动作敏感/反事实训练想法；本项目应以它为对照，进一步隔离自车运动、人的响应与安全尾风险。

最小验证：同场景分支采集四动作短结果，比较普通对比、CLWM式难负例与风险校准；若只区分视角而不区分碰撞/进度则否定。

正文定位：[§2; §4.1–4.5; §8](https://arxiv.org/html/2609.05834v2)。

资源备注：标签采集和多分支仿真是主要成本；低维投影头可小，但原论文预算尚未独立核验。

**W13 | One-Step Next-Latent Prediction Is Not a World Model**

2026；Wang, Shitong；Cai, Zhongang；Hong, Yuzhou。版本：v1。[原文](https://arxiv.org/abs/2609.36227)

方法：平方损失识别条件均值；均值复合通常不等于多步条件均值，非单射观测需要历史，边缘各向同性不约束转移。

边界：理论/合成过程不能据此否定全部JEPA；文中Markov表述只写条件均值关系，引用时应使用标准条件分布定义并明确定理假设。

迁移假设：推断：对NavThinker评估多时域分布/行动排序、历史长度和残差校准，而非把MSE与防塌缩当充分条件。

最小验证：CPU构造非线性噪声与遮挡混淆反例；再测真实回合一步/开放环差异；增大记忆若不能改善决策则不扩展模型。

正文定位：[§3.1–3.5; Theorem 2, Theorem 4](https://arxiv.org/html/2609.36227v1)。

资源备注：理论与小合成反例低成本；不提供真实导航即插即用算法。

**W14 | The Objective Is the Bottleneck: Latent World Models Encode What Their Planners Cannot Use**

2026；Singh, Joyjeet。版本：v1。[原文](https://arxiv.org/abs/2608.12959)

方法：冻结世界模型，仅换planner标量目标；检查latent欧氏距离是否随真实距离单调，以及学习代价对想象状态的分布偏移。

边界：单一玩具环境和有限checkpoint；位置probe使用真值，不能当视觉基线同信息对照；不证明社会导航同病。

迁移假设：推断：先测排序/代价，再编辑权重；线性读出或外部代价是必须超越的便宜控制。

最小验证：冻结已有表征，在保留场景测latent距离、可达性/碰撞排序、预测与真实特征两分布；只换代价并保持候选/预算一致。

正文定位：[§3–7; §9 Limitations](https://arxiv.org/html/2608.12959v1)。

资源备注：§2报告四核笔记本CPU诊断；无新世界模型训练，但probe/代价拟合与真值权限应独立计入。

**W15 | An Efficient and Multi-Modal Navigation System with One-Step World Model**

2026；Shen, Wangtian；Meng, Ziyang；Ma, Jinming 等。版本：v1。[原文](https://arxiv.org/abs/2601.12277)

方法：shortcut目标+时空注意力3D U-Net并行预测多帧；固定速度锚点初始化多模态目标规划。

边界：场景数据每场景独立训练；文中175M模型速度在H200测量，不能外推单3090或未见场景社交导航。

迁移假设：推断：适合借用少候选锚点和并行短时域，暂不复现完整像素生成器。

最小验证：在已有actor上选4–8个可解释动作chunk，比较随机和锚点候选；候选相同再比较WM打分。

正文定位：[§3.1–3.4; §4.1–4.2](https://arxiv.org/html/2601.12277v1)。

资源备注：§4.2：175M，H200 batch1预测11未来帧0.076s；这不是整个规划器延迟。

**W16 | NavWM: A Unified Navigation World Model for Foresight-Driven Planning**

2026；Mei, Yanghong；Guo, Longteng；Yu, Ming-Ming 等。版本：v1。[原文](https://arxiv.org/abs/2606.24101)

方法：共享Mamba表征联合几何/语义、锚点多轨迹、未来视频；第二阶段用预测动作训练视觉分支。

边界：深度/语义伪标签与传感器条件需公平；1.5B模型和多GPU训练不符合当前首个低预算pilot。

迁移假设：推断：借鉴候选多样性与训练动作分布匹配；联合三个大模块不是当前最佳起点。

最小验证：在同一冻结表征下比较单候选/多锚点，保持总rollout数与时域一致；先测是否减少互让僵局。

正文定位：[§3.2–3.6; §4.1.3](https://arxiv.org/html/2606.24101v1)。

资源备注：§4.1.3：1.5B，8×A100，100k+50k训练步。

**W17 | Mind the Execution Gap: Action-Semantic Mismatch in World-Model Control**

2026；Wen, Shengtao；Chen, Xiang；Tian, Yu 等。版本：v1。[原文](https://arxiv.org/abs/2610.06582)

方法：区分规划未来动作时间线与循环模型上一动作归因，使用执行一致接口修复异步失配。

边界：主要针对异步通信/执行；Habitat同步基准若无该失配，不应凭空加延迟制造主实验收益。

迁移假设：推断：先审计动作、时间戳、碰撞后实际位移和reset；属于应先做的实现检查，不能冒充权重创新。

最小验证：记录commanded/applied actions，对随机延迟作独立压力测试；同步条件无差异则不作为主方向。

正文定位：[§2; §3; Appendix C–E](https://arxiv.org/html/2610.06582v1)。

资源备注：接口修复无需重训；需修改日志/执行层而非GPU大训练。

### 权重与表征编辑

**E01 | Locating and Editing Factual Associations in GPT**

2022；Kevin Meng；David Bau；Alex Andonian 等。版本：NeurIPS author PDF。[原文](https://papers.baulab.info/papers/Meng-2022.pdf)

方法：ROME：因果干预定位后，用保持旧映射的受约束最小二乘得到秩一更新。

边界：实验证据来自语言事实编辑；不证明循环导航状态可同样局部修复，也不是完整流程无梯度。

迁移假设：我们推断：借其保留集协方差与干预定位，将Falcon隐藏状态视为固定输入；不要照搬LLM层号。

最小验证：采集失败前与正常状态，先检验可分性，再比较协方差约束编辑和普通ridge头。

正文定位：[§2.1; §3.1 Eq.(2); §3.2 Eq.(4)](https://papers.baulab.info/papers/Meng-2022.pdf)。

**E02 | AlphaEdit: Null-Space Constrained Knowledge Editing for Language Models**

2025；Junfeng Fang；Houcheng Jiang；Kun Wang 等。版本：v4, 2025-04-22; canonical author list includes Jie Shi。[原文](https://arxiv.org/html/2410.02355v4)

方法：用保留输入矩阵的零空间投影约束参数扰动。

边界：有限保留样本及近似零空间不等于新场景闭环不退化；满行秩保留矩阵可能只剩零编辑。

迁移假设：我们推断：保留正常导航状态的logits，在失败方向上改actor；需检查特征谱和新策略状态分布。

最小验证：512维保留协方差谱诊断；hard-null与soft-retention、bias-only同预算比较。

正文定位：[§3.1–3.3 Eq.(7)–(14), §3.2 footnote 1](https://arxiv.org/html/2410.02355v4)。

**E03 | Reproducibility Study of "AlphaEdit: Null-Space Constrained Knowledge Editing for Language Models"**

2026；Ananth K S；Arya Hariharan。版本：v1, 2026-06-25。[原文](https://arxiv.org/html/2606.26783v1)

方法：复核AlphaEdit并扩展架构、下游任务和连续编辑长度。

边界：独立预印本；退化阈值不是跨模型定律，文内含模型依赖的恢复和未解决实现问题。

迁移假设：我们推断：导航连续编辑须记录累积漂移；出现负结果先排查状态恢复/输入与动作一致性。

最小验证：比较独立编辑与连续叠加，保存每版校验和及保留集表现，不把5000次当导航安全阈值。

正文定位：[§5.3 Table 5; §5.5; §7](https://arxiv.org/html/2606.26783v1)。

**E04 | Repairing Neural Networks for Safety in Robotic Systems using Predictive Models**

2024；Keyvan Majd；Geoffrey Clark；Georgios Fainekos 等。版本：v1, 2024-11-07。[原文](https://arxiv.org/html/2411.04408v1)

方法：SARP：通过可微预测模型与约束优化离线修复策略。

边界：预测器误差会传入安全约束；连续动作可微优化不能原样用于四动作采样策略。

迁移假设：直接重叠：世界模型辅助安全权重修复已有先例；我们的潜在差异只能落在离散视觉策略、预算、校准和分布外保留。

最小验证：作为可微修复对照；先穷举四动作风险，不把期望动作当真实执行动作。

正文定位：[§III–IV Algorithm 1; §V-A](https://arxiv.org/html/2411.04408v1)。

**E05 | Behavior Uncloning: Distilling Mode Redirection into Policy Weights without Inference-Time Steering**

2026；Hao Wang；Jiuzhou Lei；Danyou Li 等。版本：v1 header 2026-06-28; rendered manuscript date 2026-08-24 (recorded discrepancy)。[原文](https://arxiv.org/html/2606.29201v1)

方法：MoRE：训练行为模式分类器，用保留损失和模式重定向损失反传至策略，部署移除分类器。

边界：需要模式标签与梯度；BC/动作块体系与Falcon PPO离散动作不同。

迁移假设：我们推断：在碰撞/震荡已经不可挽回前定位分叉时刻，比较retain+redirect编辑。

最小验证：相同失败/正常轨迹与标注预算，比较末层微调、MoRE式损失和闭式编辑。

正文定位：[§3.2–3.4 Eq.(3)–(6); §4](https://arxiv.org/html/2606.29201v1)。

**E06 | What do VLM-Based Vision-Language Navigation Models Rely on: Interpreting and Steering Policy Behavior**

2026；Débora Oliveira Makowski；Samiran Gode；Abhijeet Nayak 等。版本：v1, 2026-09-21。[原文](https://arxiv.org/html/2609.24576v1)

方法：通过输入干预、概念和激活方向解释并引导VLN行为。

边界：实机恢复含人工触发；不能把局部恢复当全自动导航提升或永久权重编辑。

迁移假设：我们推断：适合作为故障状态可干预性的诊断及activation-steering对照。

最小验证：人工触发与自动触发分开报告；保存权重后无hook重载验证永久编辑。

正文定位：[§IV-B.2 Eq.(12)–(13); §V-I](https://arxiv.org/html/2609.24576v1)。

**E07 | ReFT: Representation Finetuning for Language Models**

2024；Zhengxuan Wu；Aryaman Arora；Zheng Wang 等。版本：v3; venue identity verified from NeurIPS proceedings。[原文](https://arxiv.org/html/2404.03592v3)

方法：LoReFT在冻结主干的低维表征子空间学习干预。

边界：低参数不等于无训练；带位置/条件的表征干预不能一般地当作单个永久权重修改。

迁移假设：我们推断：可比较NavThinker未来特征子空间干预；最后线性读出前的无条件仿射变换可能可折叠，应数学核对。

最小验证：相同秩和数据预算比较LoReFT式干预与actor低秩编辑，记录推理路径差异。

正文定位：[§3.2 Eq.(2)–(4)](https://arxiv.org/html/2404.03592v3)。

**E08 | LoRA: Low-Rank Adaptation of Large Language Models**

2021；Edward J. Hu；Yelong Shen；Phillip Wallis 等。版本：v2, 2021-10-16。[原文](https://arxiv.org/html/2106.09685v2)

方法：冻结W，学习低秩BA更新，可合并到原矩阵。

边界：Falcon四动作头本身只有2052参数；低秩增益须对比直接全头拟合而非夸大参数节约。

迁移假设：必要便宜对照，DPed-VLN已使用LoRA适配VLM，因此仅用LoRA不构成新机制。

最小验证：rank1/2头更新与全头ridge/微调，同数据、搜索次数和回合预算。

正文定位：[§4.1 Eq.(3); §7](https://arxiv.org/html/2106.09685v2)。

### 三篇种子论文

**S01 | From Cognition to Precognition: A Future-Aware Framework for Social Navigation**

2025；Zeying Gong；Tianshuai Hu；Ronghe Qiu 等。版本：User-provided ICRA 2025 PDF。[原文](https://arxiv.org/abs/2409.13244)

方法：Falcon：辅助行人预测与社会惩罚训练社会导航策略。

边界：辅助训练信号不等于部署动作必然使用独立预测头。

迁移假设：本项目种子与第一实验载体。

最小验证：匹配官方检查点、场景、配置后复评。

正文定位：[用户提供的正式PDF §III–V；本地research/falcon_source.txt](https://arxiv.org/abs/2409.13244)。

**S02 | NavThinker: Action-Conditioned World Models for Coupled Prediction and Planning in Social Navigation**

2026；Tianshuai Hu；Zeying Gong；Lingdong Kong 等。版本：v2, 2026-03-18。[原文](https://arxiv.org/html/2603.15359v2)

方法：动作条件未来特征融入社会导航策略，并用轨迹预测构造奖励。

边界：跨数据集成功率并非全面超过Falcon；公开默认配置与论文匹配仍待解决。

迁移假设：研究未来特征的可靠性与动作读出。

最小验证：精确配置/检查点齐备后做未来列块干预。

正文定位：[§IV; Tables I/III](https://arxiv.org/html/2603.15359v2)。

**S03 | DPed-VLN: A Benchmark for Socially Compliant Vision-and-Language Navigation in Dynamic Pedestrian Environments**

2026；Haojie Dai；Xiangyi Wang；Liuyi Wang 等。版本：v1, 2026-09-18。[原文](https://arxiv.org/html/2609.21504v1)

方法：动态行人VLN基准与DPet策略，区分普通和先验增强指令。

边界：动态先验的生成信息、版本和奖励权衡需单独审查。

迁移假设：后续迁移与语言冲突试验。

最小验证：先对齐正式模型与数据协议。

正文定位：[Methods; Tables 4/6](https://arxiv.org/html/2609.21504v1)。
