# ProvPurge 增量创新扫描（2026 下半年，重点 2026-08～10）

> 任务：为已收敛的核心内核「probe-gated 硬删事务」（句级来源图给闭包候选 + 实体扩展补可推导证据，删除以预承诺攻击探针集在线全不过且兄弟保全全过为提交条件，不过自动扩候选重删，回执签探针成绩单）寻找尚未被占领、可组合的增量空间。
> 约束：主基准 BEMULA（RPR/Score），辅助 LongMemEval-S 知识更新子集 + STALE，MemCut 只作残余恢复辅证；硬件 5×RTX 3090、本地推理、无训练大模型预算。
> 已确认占领、本报告不再推荐为创新点：版本化/双时态事实库（Zep、ChronoMem）、墓碑+级联失效（PACMI）、自建删除后基准（BEMULA、MemCut、ForgetEval）、签名删除回执本身（LETHE）、写入准入控制（TRACE）、原生/残余恢复测量口径（MemCut）、下游 QA 与内在指标分歧（FEB-30）。
> 证据纪律：【已证实】= 本次实际打开论文摘要页/全文 HTML/PDF 或仓库论文并读到原文；【index】= 仅检索结果片段，未开全文；【分析推断】= 本报告基于已证实事实的推演，不是来源结论。查不到的写「未获取」。读取日期均为 2026-10-30。

## Summary

**一句话结论：2026 年 8–10 月这个方向不是变空了，而是发生了「基准与审计的集中爆发」——10 月单月就落了 BEMULA（10-13）、PolyGraph（10-13）、FEB-30（10-15）、MemCut（10-15）、AfterDeleteBench（10-22）五篇直接相关工作，9 月还有执行状态遗忘、遍历安全删除、分词绕过三篇。留给 ProvPurge 的增量空间因此不在「再造一个删除机制/基准」，而在探针集本身与回执的可证伪性上。**

排序结论（详见 §4）：

| 级别 | 候选 | 一句话理由 |
|---|---|---|
| **值得做第二贡献（二选一或组合）** | C1 留出探针泛化缺口（预承诺 + sealed holdout） | ProvPurge 独有的结构性风险——删除器在闭环里反复对同一探针集优化，天然产生探针过拟合；全场没有任何工作测过这个缺口 |
| | C2 回执可验证性评测（LETHE 路线图 v0.6 的落地） | LETHE 论文路线图原文自承规划未做；且 ProvPurge 的回执是「探针成绩单」而非删行清单，验证对象天然不同 |
| **值得做（强化内核/分析章）** | C3 可推导覆盖律：用 cut 完备度取代删除计数 k | MemCut 的 `Res≈13×(8.2−k)` 是为「删不全的 baseline」拟合的；ProvPurge 是第一个声称能完成 cut 的删除器，有资格也有数据检验「律在 cut 完成时是否变成阶跃」 |
| | C4 可归因泄漏门控（对照校准的 Δ-commit） | 把 BEMULA/AfterDeleteBench 的配对对照从「评分条件」搬进「事务提交条件」，解决探针因合法可推断/参数知识而永远不过 → 无限扩候选的问题 |
| **锦上添花（低成本切片，别当贡献写）** | C5 探针强度证书（攻击预算曲线 + 换读者迁移）；C7 删除后生命周期耐久再探；C6 回执自身泄漏审计（宜并入 C2） | 都是在已有主实验日志上加一维重测，便宜、能挡审稿人，但单独撑不起贡献点 |
| **坑（不要做）** | 执行状态/KV-cache 重放、参数通道遗忘、遍历安全向量删除、多模态通道、多主体治理、摘要逐句再派生、把分词通道当卖点、再自建一个基准、声称可证明 (ε,δ) 式认证删除 | 逐条撞车或成本失控，见 §4.3 |

特别机会 (a)(b)(c) 的核查结论：(a) **属实且仍空**——LETHE 路线图原文写明 v0.6 才做 receipt-verification family，本次扫描未发现任何已发表的删除回执可验证性评测（最接近的 TSD-AUDIT 验的是向量遍历过程、PolyGraph 验的是构造来源回执，都不是行为回执）；(b) **探针设计空间大半未被占领**——已有工作全是「固定探针集测别人」，没有「删除器对着探针集迭代时探针如何防过拟合」的处理，这是 ProvPurge 闭环结构独有的问题；(c) **能做自己的版本，但必须换自变量**——照抄 k（删除计数）必被 MemCut 压死，换成 sufficient-set 覆盖完备度才有经验内容，且要警惕「定义性恒真」的陷阱（见 C3）。

---

## Findings

### 1. 2026 下半年新工作扫描与撞车地图

#### 1.1 时间线（【已证实】除标注外均打开了 arXiv 摘要页或全文）

**2026 年 8 月**

- **StateAuditor**（arXiv:2608.01619，2026-08-02）：STALE 的 implicit policy adaptation 缺口——记忆更新了但行为没变。它从 stored state → draft 反向审计，只让经来源与时序验证的 old→new 转换触发修复；在 STALE 全协议（400 场景）strict VTA 0.736 vs 锁定前任 0.686。URL: https://arxiv.org/abs/2608.01619
  - 对 ProvPurge 的含义：STALE 方向的「更新后行为修复」已被占；ProvPurge 在 STALE 上只能作删除/更新的被测系统，不要顺手做 draft-repair。
- **ChronoMem v2**（arXiv:2607.27773，v2 2026-08-05）：语义版本控制/回滚。任务书已列占领，此处仅确认其 v2 仍在活跃更新。URL: https://arxiv.org/abs/2607.27773v2

**2026 年 9 月**

- **Forgetting Without Restarting: Execution-State Unlearning**（arXiv:2609.04875，2026-09-04，已开摘要页）：形式化「行为如同从未见过目标」的反事实标准，证明精确遗忘至少需 `T−τ+1` 次重计算（τ 为注入步），Provenance-Guided Selective Replay 达到该界，跨 prompt/压缩摘要/KV cache 三层；普通记忆删除泄漏不变，指令式遗忘在 elicitation 下 `Leak@probes = 1.00`。URL: https://arxiv.org/abs/2609.04875v1
  - 含义：这是 9 月最危险的一篇——它占领了「删除要管衍生状态 + 下界 + 选择性重放」的理论叙事，且其探针是 elicitation/stochastic/string-free 行为测试。ProvPurge 不要碰 KV cache 与重放，也不要再声称「首个给出删除成本下界」。
- **Does Your Agent's Memory Survive a Model Upgrade?**（arXiv:2609.05339，2026-09-04，已开摘要页）：同历史换 writer/reader，模型写的 NOTES 迁移后 ±(+9.91 / −13.28) 个点，固定 schema KG 仅 +0.0004。URL: https://arxiv.org/abs/2609.05339v1
  - 含义：「结论依赖哪个模型读写」已有受控证据，是 C5 换读者审计的立论依据。
- **TSD-AUDIT / Beyond Private Training**（arXiv:2609.19456，2026-09-16）：区分 output safety 与 **traversal safety**——HNSW 删除后返回结果不含已删 id，但遍历时仍对已删向量计算距离；其框架强制 alive-before-scoring 并发 **per-query scored-trace 证书供独立验证者对照删除快照核验**。URL: https://arxiv.org/abs/2609.19456
  - 含义：「可独立验证的删除证书」在向量索引层已被占领，且威胁模型明确是 honest-but-buggy、非伪造证书的恶意运营者。C2 必须与之划界（见 §3a）。
- **The Tokens Remember**（arXiv:2609.29045，2026-09-24）：同一字符串的替代合法分词可绕过知识编辑/参数遗忘，38.6% 替代分词恢复编辑前回答，Toketive 攻击无需原模型。URL: https://arxiv.org/abs/2609.29045v1
  - 含义：对象是参数层编辑/遗忘，不是外部记忆存储层；但它把「探针只用规范表述测」钉成已知盲区。可作 C1 的一个留出变换族，不能当 ProvPurge 的主卖点（外部记忆里分词变体大半塌缩为 ForgetEval-Adv 已有的 identifier obfuscation 类别）。
- **Stashbird**（arXiv:2609.34242，2026-09-28，v2 2026-10-17）：episode-grounded provenance reconciliation——删除时移除失去全部支持的关系、保留仍有其他支持的关系，263 个删除干预全过定义检查。URL: https://arxiv.org/abs/2609.34242
  - 含义：「多支持则保留、单支持则删」的支持计数对账在对话记忆上已被占领（与 PolyGraph 的替代证据束同构）。ProvPurge 的句级来源图若往这个方向写创新，会正面撞这两篇。
- **TRACE**（arXiv:2609.33517，2609 系）：把 re-entry 时的记忆准入形式化为 temporal memory admission，返回时只释放经校验的 Return View。URL: https://arxiv.org/abs/2609.33517v1【index 级日期，为 9 月编号】
  - 注：此 TRACE 是「返回时准入」，与任务书所述「写入准入控制」同属 admission 族；无论哪种读法，admission 叙事都不要再碰。
- 另有二手报道（【index，未开 arXiv 全文】）：2026-09-24 的工作称本地编码 agent 可删除自己的执行 traces、监视器约一半情形漏检。对 C2 的启示是「回执/日志若在被审计系统可写域内则不是证据」，本报告只把它作背景，不作引用级证据。

**2026 年 10 月（爆发月）**

- **PACMI**（arXiv:2610.05732，2026-10-05）：四态 validity lattice + 来源级联失效。任务书已列占领。URL: https://arxiv.org/abs/2610.05732v1
- **The Right Memory in the Wrong Context**（arXiv:2610.07309，2026-10-05）：retrieval admissibility 验证——相关但属于他人/违反政策/生命周期状态不符的记忆不可进 prompt；分 admissible/inadmissible/unresolved 三态、matched recall 比较。URL: https://arxiv.org/abs/2610.07309
  - 含义：多主体/政策准入验证已被占领，C 类候选不要往治理方向扩。
- **MemForm**（2026-10-09，【index，仅 arXiv 检索页片段，未开全文】）：记忆形成投毒基准，攻击面在记忆形成接口。与删除正交，仅记录存在。
- **BEMULA**（arXiv:2610.18052，2026-10-13，已开全文核指标）：50 对话/135 请求/1742 题；五套件 retention/efficacy/preservation/relearning/attack，其中 attack 套件是 **5 个固定攻击帧（indirect questioning、role playing、jailbreak prefixes、Boolean guessing、multi-hop reasoning），原文明确「从不改写这些帧，因为其措辞就是攻击向量」**；指标 `RPR` = 一个请求的所有问题 ∧ 该实例全部 preservation 问题全对，`Score = FA×UG，FA=√(FE·AR)，UG=√(PR·RLA)`；所有系统 RPR ≤ 1.5%。URL: https://arxiv.org/html/2610.18052v1
  - 含义（关键）：BEMULA 的攻击帧是公开且固定的——任何在 BEMULA 上迭代调删除的系统，都存在对这 5 帧过拟合的通道。这正是 C1 的靶子，而且靶子是基准自己写明的。
- **Keep Your Receipts / PolyGraph**（arXiv:2610.17719，2026-10-13，已开摘要/HTML）：KG 构造器在生成 artifact 时同时发出「替代证据束」receipt（每束自称单独充分），清理退化为集合运算、零模型调用；因 receipt 是模型自报，用 **re-derivation audit** 核验（删掉所引证据、看构造器是否还生成该 artifact）：merge receipt 与重派生一致率 68%/85%，整篇摘要的 flat receipt 约 2/3 出错，**逐句 receipt 忠实度 100%/88%**，且逐句清理能保住 35 篇受影响摘要中的 33 篇。URL: https://arxiv.org/abs/2610.17719v1
  - 含义：双重撞车预警——(i)「receipt」一词在同月已被一篇高相关论文用作「来源证据束」，ProvPurge 的回执叙事必须明确改称/界定为 probe-scorecard receipt 并引它作近邻；(ii)「逐句粒度才忠实」与 ProvPurge 句级来源图同向，审稿人会问与 PolyGraph 的差别，答案必须落在「对象不同：对话记忆的删除事务 vs KG 构造清理；且我们有在线探针门控，他们无行为验证」。
- **FEB-30 / Beyond Downstream QA**（arXiv:2610.22209，2026-10-15）：内在 vs 外在评测平均分歧 16 点、操作分类仅 20% 正确。任务书已列占领。URL: https://arxiv.org/abs/2610.22209v1
- **MemCut**（arXiv:2610.21405，2026-10-15，已开全文）：456 个人工秘密、平均每事实 15.7 个 minimal sufficient sets 全标注；Native/Residual 双恢复与 hidden-risk share；LLM-guided 清理后残余恢复仍 39–76%；§6.1 经验律原文（见 §3c）。URL: https://arxiv.org/html/2610.21405v1
- **AfterDeleteBench**（arXiv:2610.31366，2026-10-22，已开摘要页与 HTML）：exposed-then-deleted vs **never-exposed 配对世界**把恢复归因于那次暴露；完整秘密串不存在、仅分量分散存储时仍能精确重构 62–99%（Qwen/Mistral/Phi-3）；把 reconstruction、write-back、fresh-session recall 分开测（R/C/A）；provenance-aware closure 在其测试设置下清零恢复，exact-literal write filter 只能挡回写、挡不了已有残余；作者自承「不主张广义 agent 遗忘问题为新」。URL: https://arxiv.org/abs/2610.31366
  - 含义：这是 10 月下旬最晚、离 ProvPurge 最近的一篇。它占领了「配对对照归因」与「回写/新会话复活」作为**基准贡献**的写法；ProvPurge 若要用这两样，只能用作事务内机制/耐久切片并明确引它为并发工作（concurrent work），抢「首个」必被击穿。

#### 1.2 上半年锚点（划定候选边界用，【已证实】）

- **LETHE / ForgetEval**（论文 PDF 随仓库发布，2026-05-13）：§5.4 回执 = 被删 row ids + **已删文本的 SHA-256** + purge 事件 id + 事件日志 Merkle root，Ed25519 签名，第三方可验 (a) 删了哪些行 (b) 日志未被篡改 (c) 签发者合法；§11 Roadmap 原文规划 v0.2–v0.6 五个扩展，其中 **v0.6 = "a receipt-verification family that scores whether a system produces auditable proof of deletion"**（详见 §3a 原文引证）。URL: https://github.com/deeplethe/lethe/raw/refs/heads/main/paper/paper.pdf
- **Control-Plane Placement**（arXiv:2606.15903）：ForgetEval-Adv 385 例，mutation-time LLM hook 带来 +22.6～24.1pt——「LLM 放在删除变异点」已被系统研究。URL: https://arxiv.org/html/2606.15903
- **Ghost Vectors**（arXiv:2606.18497，2026-06-16）：soft-delete 的 HNSW 向量可用 Vec2Text 反演（人名精确恢复 25.5% 等），对策 Epoch Key Rotation + ECDSA 删除证明。URL: https://arxiv.org/abs/2606.18497 ——物理/索引层「删除证明」已占。
- **SBU / Agentic Unlearning**（arXiv:2602.17692，2026-02）：参数 + 持久记忆双通道同步遗忘（dependency closure + 参数随机参考对齐）。URL: https://arxiv.org/abs/2602.17692v1 ——双通道已占且需训练，超预算。
- **REBEL**（arXiv:2602.06248，2026-02）：进化式对抗提示恢复「已遗忘」知识，TOFU ASR 达 60%、WMDP 达 93%，并证明标准遗忘分数与 relearning 动态都是对抗可恢复性的弱预测。URL: https://arxiv.org/pdf/2602.06248 ——「静态探针低估泄漏、自适应攻击更强」在参数遗忘上已证；但它测的是**别人训练好的遗忘方法**，没有「删除器在环内对着探针优化」这一层。
- **Auditing Forgetting in LMLMs**（arXiv:2607.00605，2026-07-01，已开 PDF）：FULL/DEL-ON/DEL-OFF 三干预因果分解参数泄漏 vs 检索介导正确性，残余 0.7%～13.6% 主要来自 near-neighbor 检索重构。URL: https://arxiv.org/abs/2607.00605v1 ——「泄漏通道分解」作为审计方法已占，ProvPurge 不要再把通道分解包装成创新。
- **Deployment-Time Memorization**（arXiv:2606.10062，2026-06）：Forgetting Residue Score；raw-only 删除后派生摘要层约 20% 可恢复，只有 full purge/tombstone 清零。URL: https://arxiv.org/pdf/2606.10062
- **MemLeak**（arXiv:2606.29788）：多模态通道——直接探针 <1%，保留相关文本 18.3%、保留图像 12.0% 可恢复。URL: https://arxiv.org/pdf/2606.29788
- **GateMem**（arXiv:2606.18829，2026-06-17）：多主体共享记忆治理（效用 + 访问控制 + 主动遗忘联合评测）。URL: https://arxiv.org/abs/2606.18829v1
- **长期记忆安全综述**（arXiv:2604.16548）：Verifiable Memory Governance 五原语含 Verified Forgetting；其生命周期表明确指出**保留的审计日志本身可能成为提取目标**【index 级，检索片段】。URL: https://arxiv.org/pdf/2604.16548

#### 1.3 扫描的负结果（同样重要）

在上述范围内，本次**未发现**以下已发表工作（表述限于本次扫描范围，不等于全网不存在）：
1. 以「删除器在闭环中反复针对同一探针集扩删」为威胁模型、测**门控探针 vs 留出探针泄漏差**的工作；
2. 针对**行为探针成绩单型回执**的伪造/假接受评测（LETHE 的 v0.6 仍是路线图）；
3. 以 sufficient-set **覆盖完备度**（而非删除条数）为自变量的残余恢复律；
4. 对回执 artifact 本身做目标重构测量的 **ReceiptLeak** 指标（只有综述的风险提示与一个开源包的内容无关回执实践，见 C6）。

---

### 2. 候选创新点（逐个五件套）

#### C1. 留出探针泛化缺口：预承诺门控集 + 密封留出集的双集事务【第二贡献首选】

- **一句话定义**：把探针集切成删除闭环可见的 gate 集（删除前哈希预承诺、冻结）与删除器永不可见的 holdout 集（提交后才由不同变换族/不同生成种子生成并开封），回执同时签两套成绩，并把 `Δ = Leak_holdout − Leak_gate` 定义为该次删除事务的**探针过拟合缺口**，Δ 超阈值则回执降级为「仅对门控探针成立」。
- **与最接近工作的区别**：
  - REBEL（2602.06248）证明静态 benign 探针低估参数遗忘的可恢复性、用进化攻击测**已完成的**遗忘——它没有闭环：被测系统从未对着 REBEL 的探针迭代优化过。ProvPurge 的删除器恰恰在环内对 gate 集迭代扩删，这是教科书式的 adaptive overfitting 通道【分析推断：该通道由 ProvPurge 结构产生，非 REBEL 结论】，REBEL 的框架测不到、也管不了它。
  - BEMULA 的 attack 套件是 5 个公开固定帧且明言从不改写——在 BEMULA 上调系统必然部分是对这 5 帧的拟合，但基准本身不提供留出帧，C1 等于给 BEMULA 补上它缺的泛化检验（注意措辞：是「评测协议补充」，不是新基准，避开自建基准红线）。
  - MemCut 的恢复问题是单一固定四选一题、reader 固定；AfterDeleteBench 的 auditor 用固定算子词表。均无 gate/holdout 之分。
- **基准验证**：BEMULA 为主——gate 用 attack 5 帧中的 3 帧 + efficacy 直问，holdout 用其余 2 帧 + 对帧的改写变体（改写只用于 holdout，不动基准原帧，保证可比性声明干净）；报告每个系统/每次事务的 Δ 与 RPR 的关系。MemCut 辅证：gate 用其四选一原题，holdout 用需要跨证据组合的变体问法，测 Res 口径下 Δ。LongMemEval-S 知识更新子集可作「探针不过是因为真忘了还是没存住」的 retention 对照。
  - **防「难度混杂」的关键设计（必须有）**：holdout 与 gate 在**未删除**的 store 上先做阳性对照校准，两套的删前泄漏率须大致相等（探针「有力」且等难），否则 Δ 只是难度差。这一条同时把 C4 的校准思想吃进 C1。
- **复杂度与 5×3090 可行性**：低。无新模型、无训练；成本 = 每事务多跑一遍 holdout 探针的本地推理（7B–32B-AWQ 级 reader 在 3090 上是 BEMULA 官方已用的量级，其 Qwen3-32B-AWQ 配置即为一例）。生成 holdout 变体用本地模型 + 规则变换（实体替换、语序/时态变换、跨跳组合、否定前提）即可，不必调外部 API。
- **审稿人最可能的攻击点**：(1)「holdout 只是更难，Δ 不是过拟合」——靠删前等难校准 + 同一变换族内随机切分 gate/holdout（族内 Δ）回击，论文里必须预置这个实验；(2)「这不就是 train/test split 的常识」——回击点在事务语义：预承诺哈希 + 回执签 Δ + Δ 决定回执等级，使它成为删除**证明强度**的一部分而非普通评测卫生；(3) 若实测 Δ≈0，贡献塌掉——这是真实风险，应先在 BEMULA 子集上做 pilot 再决定是否升为第二贡献（Δ≈0 本身也可写成「probe-gated 事务未出现可测过拟合」的负结果，但分量明显变弱）。

#### C2. 回执可验证性评测：针对探针成绩单回执的假接受分类学（LETHE v0.6 落地）【第二贡献并列首选】

- **一句话定义**：定义一个独立验证者协议——验证者只拿到（回执、删后 store 快照、有限探针预算），输出 accept/reject；系统性构造**签名合法但语义为假**的回执并测验证者的假接受率（false-accept rate），把回执验证拆成三个正交性质：**完整性 integrity**（成绩单未被事后改）、**完备性 completeness**（成绩单覆盖的探针/对象范围外无可推导残余）、**新鲜性 freshness**（回执绑定当前 store 版本，不是旧事务的重放）。
- **与最接近工作的区别**（这一条必须写得非常硬，因为近邻多）：
  - LETHE §5.4 的回执只证删行清单 + 日志完整性 + 签发者，**不证行为上不可恢复**；其 §11 路线图把 receipt-verification family 列为 v0.6 规划（原文见 §3a）。C2 不是再发明一个回执，而是**实现这个被自承未做的评测族**，且验证对象换成 ProvPurge 的探针成绩单——验证者要重跑/抽查探针，而非只验签。
  - TSD-AUDIT（2609.19456）的证书验的是 HNSW **遍历过程**（打分节点集合 vs 删除快照），威胁模型是 honest-but-buggy；C2 验的是**行为可恢复性与成绩单范围**，且包含主动伪造回执的攻击者（挑探针、缩范围、重放旧回执）——过程合规 ≠ 行为遗忘，两者互补。
  - PolyGraph（2610.17719）的 re-derivation audit 验的是**构造来源 receipt 的忠实度**（所引证据是否真充分），对象是 KG artifact 的来源束；C2 验的是**删除回执的完备性**（没写进回执的残余是否存在），方向相反：一个验「引的对不对」，一个验「漏的有没有」。
  - Ghost Vectors 的 ECDSA proof、VeriForgot 式区块链证书（【index，仅预印本检索片段，未开全文核验，不作强引用】）都停在「事件发生过」的证明层。
- **攻击分类（建议至少做满 5 类）**：A1 诚实不完备——真删了回执所列对象、签名全合法，但可推导证据残留（用 MemCut sufficient set 构造）；A2 挑探针——回执只签容易探针子集，难探针（多跳/聚合）被排除在成绩单外；A3 范围缩水——回执对象清单只含 disclosure、悄悄不含派生对象；A4 重放/过期——删除后又写入新证据（或合法 relearn 了相似事实），旧回执被拿来证明当前 store；A5 成绩单篡改——改一格探针结果（这一类应被哈希/签名平凡挡住，**只作健全性 sanity 对照，不作卖点**，否则会被审稿人嘲笑在打稻草人）；A6 回执自身泄漏——见 C6，并入此处最自然。
  - 核心论点【分析推断】：在一个**持有签名密钥的运营者可以说谎**的模型下，密码学只能给 integrity；completeness 只能靠验证者独立重探（且受探针预算约束，假接受率是预算的函数——这正好与 C5 的预算曲线接上），freshness 只能靠把回执绑到 store 版本/日志头。这三分解是 C2 的概念贡献，比攻击清单本身更重要。
- **基准验证**：在 BEMULA 135 个请求上由 ProvPurge 与两个弱删除基线（disclosure-only ≈ MemCut S0、检索清理 ≈ S2）各发回执，验证者在不同探针预算（如 5/20/50 次）下的假接受率曲线为主结果；MemCut 子集提供 A1 的金标准（sufficient set 未打断 = 客观为假回执）。不需要新基准，全部寄生在既有基准的删后状态上。
- **复杂度与 5×3090 可行性**：中低。签名/哈希/版本绑定是确定性代码（LETHE 自报回执签发验证亚毫秒级，可作量级参照）；主要成本是验证者重探的推理调用，与 C1 的 holdout 设施可共用同一套探针运行器。完全在本地推理预算内。
- **审稿人最可能的攻击点**：(1)「LETHE 路线图已经预告了，你这是抢跑他们的 v0.6」——回应：路线图是公开议程、且其回执对象与 ProvPurge 不同；在相关工作里主动引 LETHE 路线图并声明「我们实现并扩展之」比假装没想到更安全；(2)「运营者恶意时整个回执体系无意义」——必须预先把威胁模型分层写清（honest-but-incomplete 是主战场，恶意运营者下回执退化为承诺 + 独立重探），含糊其辞必被打；(3)「假接受率依赖验证者探针质量，测的是验证者不是回执」——这正是要报告预算-假接受曲线而非单点的原因。

#### C3. 可推导覆盖律：ProvPurge 版的成本-效用经验律（换自变量的 MemCut 律）【分析章/强化贡献】

- **一句话定义**：在 MemCut 的 sufficient-set 标注上，以**未打断 sufficient set 的可利用比例 U**（而非删除条数 k）为自变量拟合残余恢复 `Res ≈ f(U, 冗余度)`，并以闭包扩张轮次/闭包超出 minimal cut 的倍率为成本自变量拟合兄弟损伤，检验一个可证伪命题：**当且仅当 U→0（cut 完成）时 Res 出现阶跃式塌陷，而不是随 k 线性下降**；同时给出回执的统计读法——「n 条覆盖不同 sufficient set 的探针全不过」对应的泄漏率置信上界（按探针间相关性折算有效样本量 n_eff，而非把 n 条同源探针当独立样本）。
- **与最接近工作的区别**：MemCut §6.1 原文（已核）：在 132 个 sweep 设置上 `Res ≈ 13×(8.2−k)，R²=0.95`，system/model/strategy 合计移动 ≤4 点，外推 k=8.2（≈平均事实全部证据数）时 Res→0，并附带一句「complete cut 达成时单事实所需删除远少于此」——即 MemCut 的律是**为删不全的 baseline 拟合的计数律**，cut 完备只在文末被定性提及、没有为「能完成 cut 的删除器」拟合过。C3 的经验内容有三块 MemCut 没有的：(i) 以 U 为自变量的律在 ProvPurge 这类主动求 cut 的删除器上成立且优于 k 律（比较 R²/预测误差）；(ii) 律的**跨基准迁移**：在无标注的 BEMULA 上用 ProvPurge 自建来源图估计 sufficient set，律仍能预测 RPR/FE；(iii) 成本侧的律（扩张轮次 → 兄弟损伤），MemCut 只有「非证据选择占比」这一源头级附带量，没有把损伤写成扩张过程的函数。
  - **必须自曝的陷阱**【分析推断】：在 MemCut 的定义内，「打断全部 sufficient set ⇒ 按标注不可推导」是定义性恒真，本身没有经验含量；C3 的经验含量只在三处——reader 的实际行为是否服从标注推导结构（reader 可能用标注外路径恢复，也可能打断了还答对靠猜）、U 的**中间区间**形状、以及向 BEMULA 的迁移。论文里若不主动点破这一点，审稿人会点破，且更难看。
- **基准验证**：MemCut 为律的拟合主场（456 事实 + 复刻其 144 事实 subset sweep 的设计，换上 ProvPurge 删除器）；BEMULA 为迁移验证（预测 FE/RPR 与实测对齐度）；LongMemEval-S/STALE 不适合此律（无 sufficient-set 结构），不要硬套。
- **复杂度与 5×3090 可行性**：低-中。本质是主实验日志的再分析 + 一轮受控 sweep（不同扩张深度/不同候选池宽度下重复删除同一批事实）。推理量以「数百事实 × 几种配置」计，3090 集群数天内可完成；无训练。
- **审稿人最可能的攻击点**：(1)「曲线拟合的事后讲故事（post-hoc law）」——靠预注册拟合形式（先定 f 的函数族再看数据）+ 留出事实集上验预测误差回击；(2)「U 依赖 MemCut 一家的人工标注，律不可迁移」——BEMULA 迁移实验是生死线，做不出就把 C3 降级为 MemCut 辅证章的一节，不要硬撑为贡献；(3)「统计上界是教科书 Clopper–Pearson 换皮」——承认其标准性，卖点落在 n_eff 的相关性折算（同源探针共享 sufficient set 时有效样本远小于 n）与回执的实际误读纠正，不声称新定理。

#### C4. 可归因泄漏门控：对照校准的 Δ-commit 与探针效力前置检验【强化内核】

- **一句话定义**：把提交条件从「探针绝对全不过」改成三段式——(i) **效力检验**：每条探针在删前 store 上必须真泄漏（否则该探针是空探针，不得计入回执分母）；(ii) **可归因检验**：删后泄漏率减去匹配对照 store（从未暴露该目标 / 实体替换版）的泄漏率 `Δ ≤ τ` 才算「残余泄漏」，对照上同样能答出的部分记为「合法可推断/参数知识」并在回执中单列、不触发扩删；(iii) 兄弟保全仍用绝对全过。
- **与最接近工作的区别**：BEMULA 已在**评分**上做配对对照（effective correctness = 删后结果 ∧ 对照探针，原文 Eq.2–3，已核）；AfterDeleteBench（10-22）已在**审计**上做 never-exposed 配对世界归因。两者都把对照放在评测端。C4 的剩余空间只在把对照搬进**事务的停机条件**：它决定删除循环何时停、以及不停的原因分类（真残余 / 合法可推断 / 空探针），直接治 ProvPurge 闭环的一个真实病——目标若可由合法保留的兄弟事实或模型常识推出，绝对门控将永远不过、闭包无限扩张、兄弟被误伤。范围必须收窄到「停机规则 + 回执中的不提交原因分类学」，声称「首个配对对照」必撞 AfterDeleteBench。
- **基准验证**：BEMULA 天然有 control 版对话，对照 store 免费；指标看 RPR 不降的前提下，平均删除量/扩张轮次下降多少、无限扩张型失败减少多少。MemCut 是反例边界：其原文控制实验显示无记忆时恢复 0%、单条证据 0%——即 MemCut 的事实被设计成不可猜，Δ 门控在 MemCut 上应几乎不触发，正好用作「门控不滥用」的阴性验证。LongMemEval-S 知识更新子集里多为现实事实，参数知识污染最重，是 Δ 门控的主战场。
- **复杂度与 5×3090 可行性**：中。对照 store 需多维护一份状态、多跑一遍探针（探针推理量翻倍，但仍是小模型本地推理）；BEMULA 已提供配对对话，省去构造对照的成本。
- **审稿人最可能的攻击点**：(1)「τ 是松动删除标准的后门，隐私场景下『可推断』就是泄漏」——这是最强攻击，必须正面回应：GDPR 式删除的对象是「该次暴露的贡献」还是「一切可推得性」是规范选择，回执单列 Δ 分解恰恰让这个选择可审计，而不是偷偷放宽；且对敏感类请求可设 τ=0 退化回绝对门控。(2)「与 AfterDeleteBench 撞车」——靠评测端 vs 事务端的定位差异 + 主动引用化解。

#### C5. 探针强度证书：攻击预算曲线与换读者迁移【锦上添花·鲁棒性切片】

- **一句话定义**：回执成绩单不报二值 pass，而报两条曲线——泄漏率随攻击者预算（探针数/改写轮数/进化迭代数）的 `Leak@B` 曲线，以及把 gate reader 换成不同家族/更强的本地 reader 后的迁移泄漏差，提交阈值明确标注「在预算 B*、reader 类 R* 下成立」。
- **与最接近工作的区别与反证（必须诚实）**：REBEL 已证参数遗忘下自适应攻击 ASR 远高于静态探针；模型可迁移性研究（2609.05339）已证换 reader 可致 ±10 点以上摆动。但 **MemCut 原文报告其残余恢复在 4 个 write model 间差异至多约 5 点**——这是现成的反证风险：在 MemCut 的「把存活证据直接喂给 reader」口径下读者强度近饱和，迁移差可能测出来很小。因此 C5 只宜作低成本鲁棒性切片：若差小，它给主结论加一道保险；若差大，它救主结论。不要预设它一定是贡献。
- **基准/成本/攻击点**：BEMULA 子集 + 第二 reader 家族（如 gate 用 14B、audit 用 32B-AWQ，均为 3090 可跑的量化配置）重跑回执即可，成本低。审稿攻击：「预算曲线是 REBEL 结论的平移」——定位为回执的**强度标注规范**而非新发现即可避战。

#### C6. 回执自身泄漏审计（ReceiptLeak）与内容无关回执【锦上添花·宜并入 C2 的 A6】

- **一句话定义**：把回执本身当作删除后存活 artifact 来攻击——攻击者只拿到回执（探针问题文本、被删对象 id、**已删文本的哈希**、目标实体名）尝试重构被删事实，定义 ReceiptLeak 率；缓解版回执用加盐承诺/随机 surrogate id/按族聚合分数替代明文探针与裸哈希，并测缓解前后 ReceiptLeak 与可验证性的权衡。
- **撞车核查**：LETHE 回执含已删文本 SHA-256（§5.4 已核）——低熵事实（职位、城市、日期）的裸哈希可被字典验证【分析推断，本次未找到有人实际演示该攻击】；长期记忆安全综述已定性指出审计日志可成为提取目标【index】；一个开源包 agora-mnemo 的 PyPI 页面已明言「PII 的 hash 仍是 PII」并只用随机 surrogate id 发内容无关回执（【已打开检索片段，非论文级来源】URL: http://pypi.org/project/agora-mnemo/1.12.2/）。即：**风险意识与缓解做法已存在于工程侧，但本次扫描未发现任何论文对删除回执做过系统的目标重构测量**——所以可做的是「测量 + 在回执格式上的权衡实验」，不可声称「首个发现回执会泄漏」。
- **基准/成本/攻击点**：BEMULA 的目标值空间（同 profile 的 distractor 值）天然构成字典，攻击实验几乎零额外推理成本。审稿攻击：「探针问题里本来就含实体名，这不算泄漏」——需区分回执的**预期披露**（签发给谁、谁有权看）与**超预期重构**（仅凭回执恢复出本应被删的值本身），威胁模型写不清就会被打成伪问题。单独分量不足，建议作为 C2 的第六类攻击与一个小节。

#### C7. 删除后生命周期耐久再探：提交不是终点【锦上添花·评测切片】

- **一句话定义**：事务提交后不立即结案，在其后插入 T 步正常生命周期操作（若干无关新会话写入、一次离线 consolidation/摘要再生、一次相似事实的合法 relearn）后重跑同一探针集，报告「提交存活率」随操作步数的衰减曲线与复活来源归因（consolidation 再生 / relearn 混淆 / 新证据再推导）。
- **撞车核查**：AfterDeleteBench 已分离 reconstruction/write-back/fresh-session recall 三段（一步版耐久）；BEMULA 的 relearning 套件已测「删后学相似事实」（但那是评测套件的一环，不是已提交事务的纵向耐久）；一篇工程博客（StateCore，【index，非学术来源】）已描述摘要 digest 三种复活路径并给出墓碑式对策——机制侧已被工程界摸过。因此 C7 只能定位为**多周期纵向评测切片**，新意在「衰减曲线 + 复活归因」而非现象本身。
- **基准/成本/攻击点**：寄生 BEMULA（relearning 套件 + 多会话结构现成）与 LightMem 类离线 consolidation 系统；成本为重复探针推理，中等。审稿攻击：「这就是 AfterDeleteBench 的 A 指标拉长」——若衰减曲线平坦（不复活），此节自动降为附录，不要提前写进 contribution 列表。

#### （附）C8. 主动探针调度——只建议写进 future work，不建议做

按 sufficient-set 覆盖与不确定性主动选下一条探针、以最小探针预算达到给定置信停机：想法干净，但 REBEL 已占领「自适应生成探针」的叙事高地，调度增益需要大量实验才能证出与随机/全跑的差距，在硕士论文周期内性价比低。本次扫描未发现删除审计侧的主动调度工作【负结果，限扫描范围】，故列为可提的 future work，不要排进实验计划。

---

### 3. 三类特别机会深挖

#### 3a. LETHE receipt-verification：原文坐实「规划了、没人做」

【已证实】LETHE/ForgetEval 论文 §11 Discussion 的 Roadmap 段原文（打开仓库论文 PDF 核读，2026-10-30）：

> "Roadmap. Beyond v0.1 … we plan five extensions: (v0.2) a human-curated adversarial layer …; (v0.3) multilingual scale-up …; (v0.4) crosslingual paraphrase …; (v0.5) temporal probes …; **(v0.6) a receipt-verification family that scores whether a system produces auditable proof of deletion.** The Adapter Protocol will gain at most one optional method per extension."

同论文 §5.4 定义的回执只含删行 id、已删文本 SHA-256、purge 事件 id 与日志 Merkle root，第三方可验的是删行清单、日志未篡改、签发者合法——**不含任何行为可恢复性证据**。

判断：
- 「LETHE 路线图自承没人做 receipt-verification 评测」这一前提**成立**，且截至 2026-10-30 的本次扫描，未发现该 v0.6 已被 LETHE 团队或他人发表（其后续论文 Control-Plane Placement 做的是 ForgetEval-Adv 与控制面位置，不是回执验证族）。
- 但空位比字面更窄：TSD-AUDIT 已做了「过程证书的独立验证」，PolyGraph 已做了「来源回执的忠实度审计」。真正无人区是 C2 定义的**行为成绩单回执的完备性/新鲜性假接受评测**。ProvPurge 的回执恰好是成绩单型（签探针成绩），与 LETHE 的清单型回执对象不同——这是能站住的差异化，也是必须在论文里第一时间讲清的差异化。
- 风险提示：这是公开路线图上的空位，LETHE 作者本人或其他组随时可能抢跑；越晚动笔风险越高。若选 C2，建议优先做出 A1/A2/A4 三类攻击 + 预算-假接受曲线这一最小可发表核。

#### 3b. 探针集设计空间：哪些子空间还空着

把探针设计拆成五维，逐维核查占领情况：

| 维度 | 已占领（已证实） | 未占领/可做 |
|---|---|---|
| 生成方式 | 模板生成（ForgetEval、BEMULA 固定帧）、LLM 生成（BEMULA 题面）、进化式生成（REBEL，参数遗忘）、固定算子 auditor（AfterDeleteBench） | 面向 sufficient set 的**覆盖导向生成**：按未打断的推导组合定向生成探针（与 C3 的 U 联动）；生成器与删除器同源时的自证污染控制 |
| 防过拟合 | 通用 ML 的 holdout 常识；**删除领域内无人处理**（本次扫描负结果） | C1 的预承诺 + 密封 holdout + Δ 入回执——这是 ProvPurge 闭环独有的问题意识 |
| 强度标定 | REBEL 的 ASR（参数层）；MemCut 的删前/对照控制（恢复须 96–99% 且无记忆 0%）；执行状态遗忘的 Leak@probes | **预算标定**（Leak@B 曲线）与**等难校准**（gate/holdout 删前等泄漏）作为回执的强度标注——C5/C1 的组件 |
| 效力/阴性对照 | BEMULA 配对对照评分、MemCut 三项 0% 对照、AfterDeleteBench never-exposed 世界 | 把效力检验搬进事务停机条件——C4 |
| 通道覆盖 | 多跳/聚合/时序（MemCut 四类推导）、跨模态（MemLeak）、遍历过程（TSD-AUDIT）、分词（Tokens Remember，参数层）、KV cache（执行状态遗忘） | 文本记忆层内通道已相当满；再加新通道（如纯分词通道）边际价值低，不要为通道而通道 |

结论：探针方向最有价值的不是「再 invent 一种探针」，而是**探针集的治理**（预承诺、防过拟合、等难、强度标注）——这恰好与回执（成绩单的治理）同构，C1+C2 可以合并成一条「可证伪的删除证明」主线，互相供血。

#### 3c. 删除成本-效用律：MemCut 律的原文与 ProvPurge 版的活路

【已证实】MemCut §6.1 原文（打开 HTML 全文核读）：

> "Across the 132 settings of the two sweeps, residual recovery tracks the number k of evidence turns removed almost exactly, **Res ≈ 13(8.2 − k) (R² = 0.95)**, while system, model, and strategy together shift it by at most four points. The fitted line extrapolates to zero near k = 8.2, nearly all the evidence of an average fact. … Individual facts may need far fewer deletions when a complete cut is reached and selected."

解读与推论【分析推断】：
- 这条律的惊人之处在于**与系统/模型/策略近无关**——它本质是「baseline 删除器都只能按检索相似度捞证据，捞到几条决定残余」的覆盖律。ProvPurge 若也只拟合 Res~k，等于在别人的律上复现，必死。
- 活路在换轴：ProvPurge 的删除量不是 k 的随机子集，而是瞄准 minimal cut（MemCut 原文：minimal cut 平均仅 3.0 条、最大 5 条，而证据平均约 8 条）。若 cut 导向删除真有效，**同样 k 下 Res 应显著低于 MemCut 律的预测线**——「偏离 MemCut 律的幅度」本身就是 ProvPurge 主贡献的一个漂亮量化表述（律外点/律的失效边界），而 C3 的 U 律是它的正面形式。这比另起一条无关的律更聪明：借已发表的律当零假设。
- 可证明界的部分要克制：能严格写的只有 (i) 标注模型内 cut 完备 ⇒ 不可推导（定义性，非成果）；(ii) 探针全不过 ⇒ 泄漏率的统计上界（含相关性折算，标准工具）。把这两条包裝成「可证明删除保证」会招来最严厉的审稿火力（行为探针分布外、reader 外的一切都不在界内）。定位为「经验律 + 诚实的统计读法」是安全区。
- 未获取：MemCut 是否在附录给出了按推导类型（fact assembly / relation chain / aggregation / temporal）分开的律——本次只核读到正文 §6.1 与 §6.2 开头，**分类型拟合未获取**；若其附录已有分类型律，C3 应进一步收窄到「U 律 + 迁移」两块，动笔前需补核 MemCut 附录 B.3/Table 9 一带。

---

### 4. 排序与取舍建议

#### 4.1 值得作为第二贡献（选一主打，可组合）

1. **C1 留出探针泛化缺口**——与内核的耦合最紧（没有 probe-gated 闭环就没有这个问题）、撞车面最小、实验最便宜。建议与 C2 合并叙事为「删除证明的可证伪性」：C1 管成绩单**是否被刷**，C2 管成绩单**是否可被独立核验与伪造**。合并后是一个完整的第二贡献，单独任何一个都略单薄。
2. **C2 回执可验证性评测**——议程合法性最强（LETHE 路线图背书），但近邻最密、且有被抢跑的时间风险。若论文定位偏治理/审计选它作主打；若定位偏系统/方法，选 C1 主打、C2 作其验证章。

#### 4.2 值得做但只作强化（写进方法/分析，不进 contribution 列表）

- **C3 覆盖律**：作为分析章 + 主结果的一个图（偏离 MemCut 律 / U 律拟合 / BEMULA 迁移）。迁移成功可升格，失败不伤主线。
- **C4 Δ-commit 门控**：作为内核的停机规则修订 + 消融（绝对门控 vs Δ 门控的删除量与 RPR 对比）。它实际提升系统，但创新叙事窄，适合「方法完善」而非「贡献点」。

#### 4.3 锦上添花（有余力再做，做不出不影响毕业）

- **C5 强度证书**（一个子集、一个额外 reader、一组预算点即可）、**C7 耐久再探**（寄生 BEMULA relearning 流程）、**C6 ReceiptLeak 小节**（并入 C2 时顺手做掉，字典攻击几乎免费）。

#### 4.4 坑（看着诱人，不要做）

| 坑 | 为什么是坑（撞车/成本/可证伪性） |
|---|---|
| 执行状态遗忘 / KV-cache 作废 / 选择性重放 | 2609.04875 已形式化 + 给下界 + 实现跨层重放；且需要改造 serving 栈，5×3090 本地推理下工程量失控 |
| 参数通道遗忘（与 SBU 式双通道） | 2602.17692 已占；需微调/训练，违反无训练预算 |
| 遍历安全向量删除 / 物理层删除证明 | TSD-AUDIT（2609.19456）+ Ghost Vectors（2606.18497）双占；是索引引擎工程，不产生 BEMULA 收益 |
| 多模态残余通道 | MemLeak（2606.29788）已系统做过；引入 VLM/图像数据直接换赛道 |
| 多主体/政策治理删除 | GateMem（2606.18829）+ Retrieval Admissibility（2610.07309）+ TRACE 族已占满，且主基准不支持 |
| 摘要逐句再派生/支持计数对账当卖点 | PolyGraph 逐句 receipt（2610.17719）+ Stashbird 支持对账（2609.34242）同季双占；可作实现细节借鉴，不可作创新宣称 |
| 分词/编码变体探针当主通道 | Tokens Remember（2609.29045）已占参数层叙事；文本记忆层内与 identifier obfuscation 高度重叠，只配作 C1 的一个留出变换族 |
| 再自建一个删除基准 | 2026-10 单月已有 BEMULA/MemCut/AfterDeleteBench 三个，审稿人基准疲劳；且任务书已明令排除 |
| 声称 (ε,δ) 式 certified deletion / 「可证明遗忘」 | 行为探针只能给出相对于探针分布与 reader 的统计陈述，声称证明级保证不可证伪且必被以 REBEL（自适应攻击恢复 93%）反例攻击；只可写统计上界（C3） |
| 在 STALE 上做 draft 审计修复 | StateAuditor（2608.01619）已占 state→draft 修复；STALE 本身是更新基准而非用户请求删除基准，错位使用会被质疑评测有效性 |

---

## Could not verify

- **MemCut 附录的分类型/分系统拟合细节未获取**：只核读正文至 §6.2 开头；其附录 B.3、Table 9（whole-pool deletion 等）仅经正文转述得知，未逐行核读。若 C3 动笔，需先补核附录是否已有按推导类型的律。
- **MemForm（2026-10-09）未开全文**：仅 arXiv 检索页片段确认其为记忆形成投毒基准，与删除方向正交的判断为 index 级。
- **「agent 删除自身 traces」工作（2026-09-24）未开 arXiv 全文**：仅二手报道（dev.to / theagenttimes），本报告未把它用作引用级证据，仅作 C2 背景。
- **VeriForgot 未核验**：仅预印本平台检索片段（MIA oracle + 区块链证书，CIFAR-10/ResNet-18 实验），未开全文，质量与结论均未证实，报告中仅提及存在、不引其数字。
- **TRACE（2609.33517）的确切提交日期未获取**：依据 arXiv 编号判为 2026 年 9 月；其与任务书所述 TRACE 是否同一篇未完全确认——本报告按「admission 族已被占领」处理，两种情形下结论相同。
- **负结果的范围限定**：§1.3 与 C1/C2/C6 中「未发现已做者」的判断，仅限本次以英文关键词在 arXiv/仓库范围内、于 2026-10-30 的检索与开页核查；未系统检索中文数据库、OpenReview 在审稿件与工业博客（如 StateCore、ZeroRecall 等商业/工程来源仅作背景提及）。抢跑风险无法排除。
- **LETHE v0.6 是否已有未公开实现**：其仓库本次只核读了论文 PDF 的路线图与 §5.4；仓库其余文档/代码是否已含 receipt-verification 实现未逐一核查，动笔 C2 前应再查一次该仓库的 bench 目录与更新日志。

## Sources

（读取日期均为 2026-10-30；【全文/摘要页】= 已打开核读，【index】= 仅检索片段）

1. BEMULA — Can Agents Forget When Asked? arXiv:2610.18052, 2026-10-13【全文 HTML】https://arxiv.org/html/2610.18052v1
2. MemCut — Every Secret Leaves a Trail. arXiv:2610.21405, 2026-10-15【全文 HTML，§6.1 经验律原文】https://arxiv.org/html/2610.21405v1
3. PolyGraph — Keep Your Receipts: Trustworthy Cleanup of Agent-Built Knowledge Graphs. arXiv:2610.17719, 2026-10-13【摘要/HTML】https://arxiv.org/abs/2610.17719v1
4. FEB-30 — Beyond Downstream QA. arXiv:2610.22209, 2026-10-15【摘要页】https://arxiv.org/abs/2610.22209v1
5. AfterDeleteBench. arXiv:2610.31366, 2026-10-22【摘要页+HTML】https://arxiv.org/abs/2610.31366 ；HTML https://arxiv.org/html/2610.31366
6. Execution-State Unlearning. arXiv:2609.04875, 2026-09-04【摘要页】https://arxiv.org/abs/2609.04875v1
7. TSD-AUDIT — Beyond Private Training. arXiv:2609.19456, 2026-09-16【摘要页】https://arxiv.org/abs/2609.19456
8. The Tokens Remember. arXiv:2609.29045, 2026-09-24【摘要页】https://arxiv.org/abs/2609.29045v1
9. Memory Portability — Does Your Agent's Memory Survive a Model Upgrade? arXiv:2609.05339, 2026-09-04【摘要页】https://arxiv.org/abs/2609.05339v1
10. PACMI. arXiv:2610.05732, 2026-10-05【摘要页】https://arxiv.org/abs/2610.05732v1
11. Retrieval Admissibility — The Right Memory in the Wrong Context. arXiv:2610.07309, 2026-10-05【摘要页】https://arxiv.org/abs/2610.07309
12. StateAuditor — When Memory Updates but Behavior Does Not. arXiv:2608.01619, 2026-08-02【摘要页】https://arxiv.org/abs/2608.01619
13. Stashbird. arXiv:2609.34242, 2026-09-28（v2 2026-10-17）【摘要页】https://arxiv.org/abs/2609.34242
14. TRACE — Governing Memory Validity in Evolving Multi-Agent Systems. arXiv:2609.33517【摘要页，index 级日期】https://arxiv.org/abs/2609.33517v1
15. LETHE / ForgetEval 论文 PDF（DeepLethe 仓库，2026-05-13；§5.4 回执、§11 Roadmap v0.6 原文）【全文 PDF】https://github.com/deeplethe/lethe/raw/refs/heads/main/paper/paper.pdf
16. Control-Plane Placement Shapes Forgetting. arXiv:2606.15903【HTML】https://arxiv.org/html/2606.15903
17. Ghost Vectors. arXiv:2606.18497, 2026-06-16【摘要页】https://arxiv.org/abs/2606.18497
18. Agentic Unlearning / SBU. arXiv:2602.17692, 2026-02【摘要页】https://arxiv.org/abs/2602.17692v1
19. REBEL. arXiv:2602.06248, 2026-02【PDF 片段】https://arxiv.org/pdf/2602.06248
20. Auditing Forgetting in Limited Memory Language Models. arXiv:2607.00605, 2026-07-01【全文 PDF】https://arxiv.org/abs/2607.00605v1
21. Deployment-Time Memorization in Foundation-Model Agents. arXiv:2606.10062, 2026-06【PDF 片段】https://arxiv.org/pdf/2606.10062
22. MemLeak. arXiv:2606.29788【PDF 片段】https://arxiv.org/pdf/2606.29788
23. GateMem. arXiv:2606.18829, 2026-06-17【摘要页】https://arxiv.org/abs/2606.18829v1
24. ChronoMem. arXiv:2607.27773（v2 2026-08-05）【摘要页】https://arxiv.org/abs/2607.27773v2
25. STALE. arXiv:2605.06527, 2026-05-07【摘要页】https://arxiv.org/abs/2605.06527
26. A Survey on Long-Term Memory Security in LLM Agents. arXiv:2604.16548【index/PDF 片段】https://arxiv.org/pdf/2604.16548
27. agora-mnemo PyPI 页面（内容无关回执实践，非论文）【index】http://pypi.org/project/agora-mnemo/1.12.2/
