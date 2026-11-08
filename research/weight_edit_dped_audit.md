# DPed-VLN / DPet 权重编辑靶点只读核查

核查日期：2026-11-03（Asia/Shanghai）。没有训练、没有下载模型检查点、没有运行导航评测。本记录区分论文事实、公开开发代码事实及待验证的编辑假设。

## 1. 结论

最值得先检查的是 **actor 的动作输出层**：`action_distribution.linear.weight` 与 `action_distribution.linear.bias`。它把 LSTM 的状态特征直接映射为动作 logits，适合检查过早停止、不必要的前进、左右转振荡等具体失败模式；这些失败模式在 DPet 中是否占主要比例尚未统计，不能预先宣称编辑会提高分数。

这个精确模块名已经在同一作者的公开开发仓库中静态核验，但尚未用论文最终检查点核验。完整 `state_dict` 键可能另含 `actor_critic.` 等外层前缀。不能把开发仓库直接等同于论文正式发布。

## 2. 发布状态与证据边界

- [正式论文入口仓库 tju-dhj/DPedVLN](https://github.com/tju-dhj/DPedVLN)：API 核查根目录只有 `README.md`。README 明确说明内部仓库正在清理、代码稍后发布；承诺提供 DPet RL/IL 和评估代码。没有检查点链接；releases 列表为空。
- [README](https://github.com/tju-dhj/DPedVLN/blob/main/README.md) 链接数据 DOI `10.5281/zenodo.22816620`。本次网页工具未能打开 Zenodo，所以仅确认仓库给出了链接，没有核验数据包内容。README 称数据有回合配置、双级指令、专家轨迹，场景资产另行取得。
- 同一作者另有公开[开发仓库 tju-dhj/DPed-VLN](https://github.com/tju-dhj/DPed-VLN)，当前 main 为 `54ceed00efeabcd7ea4e8ba69e57d072679572c0`，提交时间 2026-09-16。README 仍是 GitLab 模板。该树含 4、5、6 动作及多种实验配置，不能任选一个配置便声称复现了论文。
- 开发仓库完整递归树中未找到 `.pt` / `.pth` / `.ckpt` / `.safetensors` 权重文件；releases 为空。评估配置指向作者服务器本地检查点路径，不是公开下载地址。本次未发现可直接下载并确认对应论文表 2/3 的 DPet 训练权重。

## 3. 推理图与精确靶点

代码固定版本均为上面的 commit。

1. [4 动作 start-relative 评估配置](https://github.com/tju-dhj/DPed-VLN/blob/54ceed00efeabcd7ea4e8ba69e57d072679572c0/habitat-baselines/habitat_baselines/config/DPed_pro/new_data/eval/dped_eval_4a_start.yaml#L13-L16) 指定 stop、move_forward、turn_left、turn_right。L161 为 `PointNavResNetPolicy`，L182 为 hidden_size 512，L208–209 为 2 层 LSTM。
2. [NetPolicy 创建动作头](https://github.com/tju-dhj/DPed-VLN/blob/54ceed00efeabcd7ea4e8ba69e57d072679572c0/habitat-baselines/habitat_baselines/rl/ppo/policy.py#L252-L289)：`CategoricalNet(self.net.output_size, self.dim_actions)`。
3. [CategoricalNet 定义](https://github.com/tju-dhj/DPed-VLN/blob/54ceed00efeabcd7ea4e8ba69e57d072679572c0/habitat-baselines/habitat_baselines/utils/common.py#L85-L96)：唯一参数层是 `self.linear = nn.Linear(num_inputs, num_outputs)`，直接构造 logits 分布。
4. [策略前向](https://github.com/tju-dhj/DPed-VLN/blob/54ceed00efeabcd7ea4e8ba69e57d072679572c0/habitat-baselines/habitat_baselines/rl/ppo/policy.py#L324-L349)：特征分别送动作分布与 critic，动作由分布采样或 mode 得到。
5. [LSTM 输出](https://github.com/tju-dhj/DPed-VLN/blob/54ceed00efeabcd7ea4e8ba69e57d072679572c0/habitat-baselines/habitat_baselines/rl/ddppo/policy/resnet_policy.py#L1051-L1059)：拼接特征进入 `state_encoder`，其输出成为上述 actor 特征。

据此，该 4 动作 / 512 隐维配置的动作层**预期**是 `W ∈ R^(4×512)`、`b ∈ R^4`，共 2,052 个参数。拿到实际 checkpoint 后仍需核验参数形状、动作 index 顺序、外层键前缀和观测配置，不能仅凭 YAML 顺序硬编码动作行号。

建议顺序：

| 靶点 | 为什么可尝试 | 必须先满足的证据 |
|---|---|---|
| actor weight 的少量方向/列，以及特定动作行 | 能做状态依赖的动作纠偏，直接影响部署策略 | 同一状态下确有可定位的错误偏好；编辑前后受控干预有效 |
| actor bias | 极低成本基线，检查全局动作偏好失衡 | 同时报告误停、漏停、碰撞、路长与超时；不可仅提高一个分数 |
| `net.visual_encoder.cross_modal_attention.out_proj.weight` 或 V 投影块 | 若语言分量过强/过弱，可影响语言信息写入视觉残差 | 先证明失败来自该融合路径，不是识别能力/记忆能力不足 |
| `net.state_encoder.rnn.*` | 可能影响重复转向及记忆保持 | 影响更广，需先定位门/方向；不建议作为第一步盲改 |

STOP 是结束回合，不是 WAIT。论文的 4 动作设置没有独立等待；开发树虽含 pause/backward 扩展，也不能直接加入论文对照中而不声明改变动作空间。

不推荐只改 `critic.fc.*`：此代码中 critic 是独立价值分支，冻结策略且只做部署推理时，改它不会改变动作分布。不推荐仅改用于辅助监督的行人数/位置/轨迹头，除非实际计算图表明它们输出被送入 actor；辅助训练损失有作用不等于其头在测试时驱动控制。

## 4. 一个与“选哪块 attention 权重”直接有关的静态发现

[开发版 resnet_policy.py L1815–1840](https://github.com/tju-dhj/DPed-VLN/blob/54ceed00efeabcd7ea4e8ba69e57d072679572c0/habitat-baselines/habitat_baselines/rl/ddppo/policy/resnet_policy.py#L1815-L1840) 把一条完整指令编码为 `[B,D]`，再 `unsqueeze(1)` 成为 `[B,1,D]` 的唯一 key/value token；视觉也是一个 query token。这里是标准 `nn.MultiheadAttention`，默认 dropout=0（L1317–1321）。

数学上，key 长度为 1 时，对单个数的 softmax 恒为 1。因此正常有限数值条件下，该 attention 的输出对 Q/K 投影数值不敏感；编辑 `in_proj_weight` 的 Q/K 两段无法让它“更关注行人词”，因为词已经池化成一个向量。V 投影、`out_proj`、后续 `output_projection` 和视觉残差仍影响输出。这是代码路径的静态性质，**不是论文最终实现已确认的缺陷**，也不是经过导航实测的性能归因。

这进一步支持先选 actor 输出层；如果第二阶段研究融合，须先核验正式实现的 token 粒度。不能照搬 LLM 里“改某个 attention head”的叙事。

## 5. 能瞄准什么分数

来源为用户提供的论文 PDF 表 2、3、6，并用[官方 arXiv HTML](https://arxiv.org/html/2609.21504v1)交叉检查。以下是论文自身评测中的对照，不能声称截至今天的整个 VLN 领域排行榜。

| 训练与评估条件 | SR ↑ | SPL ↑ | STL ↑ | PSC ↑ | HCR ↓ |
|---|---:|---:|---:|---:|---:|
| DPet-RL level-1→level-1 | 37.13 | 33.36 | 36.83 | 92.52 | 42.45 |
| DPet-RL level-2→level-2 | 38.55 | 36.35 | 37.97 | 92.88 | 36.70 |
| DPet-IL level-2→level-1 | 22.69 | 18.39 | 20.58 | 94.36 | 28.23 |
| StreamVLN LoRA FT level-2 | 17.26 | 13.56 | 14.98 | 94.15 | 26.06 |
| DPet-RL level-2 without SCP | 36.59 | 35.80 | 36.13 | 93.70 | 30.29 |

DPet-RL 是论文报告方法里 SR/SPL/STL 最好的方法，但不是各项安全指标最佳。最有价值的编辑目标是改善成功—安全折中，例如在相同指令、场景与动作空间下保持 SR/SPL，同时降低 HCR；不能用更早停止或原地转圈刷低碰撞率。

本次检索未找到可以据以证明 DPed-VLN 最新外部 SOTA 的正式排行榜或更新的第三方基准结果。因此建议称“挑战论文中最强 DPet-RL 对照”，不要称“已找到通往 SOTA 的确定权重”。

## 6. 最小验证建议（尚未实施）

先取得可验证原始检查点，并复现其对应条件。只使用训练场景里的独立开发回合诊断与选择编辑，预留未见场景作最终比较。按失败原因分组：过早 STOP、到达后不停、转向循环、面对可观测行人仍前进、语言目标错误。只有前几类确实占可观测损失，且 actor 特征含有纠正所需信号时，动作层编辑才有可信的收益通道。

需要对照：不编辑、bias-only、随机同范数/同稀疏度编辑、同预算参数搜索、提出的定向编辑；若声称优于低成本适配，还应加 head-only 常规优化。统计 SR、SPL、STL、HCR、PSC、误停/超时率，使用配对场景与回合的置信区间。单个固定场景修复不能代替泛化结果。
