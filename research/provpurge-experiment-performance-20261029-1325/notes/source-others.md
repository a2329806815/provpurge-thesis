# 其他来源（2026-10-29 读取）

## MemCut arXiv:2610.21405v1 https://arxiv.org/html/2610.21405v1
- 456 手工设计 secrets，证据链全标注；native recovery（走系统自身检索）vs residual recovery（把存活的全部相关对象直接喂同一 reader）；hidden-risk share=(Res-Nat)/Res，删除越深 hidden share 越高（S2 后 39–92% 不等）。
- S0 只删 disclosure、S1 检索排序删 10、S2 LLM 引导从 20 候选删 10：残余恢复 39–76%。
- 候选池 sweep（144 facts 子集）：固定 budget=10，池 20→60 时 TurnRAG/DeepSeek S2 residual 52.1→39.6；固定池=20，budget 10→20 residual 仅 52.08→50.69（-1.39 点）却多删 9.95 个非证据 turn；扩池反而少删 1.04 个非证据 turn 且 residual -12.5 点。
- 132 个 sweep 设置：residual recovery 与“平均删掉的证据 turn 数”近线性，约 -13 点/turn，R²=0.95；给定该数，系统与 write model 影响 ≤~4 点。LLM 判断 S2 相对 S1 只多删 0.7 证据 turn/fact；池 20 时判断增益 5–15 点、池 60 时 20–32 点（判断的价值取决于 reach）。
- 池 20 只含平均 4.5/8.1（TurnRAG/LightMem）或 3.0（MemoryBank）个证据 turn。关系链最难：证据与请求共享实体不共享措辞，相似度排名 3–141；S2 每 fact 只删 2.4 个关系链证据 turn（vs 组装 3.8/聚合 4.6/时间 3.9）。
- 删除一个 turn 会闭包删除其全部派生对象（closure）；派生存储中 S0 的 residual 损失多由 closure 带走（含 disclosure 的派生对象连带证据）——双向提示：级联删能清派生层，但共享对象会连带误伤。

## PolyGraph《Keep Your Receipts》arXiv:2610.17719 https://arxiv.org/html/2610.17719
- 构造时同调用为每个 artifact 发 receipt（support label：若干可替代证据束，任一束足以支撑）；删除=集合算术，无模型调用，24,381 artifact 图 1–21 ms。
- 现行管线只删直接抽取事实：污染查询 32.2%→28.0%（retract-source），predicate revocation 36.8%→36.8% 不变（损伤在派生边与摘要）。
- 整段摘要级 flat receipt 与 re-derivation 一致率仅 33.3%/36.4%（约 2/3 出错，几乎全是“谎称还活着”=泄漏方向）；逐句 receipt 一致率 100%/87.6%，误差双向（4.1% leaky vs 8.3% over-deleted @revoke），且可只删死句、保留 33/35 被触及摘要（5/7 与 28/28）。
- 分层策略（不信任摘要层标签）：污染降至 0.4%/0.8%，recall 达完美维护者的 99.8–100%。receipt 成本：约占构造输出 token 25%、存储比 flat 来源多 ~10% 字节、本图约 $0.30。
- 局限（对自验的警示）：re-derivation 审计测 liveness 而非 groundedness——模型能靠参数记忆再生的 artifact 会被判“活着”；换 oracle/claim-blind prompt 不改变层序。重放/重建不可靠：同一语料两次构建差 29,475 artifact，是删除信号（6,248）的 5 倍。

## STALE arXiv:2605.06527 https://arxiv.org/html/2605.06527v1
- 400 专家验证场景、1,200 题、>100 主题、上下文至 150K；三维探针 SR/PR/IPA × Type I（同属性共指）/Type II（跨属性传播）。
- 最好模型 Gemini-3.1-pro 55.2%（SR Type I 92.0 但 PR 仅 30.0、Type II PR 14.0）；GPT-4o-mini 全上下文 8.7%；记忆框架（同 GPT-4o-mini backbone）：LightMem 17.8、Mem0 8.3、LiCoMemory 7.6、Zep 6.0、A-MEM 5.1。Judge=Gemini-3.1-flash-lite。
- LightMem 诊断（Table 3）：新证据在 SR/PR 中 77.5% 被检索到，但新旧同现 ~71%、旧证据 top-1 84–88%；构造时新证据 top-3 召回含旧证据 60.5%，却只有 3.3% 旧条目被判需更新——“visibility does not imply authority”，失败在裁决不在召回。PR 中“新证据在场仍失败”99.0%。
- CUPMem：同 backbone 8.7%→68.0%，PR Type I/II 78.0/75.0。三组件：①写时 LLM 裁决 KEEP/STALE/REPLACE/UNKNOWN；②拓扑触发传播搜索（候选=Direct∪Affected∪Global_k 有界集，裁决器做最终退休决定）；③受授权状态约束读出（active 才作答题依据、stale 只作历史、unknown 槽位阻断旧前提）。其状态 schema Ω 独立于基准生成本体、评测前固定（防污染范例）。

## PACMI arXiv:2610.05732 https://arxiv.org/abs/2610.05732v1（index 摘要+HTML 片段）
- 来源图+类型化依赖边、四态有效性格、BFS 沿出边级联传播、validity-aware 检索+stale-premise 检查。100 cases/300 queries 五域自建诊断集。
- Final accuracy 0.99 vs A-MEM-style 0.90（配对 exact McNemar 显著，二级来源报 p=2.56e-6）；premise checker 在受控分布上 P/R/F1=1.0。
- 消融：去 premise checking final 0.99→0.83；去历史保留→0.77（历史题 0.36）；去级联错误 3→11 但 p=0.0574 未达 0.05——级联的答案级收益证据弱于状态级，引用时须诚实标注。小样本自建集，外推有限。

## LongMemEval arXiv:2410.10813 https://arxiv.org/html/2410.10813v2 + 仓库 https://github.com/xiaowu0162/LongMemEval
- 500 题；LongMemEval-S 每题约 115K tokens 历史（M 版约 1.5M）。题型量：KU 78、temporal 133、multi-session 133、single-session-user 70/assistant 56/preference 30，另 30 abstention 与各型重叠（二级来源与多篇复现一致；原论文 Table 需再核——报告中标注）。
- KU 定义：识别用户信息变化并动态更新（答当前值）。官方 judge 为按题型 prompt 的 GPT-4o（gpt-4o-2024-08-06），与人工一致率 >97%（二级来源引原论文 meta-evaluation：各型 0.90–1.00、平均 0.98）。
- 已发表设计优化（可迁移到 ProvPurge 读/写）：round 级粒度优于 session 级；fact-augmented key expansion 使 recall@k +9.4%、QA +5.4%；time-aware query expansion 使 temporal recall +6.8~11.3%；Chain-of-Note 阅读策略最高 +10 点。
- 数据版本坑（二级来源）：2025-09 有 cleaned 版替换原发布，报告 LongMemEval 分数必须注明数据版本/hash、judge 与子集口径，否则不可比。

## Supersede arXiv:2606.27472（index 摘要）https://arxiv.org/pdf/2606.27472
- LongMemEval KU 78 题（oracle）：全上下文 92% vs 有界自维护记忆 77%（gpt-5.4，配对 McNemar p<0.005），差距随规模持续；对话长 24× 时 68%→28%，等比扩大记忆容量无恢复（28%→28%）——瓶颈是记忆维护（supersession）而非容量/理解。

## LLM judge 偏差（Zheng et al. 2023 MT-Bench，多篇二级来源一致）
- position bias、verbosity bias、self-enhancement bias；缓解：参考答案、交换顺序、rubric 分解。BEMULA 场景的对应风险：judge 与 backbone/generator 同源（数据由 GPT-5.6-Luna 生成、官方默认 judge 配置示例为 deepseek-flash、agent 亦可为 deepseek-flash）时须做同源性敏感性分析。
