# 社会导航：物理、动力系统、博弈与拓扑文献核验

检索截止：2026-11-03（Asia/Shanghai）。实际核验日期为2026-11-03，中途系统时间记录13:44:27+0800；此分支聚焦Falcon、NavThinker、DPed-VLN的可能改进，不执行训练。`physics_records.json`为11篇去重正文证据记录，含2篇经典基础、9篇2024–2026首版/修订/正式版相关工作。登记并处理的聚焦候选为20篇：11篇入证据集，9篇延后/交其他分支；这不是搜索引擎总命中数。

## 优先结论

最值得先做的是**区分预测误差、决策反复和交互模型错配**，再考虑增加结构。力模型、Koopman线性化、能量结构、左右绕行模式、模式保持、安全仲裁，均有直接先例。把其中一个模块接到已有导航器，本身尚不构成足够新颖的论文。

较有操作性的目标是：在固定视觉观测、固定动作空间和有限计算预算内，检验冻结策略的状态是否已经包含足够的风险与模式信息，能否用小改动改善难例而保留原有到达能力。这一目标目前只是研究假设。

## 应怎样使用物理

1. **适合优先约束的量**：速度/加速度、连续性、机器人可执行动作、相对距离和时间到碰撞。它们能与仿真步长、实际动作接口和追踪误差核对。
2. **适合作为软先验的量**：目标速度、人与人排斥、群体吸引、让行倾向。这些取决于目标与环境，必须允许残差和多模态。
3. **不能直接当物理定律的量**：社会势能、人的理性效用、固定避碰责任、所有人的守恒能量。SFM原文自己将“力”解释为动机；人的目标转换、耗散及机器人对人的可见性会改变动力学。参见[Helbing与Molnár，§II–III](https://arxiv.org/html/cond-mat/9805244v1)。

因此暂不优先“给世界模型加一个通用Hamiltonian/PINN loss”。先问约束的是哪一个状态、哪种可检验残差、哪类数据和哪一个安全性质；否则可能只强制模型输出更平滑，同时错误忽略突然转向和意图变化。

## 对三篇种子论文的落点（分析推断）

| 候选机制 | Falcon | NavThinker | DPed-VLN |
|---|---|---|---|
| 运动学可行性/残差 | 先接独立风险重排序；不可只改无推理依赖的辅助头 | 人轨迹decoder或预测风险特征 | 需要先确认DPet代码/动作接口；还要保持语言目标 |
| 低维Koopman动态 | 冻结recurrent feature上做线性风险预测探针 | 对未来latent作动作条件线性残差 | 可以诊断动态遮挡，但不能仅凭预测改善宣称VLN收益 |
| 拓扑模式记忆 | 动作head外的小状态重排序teacher，再测能否蒸馏 | 当前/未来融合前后都可做可解释模式探针 | 通过侧一致性应服从指令目的地，不能只追求平滑 |
| 人行为参数适应 | 仅在行人会反应且有足够观测时有意义 | 可学习belief/类型latent | 对固定轨迹回放，不能把伪互动解释成人会让路 |

上述都是工程/实验建议，不是文献已经证明在三模型上有效。

## 最小实验设计与淘汰条件

**A. 模式一致性：首先作为强基线。** 通过侧记忆应作用于一段轨迹或可解释意图，不宜简单等同于连续选择“向左转”动作。可先用一个冲突行人与左右/先后两个模式做CPU测试，再以真实观测替代oracle状态。比较原actor、普通动作平滑、SHINE式前类代价、固定dwell、证据累积器。增加遮挡导致的ID丢失、突然转向和风险突变；评价切换次数、反应时延、超时与碰撞，而不是只评价少切换。已核验[SHINE §5.3](https://arxiv.org/html/2404.16705v3)和[COMPASS §2–5](https://www.preprints.org/manuscript/202610.0120)都覆盖该机制，后者未同行评审且仅有离线决策核心证据。**若改善完全被普通dwell或动作平滑解释，就不能宣称新机制有效。**

**B. 物理可行轨迹：先小后处理对照。** 用有限加速度双积分器作为候选：

\[
v_{t+1}=\operatorname{clip}(v_t+\Delta t a_t),\quad p_{t+1}=p_t+\Delta t v_{t+1}.
\]

这是建议的离散实现，应与模拟器一致；不是复制某篇公式的保证。比较无约束、平滑、硬投影、软惩罚；控制对突然转向的响应延迟。TPK提供先例及“物理可行性提高、轨迹精度不一定提高”的反例，不能预设加物理必然改善ADE或闭环。[TPK §III-B与Table V](https://arxiv.org/html/2505.06743v4)。**若只减少加速度违例却漏掉更多碰撞风险，应淘汰该设置。**

**C. 低维动力学：证明比普通小MLP多了什么。** 可设\(z_{t+1}=Az_t+Ba_t+r_\theta(z_t,a_t)\)，但这只是我们提议的受控残差模型，不能引用DenseKoopman说它已被证明。先做ridge、同参数MLP、谱约束/耗散变体的等计算对照；测多步误差、风险排序、分布外漂移。DenseKoopman的原实验是像素坐标密集行人预测、best-of-20，不是Habitat闭环。[DenseKoopman §3–4](https://www.ijcai.org/proceedings/2024/0113.pdf)。**若改进只来自额外参数或额外oracle状态，停止“Koopman带来收益”的结论。**

**D. 互动不是回放。** 准备两种相同几何场景：轨迹回放、会响应机器人动作的行人。先对单个人拟合安全距离/让行倾向等低维变量，再考虑UKF或博弈。AToM是这条路径的直接先例，但需要显式行为模型及充分观察；未知目标、身份变化、追踪噪声和策略变化都会削弱可识别性。[AToM §III-B](https://arxiv.org/html/2502.05792v2)。**若机器人行动不改变行人未来，则不要用该测试支持双向互动学习。**

## 问题状态矩阵

| 问题 | 支持/已有解法 | 反证或剩余条件 | 本次判定 |
|---|---|---|---|
| 几何避碰是否就等于社会导航？ | ORCA局部速度可行域 | 互惠与精确感知前提，局部安全不保证活性 | 条件性解决局部避碰，整体未解决 |
| 物理约束是否自动提高预测精度？ | TPK运动学层避免定义内不可行输出 | 同论文精度取舍；标签噪声/急转/阈值不同 | 明确不能概括为普遍提升 |
| 物理+扩散+社会导航是否新？ | NaviDIFF已有该组合 | 对行人不可见、结构假设、与Habitat不同 | 笼统组合已被覆盖 |
| Koopman预测能否改善导航？ | DenseKoopman密集轨迹结果 | 无同一闭环导航证据；有限提升误差 | 可迁移假设，未证实 |
| 左右模式持续是否新？ | SHINE、COMPASS及其引文链 | 后者缺闭环且实现范围有限；需更多外部对照 | 基础机制已覆盖，不宜直接立题 |
| 人会如何适应机器人？ | AToM低维信念更新；GTSFM理性博弈 | 固定回放不含反应；非凸博弈/未知目标/异质人 | 条件性解决部分参数适应 |
| 仿真几何指标改善是否代表人更喜欢？ | GTSFM有量化和主观两类评价 | 76人研究未见显著算法差异 | 不能推导主观社会可接受性 |
| 大轨迹生成模型是否适合低预算？ | STFlow有单卡实验 | 最新版人数据为NBA、训练时长高、无导航闭环 | 借鉴先验，暂不完整复现 |

## 文献版本与证据注意事项

- 11篇的作者、版本、正文定位在JSON；SFM是1995正式论文、1998上传arXiv，不重复计数。SFM对应真实PubMed PMID **9963139**，可合法写NBIB。
- SHINE 2024首版，2025在线发表，2026卷45(1):60–79；JSON按正式卷期2026记year，另存online year。不能把这几个年份当三篇论文。
- Social Mini-Games正式版为2026四作者；早期arXiv索引曾列六作者。采用出版社正式版作者表，与作者项目页相符，不混拼。
- STFlow采用2026-05的v3，新标题与v1不同；NaviDIFF采用v2；AToM采用v2；TPK采用v4。
- COMPASS为2026-10-02发布、v1未审稿，纳入是因为创新冲突直接，不用于证明闭环领先。其已读正文主动划清离线和闭环证据。
- 查询论文主页未见撤稿标记；额外对SHINE/NaviDIFF/DenseKoopman执行correction/retraction/withdrawn反向查询，未出现相关通知。未完成穷尽Crossmark/撤稿库审核，不能声称已排除全部更正风险。

## 检索记录

来源：Web搜索发现；arXiv官方HTML/版本页、IJCAI论文集、Springer正式全文、UNC/UMD作者论文页、Preprints作者上传全文用于取证；PubMed用于SFM元数据。时间过滤：发现重点2024–2026，经典锚点1995/2011；截止2026-11-03。每条web返回的是聚合结果块，服务没有提供逐查询总命中数，故returned_count记录为`not_exposed`，不补造；20是我们实际登记的独立候选数。

完整检索式（依发出顺序，均在2026-11-03）：

1. `robot social navigation physics informed human interaction 2025 2026 arxiv game theory`
2. `social navigation Koopman pedestrian interaction Hamiltonian physics informed 2024 2025`
3. `Optimal Reciprocal Collision Avoidance ORCA social force model pedestrian 1995 paper`
4. `"NaviDIFF" arxiv`
5. `"Game Theory" "Social Force Model" robot navigation GTSFM paper`
6. `"physics-informed" "social navigation" 2025`
7. `"Koopman" "pedestrian" trajectory prediction 2024 2025 2026`
8. `site.arxiv.org "social navigation" "2026" "game"`
9. `site.arxiv.org "physics" "pedestrian" "2025" social force`
10. `"A Game of Social Forces" Galati authors`
11. `"AToM" "2502.05792"`
12. `"A Game of Social Forces" site:link.springer.com`
13. `"Reciprocal n-body Collision Avoidance" 2011 DOI`
14. `"Predicting the unseen" "11577" pdf`
15. `social navigation topology homotopy deadlock 2024 2025 robot paper`
16. `"Multi-Robot Navigation in Social Mini-Games" arxiv`
17. `"social navigation" "topological" "2026"`
18. `"SHINE" "Social Homology" correction retraction`
19. `"NaviDIFF" correction withdrawn`
20. `"DenseKoopman" correction retraction`

沿SHINE和COMPASS相关工作回溯拓扑决策链，发现模式持续、安全优先和固定dwell均已有先例；向数学分支移交BIG-CBF (2609.14343)官方核验。搜索亦出现与机器人无关的同名Navidiff代码review产品，明确排除。

## 9篇登记但未入本分支证据集的候选

| 候选 | 状态/理由 |
|---|---|
| Predicting the unseen: improving robustness in koopman surrogate models for crowd dynamics at a bottleneck, DOI10.14311/APP.2026.57.0149 | 宏观人群密度有启发，但全文网页超时、与个体局部actor距离较远，未入证据集 |
| A Physics-Informed Deep Residual Learning Framework for Pedestrian Trajectory Prediction, DOI10.1109/DSIS67228.2025.11390588 | 只核验摘要，未读正文；不是已验证证据 |
| Pedestrian trajectory prediction method based on social force – Dynamic risk field coupled graph attention network, DOI10.1016/j.physa.2025.131040 | 道路人车数据，全文未取证，暂后置 |
| Alignment Phase Transition in Socially Driven Motion, arXiv2506.01550 | 教室社交取向，不直接等价机器人走路；不移植距离相变阈值 |
| Causal Intervention and Counterfactual Reasoning for Multimodal Pedestrian Trajectory Prediction, DOI10.3390/jimaging11110379 | 已找到官方全文入口但未审方法，交因果路线或后续，不用于现结论 |
| Socially Integrated Navigation: A Social Acting Robot with Deep Reinforcement Learning, arXiv2403.09793 | 与社会reward相关但本轮优先机制更明确者；未完成正文取证 |
| LiveNet, L4DC2025 PMLR283:946–958 | 直接安全/活性相关，属于控制分支，避免重复取证 |
| CoopUQ, arXiv2610.26852线索 | 此分支只见镜像，交root官方核验；不使用镜像为证据 |
| BIG-CBF, arXiv2609.14343线索 | 从COMPASS参考文献发现，交数学分支官方核验；本分支不把引用转述当直接证据 |

## 失败来源与恢复

- 2026-11-03，arxiv插件export_citations(2409.13573/2502.05792/2505.06743/2505.18647)返回HTTP429；改用官方arXiv版本页核验作者和版本，未使用自动生成假元数据。
- IJCAI PDF的web open重定向不可访问；urllib首次SSL EOF；随后curl成功从官方域名下载237万字节量级PDF，PyMuPDF读取§3和§4；原始PDF与文本存`physics_sources/densekoopman.*`。
- ORCA原UNC和迁移UMD的web PDF抓取超时；直接官方UMD下载成功，读取§1/4.2/5；存`physics_sources/orca.*`。
- GTSFM HTML仅摘要/文后部分可见，PDF web读取可得24页但后续find超时；直接Springer PDF下载成功，正文§4.1及§6读取，存`physics_sources/gtsfm.*`。
- 2026宏观Koopman瓶颈论文OJS网页超时，未补充全文，明确不纳入11篇。

## 交付边界

这是一轮有范围的跨学科证据筛选，不是系统综述、不是SOTA复现、也不是对11篇的完整数学正确性审计。所有最小实验均为提案；未申请GPU、未训练、未宣称三篇baseline的SR/SPL/碰撞有任何改善。下一步应由root把本分支与数学、WM、权重编辑合并并去重，再生成统一引用文件。

补充访问记录：SHINE DOI web open 返回不可访问；卷期/在线年份采用作者所属Delft机构库正式稿封面和arXiv版本页交叉核验。SFM PMID再次直接打开官方PubMed，确认issue5与1995May，已补入JSON。
