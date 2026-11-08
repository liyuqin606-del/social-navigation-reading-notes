# Falcon / NavThinker 权重编辑靶点核查
核查时间：2026-11-03。范围：官方代码静态只读、README 与论文笔记；没有下载 checkpoint、执行模型或运行实验。候选修复效果均未验证。

## 官方材料及可得性

- Falcon 官方仓库：https://github.com/Zeying-Gong/Falcon
  - 本次 main commit：d15ef512c4b1451f9daeaf6d4d7ecc702ed57430。
  - README 提供 pretrained models 目录：https://drive.google.com/drive/folders/1Bx1L9U345P_9pUfADk3Tnj7uK01EpxZY?usp=sharing 。
  - web 工具未能读取该 Drive 目录；只确认官方提供此链接，未核实文件是否可实际下载。
  - `habitat-baselines/habitat_baselines/config/social_nav_v2/falcon_hm3d.yaml` 第 98 行指向 `pretrained_model/falcon_noaux_25.pth`。
  - 同配置第 13–16 行启用四动作，第 17–18 行 backward/pause 被注释；第 149 行 hidden_size=512，第 173–174 行 LSTM、2 层。
  - README 提供 Linux Habitat-Sim 0.3.1 + Bullet 依赖、HM3D/MP3D 场景与SocialNav episodes。完整训练声明 4×RTX3090 约 2 天。本次不涉及训练授权或执行。

- NavThinker 官方仓库：https://github.com/hutslib/NavThinker
  - 本次 main commit：fc3421b6991080a367a348f7f80fa899358f9a0b。
  - README 明确代码仍整理中，预训练 policy+world-model checkpoints 仍在 To do，端到端安装验证亦未完成。
  - 因而目前不能声称可直接取得官方最优 checkpoint 并编辑。

## 第一靶点：Falcon 动作输出层

**代码事实。** policy 内的相对参数名为：
- `action_distribution.linear.weight`
- `action_distribution.linear.bias`

[CategoricalNet 定义](https://github.com/Zeying-Gong/Falcon/blob/d15ef512c4b1451f9daeaf6d4d7ecc702ed57430/habitat-baselines/habitat_baselines/utils/common.py#L85-L96)
由 `nn.Linear(num_inputs,num_outputs)` 输出 categorical logits。
[NetPolicy 构造和 act](https://github.com/Zeying-Gong/Falcon/blob/d15ef512c4b1451f9daeaf6d4d7ecc702ed57430/habitat-baselines/habitat_baselines/rl/ppo/policy.py#L252-L351)
明确 actor 的输入是 net 输出，选动作由 mode/sample 完成。

按当前官方 eval 配置推导，预期 W 为 [4,512]、b 为 [4]，共 2052 个参数。**这是源码与配置推导，未加载 checkpoint 检查；真实 state_dict 可能有 agent/actor_critic 等前缀。**动作语义需从加载后的 action mapping 取，不能凭行号假定某一行。

**建议假设。**先采集重复转向、对障碍反复前进、已接近目标却不停止等明确失败，再检查 hidden feature 是否已携带足够信息而输出决策读错。若有，编辑 W 的少数方向或对应动作行，保留其他网络。单纯全局 bias 改动应作为必要廉价对照，不应自动宣称机制创新。错误感知/缺少行人预测不太可能仅靠最后一层补齐。

**不能直接改成“让路等待”。** [DiscreteStopAction](https://github.com/Zeying-Gong/Falcon/blob/d15ef512c4b1451f9daeaf6d4d7ecc702ed57430/falcon/additional_action.py#L38-L75) 设置 `should_end=True`；Pause 类虽存在，当前四动作评测未启用。提高 STOP logit 会终止 episode，并非 WAIT。

**训练支路辨别。** [act](https://github.com/Zeying-Gong/Falcon/blob/d15ef512c4b1451f9daeaf6d4d7ecc702ed57430/habitat-baselines/habitat_baselines/rl/ppo/policy.py#L324-L351) 不调用 `aux_loss_modules`；辅助任务只在 [evaluate_actions](https://github.com/Zeying-Gong/Falcon/blob/d15ef512c4b1451f9daeaf6d4d7ecc702ed57430/habitat-baselines/habitat_baselines/rl/ppo/policy.py#L361-L401) 计算损失。纯推理时编辑人数/位置/轨迹辅助输出头不会改变 action。critic 虽在 act 里计算，但其输出未进入动作选择，纯编辑 critic 也不能直接修复当前策略。

## 第二靶点：NavThinker actor 对未来特征的读取列块

**代码事实。**
[每动作未来特征编码](https://github.com/hutslib/NavThinker/blob/fc3421b6991080a367a348f7f80fa899358f9a0b/habitat-baselines/habitat_baselines/rl/world_model/navthinker_policy.py#L267-L342):
`net.lookahead_per_action_fc.0.weight` 是无 bias 的 Linear；其后是 LayerNorm + SiLU。
`_run_lookahead` 对每个候选动作进行一步预测，再平均 patch 的 deter 特征。
[晚期拼接](https://github.com/hutslib/NavThinker/blob/fc3421b6991080a367a348f7f80fa899358f9a0b/habitat-baselines/habitat_baselines/rl/world_model/navthinker_policy.py#L544-L556) 为
`[net.rnn_out_ln(rnn_out),lookahead_feat]`，
随后传入同类 `action_distribution.linear`。

当前 config 为 hidden_size=512、4动作，lookahead_dim 默认128，预期 actor W=[4,1024]、b=[4]；其中 W[:,512:] 对应四组未来特征，2048个标量。
[lookahead_dim 默认及配置解析](https://github.com/hutslib/NavThinker/blob/fc3421b6991080a367a348f7f80fa899358f9a0b/habitat-baselines/habitat_baselines/rl/world_model/navthinker_policy.py#L745-L770)
同样仅为源码/config 推导，需真实 checkpoint 加载验证。

**建议假设。**针对零样本环境中预测误差放大，先测 future-feature 列块消融/有控制的缩放，再定位特定错误预测方向，编辑 actor 对这些方向的响应；相对直接更改整个 DA-V2/world-model backbone 更易归因。保持 W[:,:512] 不变可以作为局部性约束，但整体性能保护仍需实测。

**数学提醒。**不要把整块 `lookahead_per_action_fc.0.weight` 乘正数当有效“未来信息降权”：后面的 LayerNorm 会抵消大部分整体尺度变化。对 actor 里的未来列块缩放才会直接改变其 logit 贡献。若需按状态置信度动态门控，新加 gating 网络属于架构/推理修改；没有现成可解释特征时不能只靠固定权重缩放宣称实现了不确定性感知。

**现有证据边界。**NavThinker论文同表 Social-MP3D SR47.33/SPL41.71 低于 Falcon51.67/45.54，是寻找跨域故障的动机；不能据此认定融合权重就是原因。公开代码 README 明言仍清理中，当前 eval YAML 选择 navthinker encoder/LSTM，与论文所叙架构不能未经复现就视为一一对应。未来候选应先完成匹配 checkpoint/config 的基线复现。

## 接近 SOTA 的合格判断

以同一 benchmark、同一观测、动作集、episode 划分及采样方式进行成对评估，同时报告 SR/SPL/H-Coll/PSC 和超时/停止错误。编辑/阈值选择只接触开发集，最终留出场景不用于反复试编辑。至少比较原模型、bias-only/temperature、小预算普通调优、随机同稀疏度编辑。局部修复成功不等于所有指标领先；博客式机制是否迁移到该任务只能由闭环实验回答。

