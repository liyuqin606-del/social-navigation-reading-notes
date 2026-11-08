# 算法迁移报告：原始文献与官方实现核验记录

日期2026-11-04。只读检索，无安装、无训练。主报告中的规约、门控小命题和反例为本项目推导；论文有理论条件，不等于本项目满足条件。在线main/stable内容可能变化；实施前需固定版本。

## 原始算法文献

| ID | 核验身份与原始链接 | 用途及阅读边界 |
|---|---|---|
| 1 | Hoang Le, Cameron Voloshin, Yisong Yue. Batch Policy Learning under Constraints. ICML 2019. https://proceedings.mlr.press/v97/le19a.html | 原始会议记录/论文摘要检索；FQE与约束离线学习出处。报告FQE公式为针对随机Falcon四动作的实例化 |
| 2 | Damien Ernst, Pierre Geurts, Louis Wehenkel. Tree-Based Batch Mode Reinforcement Learning. JMLR 6:503–556, 2005. https://jmlr.org/papers/volume6/ernst05a/ernst05a.pdf | 原始论文检索返回作者/题名/摘要；FQI的批量监督拟合思路 |
| 3 | Ilya Kostrikov, Ashvin Nair, Sergey Levine. Offline Reinforcement Learning with Implicit Q-Learning. https://arxiv.org/abs/2110.06169 | 原始摘要；expectile、Q备份、优势加权行为克隆。并读d3rlpy当前官方IQL源码 |
| 4 | Aviral Kumar, Aurick Zhou, George Tucker, Sergey Levine. Conservative Q-Learning for Offline Reinforcement Learning. https://arxiv.org/abs/2006.04779 | 原始摘要；离线外推与保守Q动机；离散目标另以官方实现文档核验 |
| 5 | Romain Laroche, Paul Trichelair, Remi Tachet Des Combes. Safe Policy Improvement with Baseline Bootstrapping. ICML 2019. https://proceedings.mlr.press/v97/laroche19a.html ; https://www.microsoft.com/en-us/research/wp-content/uploads/2018/01/SPIBB.pdf | 原始会议身份、作者预印本返回的概率约束及定理条件。未把表格保证转移到神经特征 |
| 6 | André Barreto et al. Successor Features for Transfer in Reinforcement Learning. NeurIPS 2017. https://arxiv.org/abs/1606.05312 | 本轮重新打开原始摘要；固定动力学、奖励变化、SF/GPI，非任意目标零样本保证 |
| 7 | Grady Williams et al. Information Theoretic Model Predictive Control: Theory and Applications to Autonomous Driving. https://arxiv.org/abs/1707.02342 ; 作者论文 https://homes.cs.washington.edu/~bboots/files/InformationTheoreticMPC.pdf | 原始检索记录；连续信息理论MPC先例。离散迁移限制来自本报告推导与当前实现 |
| 8 | Miroslav Dudík, John Langford, Lihong Li. Doubly Robust Policy Evaluation and Learning. ICML 2011. https://icml.cc/2011/papers/554_icmlpaper.pdf ; https://arxiv.org/abs/1103.4601 | 原始论文身份/摘要；contextual bandit DR。报告不把单步DR当轨迹评估 |
| 9 | David Silver, Joel Veness. Monte-Carlo Planning in Large POMDPs. NeurIPS 2010. https://papers.nips.cc/paper_files/paper/2010/file/edfbe1afcf9246bb0d40eb4d8027d90f-Paper.pdf | 原始论文作者、摘要；粒子belief与生成模型条件 |
| 10 | Aaron D. Ames, Xiangru Xu, Jessy W. Grizzle, Paulo Tabuada. Control Barrier Function Based Quadratic Programs for Safety Critical Systems. IEEE TAC 2017. https://arxiv.org/abs/1609.06408 ; 作者PDF https://xu.me.wisc.edu/wp-content/uploads/sites/1196/2019/08/Control-barrier-function-based-quadratic-programs-for-safety-critical-systems.pdf | 原始作者/题名/摘要；连续系统CBF-QP条件，不移植安全保证 |

## 当前官方文档与实现

| ID | 实际访问位置 | 已核验事实；未核验项 |
|---|---|---|
| E1 | https://d3rlpy.readthedocs.io/en/stable/references/algos.html ; https://raw.githubusercontent.com/takuseno/d3rlpy/master/d3rlpy/algos/qlearning/iql.py | stable明确列出DiscreteCQLConfig、DoubleDQN式TD＋CQL项。官方IQL源码仅IQLConfig/IQL且get_action_type返回CONTINUOUS；未找到DiscreteIQLConfig，不能推断全仓库不存在第三方实现 |
| E1b | https://raw.githubusercontent.com/takuseno/d3rlpy/master/d3rlpy/ope/torch/fqe_impl.py ; https://raw.githubusercontent.com/takuseno/d3rlpy/master/d3rlpy/ope/fqe.py | web与本机urllib只读交叉核验：FQEBaseImpl.inner_update调用self._algo.predict_best_action(batch.next_observations)；DiscreteFQE取单个next_action的Q。不是对随机策略完整动作分布积分。本机实时raw与web行号略有不同，以符号定位。无本地安装测试 |
| E2 | https://docs.pytorch.org/rl/main/reference/generated/torchrl.objectives.DiscreteIQLLoss.html ; https://docs.pytorch.org/rl/main/reference/generated/torchrl.objectives.DiscreteCQLLoss.html | 官方main有DiscreteIQLLoss及categorical输入；DiscreteCQLLoss示例含四动作网络和next done/terminated。文档目录含递归DQN/sequence batch教程；未运行示例 |
| E3 | https://stable-baselines3.readthedocs.io/en/master/guide/algos.html ; https://github.com/DLR-RM/rl-baselines3-zoo | 官方表支持Discrete的DQN/PPO，RecurrentPPO位于contrib；Zoo定位训练与调参框架。不能从此推断具备本项目离线FQE/IQL/CQL接口 |
| E4 | https://raw.githubusercontent.com/pytorch/rl/main/torchrl/modules/planners/cem.py ; https://raw.githubusercontent.com/pytorch/rl/main/torchrl/modules/planners/mppi.py | 当前源码两者描述Gaussian候选；CEM源码有done后reward忽略说明。生成文档MPPIPlanner/CEMPlanner链接本轮404，未因此错误断言源码不存在 |
| E5 | https://github.com/UM-ARM-Lab/pytorch_mppi | 作者团队库README的noise_sigma、控制上下界及连续示例；不是已核验离散类别规划器 |
| E6 | https://vowpalwabbit.org/docs/vowpal_wabbit/python/latest/tutorials/python_Contextual_bandits_and_Vowpal_Wabbit.html | 官方contextual bandit教程与日志接口；未把其能力延伸为POMDP保证 |
| E7 | https://h2r.github.io/pomdp-py/html/api/pomdp_py.algorithms.html | 官方POMCP/POUCT/ValueIteration接口，POMCP为粒子belief，POUCT动作可枚举；未做Habitat集成 |
| E8 | https://gamma-web.iacs.umd.edu/ORCA/ ; https://github.com/snape/RVO2 | 原团队ORCA页明确双方各承担一半避碰责任；RVO2为C++实现。未验证跟踪误差/四动作量化后的安全性 |
| E9 | https://github.com/StanfordASL/cbfpy | 旧作者URL重定向到StanfordASL；官方README确认为Python/JAX CBF/CLF框架。不是当前四动作屏蔽的即插即用实现 |
| E10 | https://github.com/google-deepmind/acme ; https://raw.githubusercontent.com/google-deepmind/acme/master/README.md | 官方README确认模块化研究RL框架；未审计所有算法文件，不作无依据的维护状态或离散IQL可用性结论 |
| E11 | https://github.com/RomainLaroche/SPIBB | 原作者实现入口已核验存在；未安装、未测试当前Python/Habitat兼容性，不把算法原型等同生产成熟栈 |

## 项目事实的来源

前序 ASTRA_MATHEMATICAL_MODELING.md 与本轮读回的内容：四动作、STOP终止、h不保证Markov、随机基座、训练/验证隔离、H8的限制与人群静态代码路径。此轮只新增两份迁移报告，不修改设计、契约、代码、队列或冻结文件。相关实现并未部署。
