# FEB-30 与 TRACE 核查（均读到全文主要部分，超「摘要+方法概览」要求）

## FEB-30 — Beyond Downstream QA, arXiv:2610.22209v1（2026-10-15，Toma et al.，AWS/Cluj）
- URL: https://arxiv.org/html/2610.22209v1 ；读取 2026-10-29 index 全文（正文 §1–8 + 结果表；附录未逐项读）。
- 主张：下游 QA（extrinsic）混淆记忆/检索/生成，产生假阳（LLM 补偿）与假阴（检索瓶颈）；提出 intrinsic evaluation=每步摄入后直接查记忆库对 ground truth。
- FEB-30：30 条 fact-evolution trajectories × 6 类（C1 LLM 补偿/C2 检索瓶颈/C3 分类/C4 合成/C5 时间/C6 多作者）× 3 难度（L1–L3 各 10）；paradigm inversion：先定 trajectory（states、atomic claims、docClaims、期望动作 create/update/reinforce/conflict/supersede）再生成文档→确定性 GT、无人工标注；共 147 文档（127 状态+20 噪声）；生成=6 模型轮转（单模型 ≤30%）+ 3 judge 一致验证（entail∧¬halluc）；L3 用真实文档 hybrid injection（~90% 真实文本）；人工复核 11% 内容修正、28% 去脚手架。
- 内在指标：claim coverage（embedding>0.8 + LLM entail 判定）、fact accuracy、update detection recall、false update rate、fact count、confidence MAE；外在=诊断 QA（YES/PARTIAL/NO）；MAD=|Acc−Extr| 均值，DivRate=分歧>0.15 的比例。
- 被测系统：代表性 extract-embed-classify-synthesize 系统（非贡献本身）：LLM 分类 create/reinforce/update/supersede、temporal guard（旧 observation 的 update/supersede 降级为 reinforce）、confidence ±、fact 全历史+supporting observations 链接（即 observation→fact 来源链接）、BM25(0.3)+kNN(0.7) 混合检索、temp=0。
- 结果（11 runs）：内在 60.6% vs 外在 48.8%，MAD 0.168±0.025，DivRate 43.6%（摘要约数 44%），pass/fail 不一致 21±4%、κ=0.118、r=0.423（r²≈0.18）；动作分类准确率仅 20.1%（L1 33.8→L3 10.1），FUR 22.2%，UDR 73.6%；C6 内在最低 41.5%、miss 87.5%。四配置稳健性：MAD 0.158–0.181。
- 局限（自述）：合成文档效度待真实语料验证；30 条只够诊断不够细粒度统计；评分依赖 embedding 阈值+LLM judge 有噪声；不测 episodic/procedural/单轮回忆。
- 代码：论文脚注称 https://github.com/amazon-science/feb-30-benchmark ——2026-10-29 研究窗口实开返回 GitHub 404（未登录状态）。结论：论文声称释放、实际未获取/打不开（截至核查日）。

## TRACE — arXiv:2609.33517v1（2026-09-27，Xiong et al.，上交/MemTensor/UC Berkeley）
- URL: https://arxiv.org/html/2609.33517v1 ；仓库 https://github.com/xiong-wenjun/TRACE
- 读取 2026-10-29：index 全文正文 §1–7（方法/实验/消融/局限全读，附录未逐项）；仓库 verified live（Public，2 commits，最新 2026-10-01，src/trace、data、results、tests 齐，MIT）。
- 问题：temporal memory admission——返回的 agent 还能对哪些保留记忆采取行动；记忆可检索/相关/忠实来源但已不可采（inadmissible）。不是删除：functional forgetting=被拒状态可留在 audit store，只是不准作新工作负载的前提。
- 方法（training-free 层，位于记忆后端与 agent 之间）：①捕获 return-cycle checkpoint D_a（可见状态+source receipts+epoch），私有 episodic bank 隔离、每条绑其源 work-state item；返回时 C=D_a⊕Δ。②失效解析：显式（epoch/validity/supersession/revocation 链接，确定性规则）+ 隐式（高召回 scan 提候选依赖对 → grounded verifier 逐对判是否失效，用 LLM 判定、只看 checkpoint 证据不用标注）。③四门验证 E={A∧T∧U∧P}：authorization / temporal / applicability / provenance（P=须有 bound receipts，只记来源派生不证真）。④义务覆盖选择：贪心近似带预算集合覆盖（Cov≥γ、Crit≥ρ、|S|≤B_n、字符≤B_c），覆盖不了则 reset/block（fail closed）。⑤签名 Return View：含 return-cycle id、principal、role、task、epoch、expiry、item digests、coverage report；暴露前重算 digest 并复查谓词；私有经验仅当其绑定的 work-state item 在签名视图内才 readmit。
- 数据结构：work-state item 带 content、source receipt、epoch+validity interval、principal/role scope、task purpose、dependency links、supersession status。
- 评测：五 agent return 协议（1 返回+4 活跃），三 actor（Qwen3.5-122B-A10B、Gemini-3-flash、DeepSeek-v4-flash）；Memora（显式替换）、STALE Type II（隐式失效）、ManBench-Return（自衍生，传播冲突）；指标 Overall（各基准原生）、VIA（有效可用）、IIR（无效拒绝）、WSAR=1−IIR、STALE 上 LS（三探针全过）。
- 结果：ManBench-Return VIA 92.6–98.3% / IIR 98.4–99.5%（唯一双高；Restore/Reset 各有一项触 0 的对侧失败）；STALE Type II Overall 超最强对比策略 +22.3(Qwen)/+18.5(Gemini)/+27.5(DeepSeek)，Qwen LS 18.0→39.0%，token ~2.3×（499,659/episode）；写时整合 CUPMem 更准（Qwen 子集 Overall 80.56 vs TRACE 58.89）但 token 3.99× TRACE；Memora 上 TRACE 无一致增益（FAMA 40.14/42.80/42.58 vs MemTX 42.14/44.12/44.72）——作者明说显式替换场景该层不必要。
- 消融（STALE/Qwen）：去 implicit-invalidation Overall −25.83pt、IIR −31.00pt（最大）；OAS 换 relevance ranking →38.67%；去 complete-view gate →47.33%；去 source-bound private readmission →42.00%（全量 52.33%）。规模 4–32 agents 稳定（波动 ≤0.36pt）；四后端（ScopedMem/Mem0/Memobase/A-MEM）可移植，IIR +0.39~6.42、VIA −0.05~−3.03。
- 局限（自述）：假设 authenticated return、保留 departure checkpoint 与 source receipts；隐式失效依赖候选覆盖与 verifier 校准（漏检放行陈旧、过宽丢有用）；absence 越长越差（STALE Early 73.33→Late 43.33，token 316,750→664,860/episode）；ManBench-Return 为衍生集；token 分析只有总量、无 latency。
