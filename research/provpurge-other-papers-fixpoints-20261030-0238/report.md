# ProvPurge 补读：其他论文的可修改点

## Summary

本次补读 6 项（2026-10-30 实际打开页面；标注「已证实」=页面原文/数据，「分析推断」=笔者据此对 ProvPurge 的推论）。结论先行：**真正需要改规格条款的只有 4 处**——(1) 探针判定必须双通道（官方解析 + raw 回复 target-anywhere + 组件级部分泄漏），且解析合同版本进回执（AfterDeleteBench §5.4/附录 G）；(2) 候选生成对 relation chain 型秘密必须以实体跳扩展为主、相似度截断为辅，并把「每条链节的候选覆盖」写成可核查量（MemCut 附录 D/B.3）；(3) 回执中的「被删行缺席检查」必须写明缺席的层级——未返回 / 未被遍历计分 / 索引中不存在，三者不可混用（TSD-AUDIT）；(4) 删除后重建幸存摘要的 consolidation 顺序须确定性化并记入回执、兄弟保全探针须在重建后状态上跑（MemForm Path Poisoning）。其余多为改文档措辞/宣称边界：MemCut 的 `Res≈13(8.2−k)` 只能作 pooled 风险先验、**不能**改写成分类型权重（附录 B.3 没有分类型拟合）；VeriForgot 与 ProvPurge 验证对象不同（参数 vs 外部记忆），无冲突、只借「预承诺哈希」一个字段级做法，不借 MIA 单阈值门、不借区块链；MemForm 应进必引（形成端威胁）但不改方法定位；PolyGraph 仓库 **2026-10-30 复查仍为 404（verified live）**。

方案标尺（任务给定）：LLM Agent 长期对话记忆的选择性遗忘；内核 probe-gated 硬删事务（句级来源图闭包＋实体扩展候选、组件节点计入闭包、来源不完备时探针门拒绝提交、预承诺攻击探针全不过且兄弟保全全过才提交、回执签探针成绩单且验证含被删行缺席检查）；已并入 F4 源-结论双语义；主基准 BEMULA，MemCut 只作 residual 直喂辅证；本地 5×RTX 3090 推理，无训练。

---

## Findings

### 1. AfterDeleteBench（Saleh et al., arXiv:2610.31366, 2026-10-22）

**读到的层级：全文（含附录 A–J），index 全文 HTML。** 来源：https://arxiv.org/html/2610.31366

#### 与 ProvPurge 直接相关的机制或数据（已证实）

- **删除后可恢复性的测量设计**：配对世界——同一 profile 跑 exposed-then-deleted（forget）与 never-exposed（clean）两世界，共享背景记录、模型、配置、auditor、查询预算与评分规则，模型看不到世界标签、删除回执与 lineage（§3）。主终点是精确重构 R（官方接口下解析出的预测与目标字面相等，无任何规范化），报告 ΔRec = Rec_forget − Rec_clean。主设置（Qwen2.5-7B-Instruct、分层记忆、gate off）单条 DIRECT 查询 ΔRec = **0.680（IID, 95% 配对区间 [0.615, 0.745]）/ 0.640（OOD）**，clean 世界 **0/200**（§5.1、Table 1）。生命周期另行分离 R（重构披露）/ C（写回持久化）/ A（fresh session 再现）/ E=R∧C∧A，并明确 E 是描述性合取、不是写回的因果估计量（§3、附录 A/D.3）：sealed gate 可把 C 压到 0 而 A 仍为正，故 **E=0 不等于遗忘**。
- **组件分离存储＋公开组合规则的 62–99% 重构**：Module C 变换记忆套件，目标形为 `REGION-MNEMONIC-NUMBER`。条件须同时满足才有此数字：C1 中三个带标签组件**分别存于不同记录、全部保留**、且存在**公开确定性组合规则**；完整目标串在 **0 / 1,248** 个 model-visible 状态中出现（作者做了 transformed-state leakage proof）。结果 ΔR 跨 6 个 model/split 格为 **0.615–0.990**（Table 2：Qwen IID 0.817 / OOD 0.615；Mistral 0.990 / 0.856；Phi-3 0.923 / 0.923），clean 均为 0。两个边界同样是原文：C2（对 C1 施加 recorded-lineage closure）全部为 0；C3（丢掉 NUMBER 组件）完整恢复全为 0，**但部分泄漏不为 0**——region/mnemonic 在回复中的字面出现计数如 Mistral IID 104/104、Qwen IID 93/93（附录 G.3 Table 11）。注意对照：原始 compositional profiles（组件在但无此受控构造）在 8 个 auditor 下 forget/clean 均 0/50（§5.1、附录 D.2）——即「组合型 = 安全」的读法被作者自己的 C1 实验否定，差别在组件齐全＋规则公开这一受控条件。
- **答案格式/解析器错位误判探针结果的警告细节**：legacy 合同要求模型输出嵌套 `{"final_prediction": {"value": ...}}`，而冻结 scorer 只读**顶层** `value` 字段（§3 Response interface、附录 I 给出两份合同原文）。后果（§5.4、附录 E/G.1）：Mistral 与 Phi-3 在 legacy 下官方 R=C=A=E=0，但其 raw IID 审计回复中含目标者达 **77/104 与 78/104**；确认面板上 legacy 的 exact-target-anywhere 计数 Mistral 148/200（IID）、Phi-3 150/200，官方恢复仍为 0。只把 Phi-3 的 **activation 合同**从 aligned 换成 legacy（状态不变），官方 E 从 77/104 跌到 **0/104**，而 raw 回复仍含目标 78/104。更强的是方向反转：Qwen matched 状态下，aligned 时 write-allowed 使 activation +0.158，legacy 时官方 activation 反而 **−0.224 [−0.329, −0.118]**，而三个分支 raw 回复均为 76/76 含目标（附录 G.2 Table 10）。作者同时给出反例：对 Qwen，alignment 会**降低**测得恢复——所以「对齐必然抬高恢复」也不成立。原文结论句：「An official zero therefore cannot be read as "the model did not disclose the target" without further evidence.」（§3）
- **探针轮次结构**：全部重复评测中 3,081 / 3,088 个恢复事件发生在 query 1、7 个在 query 2、之后为 0（§5.5）；但作者明确限定：固定算子词汇、该 residue 通道、且原始 compositional profiles 零成功，「does not establish that adaptive auditing is unnecessary in general」。
- **它没覆盖的情形清单**（§6 + 附录 J，原文）：语义（非字面）删除——可释义事实、偏好、概念；**不完备或错误的 provenance 边**、概率/多对多推导、未追踪的语义依赖、摘要层之外的派生状态；学习/压缩/选择性摘要与语义记忆；独立生产记忆框架（模型不自主决定存储、整合、检索、删除、候选写回）；固定算子词汇之外的无限制后续生成与任意 transcript 合成；**没有 exposed-and-retained 参照**，故 ΔRec 不能校准「正常可检索信息被保留的比例」；饱和的 100 例 utility sanity check（全条件 100/100、仅 20 个 near-miss，附录 H）不刻画真实下游效用、纠缠信息上的过度删除与规模化误挡；runtime/storage 成本未测。其 closure 零恢复被作者自称为完全 provenance＋确定性抽取式摘要（按 identifier 排序逐字拼接）下的「idealized structural upper bound」。

#### 具体修改建议

1. **【探针构造＋事务与回执＋评测口径｜改规格条款】探针成绩单必须双通道签署。** ProvPurge 现行回执签的是探针成绩单；若成绩单只含「经官方解析器判定的通过/不过」，则与 AfterDeleteBench 的 legacy 零分同构——目标明明在 raw 回复里、解析器记为失败，探针门会误放行。规格应要求每条攻击探针在回执中同时记三项：(a) 解析判定；(b) exact-target-anywhere（目标字面串是否出现在 raw 回复任意位置，作者同名诊断）；(c) 解析合同的版本/哈希（合同本身是测量系统的一部分）。三项不一致时以 (b) 为准触发拒绝提交。（分析推断：这是把该文的诊断字段移植为提交门条件，原文只用作事后诊断。）
2. **【探针构造｜改规格条款】加组件级部分泄漏探针，禁止把「完整重构 = 0」写成「无泄漏」。** C3 证明完整恢复为 0 时组件仍可大面积出现。ProvPurge 已把组件节点计入闭包，这正对 C1 风险；但提交判定还应包含组件级探针：任一被删秘密的组件（或 F4 结论侧的等价片段）在探针回复中字面出现，即记为部分泄漏并在成绩单中单独成行——可以不必然否决提交（取决于威胁模型），但不得与「全不过」合并统计。
3. **【评测口径｜改实验设计】BEMULA 评测补两个参照臂。** (i) never-exposed 配对对照（clean 世界恢复应为 0，否则探针本身在猜测/背诵，Δ 无从归因）；(ii) 作者明确缺失的 exposed-and-retained 参照（校准「该模型/接口本来就能表达多少保留信息」）——AfterDeleteBench 自己没做，ProvPurge 若做了反而是相对它的评测增量，可写入宣称。成本最高的一条，可只在主表子集上做。
4. **【宣称边界｜改文档措辞】** 凡引 AfterDeleteBench 的 closure 零恢复或 ProvPurge 自身的闭包零恢复，措辞须限定为「在来源完备且已追踪的派生通道内消除已观察恢复」，不得写「语义擦除」。ProvPurge 的「来源不完备时探针门拒绝提交」与作者对 closure 上界的限定方向一致，此条只是把限定词写死。另外：不得用其原始 compositional profiles 的 0/50 作为「组合型秘密安全」的证据（见上 C1 对照）。

### 2. MemCut（arXiv:2610.21405）附录补读

**读到的层级：附录 A、B.1–B.4、C、D 全读，及正文 §6.1/§6.3/§7。** 来源：https://arxiv.org/html/2610.21405v1

#### 核对结论：Res≈13(8.2−k) 在不同秘密类型上是否同律？

**已证实**：该律出自 §6.1 与附录 B.3 小节「One quantity behind both sweeps」——对两个 sweep 的 **132 个设置**（双策略 S1/S2 × 三系统 Turn RAG/MemoryBank/LightMem 类 × 两 write model，深读文本为三系统）做**一条 pooled 拟合**：每多删 1 个标注证据 turn，residual recovery 平均降约 13 点，R²=0.95；给定 k 后，system 与 write model 合计移动至多约 4 点、strategy 不足 1 点；拟合零点 8.2 ≈ 平均每事实证据数 8.1（Table 5）。

**关键限定（已证实＋分析）**：附录 B.3 **没有给出分类型、也没有给出分系统的独立拟合系数**（斜率/截距）——我逐节核对了 B.3 全文，只有 pooled 拟合与分类型的描述性表格。因此「同律」只能在弱意义上成立：类型差异在数据中主要体现为 **k 的可达性不同**，而非同一 k 下斜率不同。分类型数字（附录 C Table 12，16 配置均值）是：S2 后 Res **relation chain 71.8%**、fact assembly 60.2%、temporal 58.4%、**aggregation 24.3%**；chain 的 hidden-risk share 87.1%。而每事实实际删到的证据 turns（B.3 Table 10，S2、三系统均值）chain 只有 **2.4**，aggregation **4.6**、assembly 3.8、temporal 3.9；分系统看 chain 在每个系统都最低（MemoryBank 1.7 / Nemori 2.4 / LightMem 2.9）。把 Res 对 k 作图时 chain 落在高 Res 端，是因为它删得少（reach 失败），不是因为它的斜率被单独测出不同。**另有作者明示的反例域**（§6.1 末）：「Individual facts may need far fewer deletions when a complete cut is reached and selected」——minimal cut 均值只有 3.04、max 5（Table 5）；一旦达成 complete cut，线性律失效（删 3 个即可归零，而非删满 8.2 个）。ProvPurge 走的是闭包/割路线而非「逐个删证据」路线，正落在该律的失效域。

#### relation chain 的具体数字与构成（附录 D，已证实）

案例秘密：「用户公司雇的 Arden Advisory 属于 CFO Ravi Shah 的妻子 Nina Shah」。构成 = **4 条 link，每条 2 个证据 turn**：hiring（E1 Q3 合同、E2 Ravi 雇 Arden 的记录）、ownership（E3 Arden 登记文件列 Nina 为 sole owner、E4 Nina 以 owner 身份的 engagement letter）、relationship（E5 Ravi 与妻 Nina 主办的 offsite 晚宴、E6 结婚纪念日晚餐订位）、identity（E2 同上、E7 员工名单列 Ravi Shah 为 CFO）——注意 **E2 跨两条 link**。每条 link 各取一 turn 即足以恢复；minimal cuts 为 {E1,E2}、{E3,E4}、{E5,E6}、{E2,E7}，各 2 turns。删除请求「Forget that the firm we hired belongs to our CFO's wife」的相似度排名把证据打散到 **3（E1）、14（E3）、22（E2）、36（E5）、45（E6）、53（E4）、141（E7）**；主设置 pool=20 只含 E1、E3（分属两条 link，任何一条都割不断）。S2 在 pool=30 时 E2 已入池却**未被选中**（其文本「reads as a routine record of a hire」）；pool=40 删到 E5；**pool=50 才完成 cut**（删 E5+E6 清空 relationship link）；pool=60 再删 E4；**E7 不进任何已跑的 pool（≤60）**。实体图则完全不同：每个证据 turn 都距请求实体 **1 hop**（Ravi→E2,E5,E6,E7；Arden→E1,E2,E3,E4；Nina→E3,E4,E5,E6），作者称实体扩展「could place a complete cut within a ten-turn pool; its effectiveness as a deletion strategy remains untested」。

其余 B.3 可参数化数字（已证实）：pool=20 候选平均只含证据 4.5 / 8.1 turns（Turn RAG、LightMem），MemoryBank 仅 3.0；pool 放宽到 60 也只升到 5.8 / 5.9 / 4.5。pool=60 时 Turn RAG 候选含 aggregation 证据 6.9 turns、chain 仅 4.4；S2 后仍可恢复比例 aggregation 3–31%、chain 49–78%（三系统×两模型）。预算与池宽的边际（B.4 Table 11，DeepSeek–Turn RAG、S2、144 facts）：pool=20 时 budget 10→20，Res 仅 52.08→50.69（−1.39 点）而 off-target 多删 9.95 turns；budget=10 时 pool 20→60，Res −12.50 点且 off-target 反少 1.04。LLM 判断（S2 vs S1）的增益随 reach 增长：pool=20 时 5–15 点、pool=60 时 20–32 点。pool=60 时 S2 已命中 92–93% 的 sufficient sets（Turn RAG/LightMem），Res 仍 31–40%——命中绝大多数充分集 ≠ 完成割。

#### 具体修改建议

1. **【候选生成｜改规格条款】实体扩展不得被相似度截断，且按 link 记覆盖。** 附录 D 是直接证据：按相似度取 top-N 时，关键证据可排到 141 名、且任何实际跑过的 N（≤60）都漏 E7；而 1-hop 实体邻域全覆盖。ProvPurge 已有实体扩展候选——需要改的是规格细节：(i) 实体扩展候选集不适用相似度 top-k 截断（截断只可用于非实体通道）；(ii) 对多实体秘密，候选生成须逐个中间实体展开（firm→owner→spouse→role 的链式实体，而不是只展开请求中直名的实体）；(iii) 回执/预提交清单中记「每个请求实体 1-hop 覆盖的候选数」作为可核查量。这与 B.3 的总教训一致：瓶颈是 reach 不是 budget——**扩大删除预算（多删）几乎无收益（−1.39 点），扩大覆盖才有（−12.50 点）**；ProvPurge 若有任何「预算换安全」的参数，应反过来设。
2. **【探针构造（权重分配）｜改文档措辞，不改规格】Res 律只准用作风险先验，不准用作权重表。** 可落地的只有一条 pooled 换算：对手方/基线式删除每漏删 1 个证据 turn，residual 直喂恢复期望约 +13 个百分点——这可用于 MemCut 辅证的预期值标注与探针抽样优先级（存活证据越多的秘密，攻击探针权重越高；system/model 差异 ≤约 4 点，不值得分系统配权）。**明确没有修改价值的部分**：附录 B.3 不提供分秘密类型的拟合系数，因此不要按 fact assembly / relation chain / aggregation / temporal 给探针设不同的斜率权重；类型差异应通过「覆盖到的 k」进入，而不是通过类型标签。且不得用该律反推 ProvPurge 的删除充分性（complete cut 时该律失效，见上）。
3. **【评测口径｜改文档措辞】** 用 MemCut 作辅证时须注明其 S2 后 Res 39–76% 是 retrieval/LLM 基线的数字，且附录 D 的实体扩展效果是作者明言 **untested** 的假说——ProvPurge 的实体扩展闭包若在 BEMULA 上有效，那是 ProvPurge 自己的实验贡献点，不要引 MemCut 为其背书。

### 3. VeriForgot

**读到的层级：全文（Preprints.org 抓取正文：引言、§3 框架、§4 实验、§5 安全分析、结论、表 1–4）。** 来源：https://www.preprints.org/manuscript/202603.2325

#### 它到底是哪篇（已证实＋一处未消解）

- **已证实**：题名《VeriForgot: Blockchain-Attested Verifiable Machine Unlearning Using Membership Inference Oracles for GDPR Compliance》，载于 **Preprints.org（MDPI 预印本平台，非 arXiv、非同行评审）**，v1，页面标注 © 2026 by the authors。主题：为**参数化机器 unlearning** 提供第三方可核查的合规证明，面向 GDPR Art. 17。
- **未完全消解**：抓取正文未含作者行；二级来源（LinkedIn 文章，index 级）引其作者为 **Tulla & Chowdhury (2026)**、DOI 为 10.20944/preprints202603.2081.v1，与本次打开的 manuscript 编号 **202603.2325** 不一致。两个编号指向同题名、同关键数字（AUC 0.5918→0.4669）的文本，但作者与 DOI 的对应关系本次**未能确证**，引用时须标注此不确定性（见 Could not verify）。

#### 它验证的对象：参数（已证实）

验证对象是**训练后模型的参数/成员泄漏行为**，既不是记忆库，也不是 LLM Agent 行为：实验为 CIFAR-10 + ResNet-18，D_forget = 500 个 class 0/1 样本；三种 unlearning（Gradient Ascent / Selective Retraining / Strong GA）。三层验证：(1) **MIA oracle**——以 forget 集 vs 非成员集的置信度 AUC 为信号，阈值 **τ=0.57** 二值 PASS/FAIL；(2) **ZK 证明（Groth16 规格）**——证明 ‖θ_new − θ_orig‖₂ > δ 而不泄露参数；(3) **区块链 Unlearning Certificate**——智能合约在 MIA 签名与 ZK 证明双过后发证，字段含 cert ID、hashed subject ID、C_orig/C_new 承诺哈希、oracle verdict、timestamp。数字：baseline MIA AUC 0.5918；GA 0.5272 / SR 0.5604 / Strong GA 0.4669；oracle 在 20 例（10 真 unlearning / 10 加噪假冒）上 TPR 100%、TNR 90%（总 95.0%）；GA 的 test accuracy 仅降 0.03%。两个原文内的保留：唯一假阳性是 noise scale 0.0029 的假模型（AUC 0.5660，刚跌破 0.57），正文称把 τ 调到 0.58 即可消除并达 100%——即阈值是在同一批 20 例上事后调的（此为已证实事实；「过拟合阈值」是分析评价）；且结论 future work 明言还要「implement the full Groth16 ZK circuit」——**ZK 部分在本文中是协议规格，不是已实现的验证**。

#### 与 ProvPurge 探针成绩单回执的关系与修改建议

- **无冲突（已证实层面的对象差异＋分析）**：VeriForgot 的「可验证」= 第三方对参数模型做统计检验（MIA）＋防篡改存证；ProvPurge 的回执 = 操作方对一次外部记忆硬删事务签探针成绩单（行为探针全不过＋兄弟全过＋被删行缺席）。两者证据类型（统计/二值阈值 vs 确定性来源闭包＋行为探针全量记录）、对象（权重 vs 外部记忆）、信任方（外部审计者 vs 事务自身）都不同，不构成对 ProvPurge 回执的否定或替代。
- **【事务与回执｜改文档措辞＋至多一个字段级规格补充】可借的只有「预承诺」**：VeriForgot Phase 1 在 unlearning 开始**之前**先上链参数承诺 C_orig=H(θ_orig‖r)。对应到 ProvPurge：回执除签成绩单外，应绑定**删除前状态哈希与删除快照哈希**（防事后改写成绩单对应的状态）——若现行回执已有等价绑定则此条无修改价值、只在文档中点明与 VeriForgot 承诺阶段的对应关系即可。
- **明确不借（分析推断）**：(i) MIA 式单阈值二值 oracle——把全量探针成绩压成一个 AUC 阈值会丢掉逐探针信息，且其 FP 例子显示阈值边缘极脆；ProvPurge 的「全不过才提交」严格强于此类门，不应降格。(ii) 区块链存证——本地无训练、无外部审计方的设定下只增成本与元数据暴露面，无收益。(iii) 引用纪律：不得引 VeriForgot 作为「LLM/记忆遗忘可验证」的证据，它是图像分类参数 unlearning 的预印本，作者自述 LLM 扩展是 future work。建议在相关工作/宣称边界中用一句话完成定位区分：「可验证遗忘」在 VeriForgot 指参数侧统计验证＋链上存证，在 ProvPurge 指外部记忆事务侧的探针成绩单回执。

### 4. MemForm

**读到的层级：摘要＋§1–§5.3（问题表述、基准构造、主结果、防御评估），index。** 来源：https://arxiv.org/abs/2610.14431v1 ；https://arxiv.org/html/2610.14431

#### 主题与方法（已证实）

《MemForm: Benchmarking Memory Formation Poisoning in LLM Agents》（Pan, Shao, Ma, Jin, Zhai, Wang, Yang；2026-10-10 提交；PDF 标注 ICLR 2027 在审；代码 https://github.com/opanhw/memform-code ）。它研究的不是遗忘/删除，而是**记忆形成投毒**：对抗影响「经验如何被生产并提交为持久记忆」的接口。五接口——Content（终端写入内容）、Evidence（形成记忆的证据）、Scope（经验的适用上下文支持）、**Selection（固定候选池＋固定保留预算下更换保留子集）**、**Path（保留多重集与 updater 均不变，只换 consolidation 顺序）**。规模：235 个通过验证的语义族、1,016 编译案例、1,891 个 format instances（994 选择题＋897 自由作答）、六个 LLM、LangChain 主实现＋LlamaIndex 第二实现。结果：macro-average ASR 选择题 35.2–84.0%、自由作答 49.0–87.1%（Table 1），Path 列选择题 ASR 37.1–95.1%；摘要称 Path 易感性取决于 updater、接口适用性取决于记忆架构；五种防御都不完整，最好的 A-MemGuard 在 DeepSeek 上平均 ASR 仅降 8.2 点且主要作用于 Path（56.8→21.4，Table 2）。

#### 相邻性判断与修改建议（分析推断，依据上述已证实内容）

- **定位：相邻但不同侧，不构成方法竞争。** MemForm 是形成端（write-path）威胁基准：记忆**写入时**被污染如何改变下游行为；ProvPurge 是删除端事务：记忆**已形成后**按来源闭包硬删并探针把关。它没有删除方法、没有 provenance closure、没有遗忘评测。因此**不需要改 ProvPurge 的方法定位或基线集合**，MemForm 不进基线。
- **【宣称边界/相关工作｜改文档措辞】应进必引。** 理由：ProvPurge 的威胁叙事若涉及「记忆写入/形成不可信」，MemForm 是该侧的系统基准，审稿人很可能以它质询定位。一句区分即可：MemForm 刻画形成接口的投毒面（what/how 被提交），ProvPurge 处理已提交记忆的来源可追溯删除（提交后如何可验证地移除）。
- **【事务与回执｜改规格条款，小】一条真实的方法含义**：Path Poisoning 证明**保留集合完全相同、仅 consolidation 顺序不同，就能改变下游行为**（且该效应对 updater 敏感）。ProvPurge 的闭包删除若涉及把幸存记录重建/重整为摘要，则「兄弟保全」不是只由保留集合决定的——重建顺序本身是自由度。规格应补：(i) 删除后重建采用**确定性顺序**（如按来源 id 排序，与 AfterDeleteBench closure 的确定性重建做法同向）；(ii) 该顺序的标识（排序规则＋重建后状态哈希）记入回执；(iii) 兄弟保全探针必须在**重建后的实际状态**上执行，不得在重建前的保留集合上预跑后沿用结论。
- 除此之外**没有修改价值**：其 Scope Poisoning（局部有效经验被越界套用）与 ProvPurge 已并入的 F4 源-结论双语义关切重叠但更粗，不产生新条款；其防御评估（关键词过滤、LLM-Judge 等）是形成端防御，与探针门不可移植。

### 5. PolyGraph / Keep Your Receipts（arXiv:2610.17719）代码仓库状态复查

**读到的层级：论文全文 §3 脚注核对仓库地址＋仓库本身 verified live 打开。** 来源：https://arxiv.org/html/2610.17719 ；https://github.com/VishiATChoudhary/polygraph

- 论文中给出的 GitHub 地址（§3 脚注 11 原文）：**https://github.com/VishiATChoudhary/polygraph**，且正文称「The prompt templates for every model-facing step are released verbatim with the code.」
- **复查结果（verified live，本次 2026-10-30）**：在研究窗口直接打开该地址，GitHub 返回 **404「Page not found」**页面。**仍不可达**，与任务给定的 2026-10-29 的 404 结果一致。（说明：GitHub 对不存在与私有的仓库均返回 404，故只能断言「公开不可达」，不能断言仓库不存在——此为平台通则，非页面原文。）
- **具体修改建议【宣称边界｜改文档措辞】**：ProvPurge 文稿中凡涉 PolyGraph 的可复现性表述，不得写其代码「已公开/可获取」；应注为「论文脚注给出仓库地址，截至 2026-10-30 复查仍为 404、公开不可达」。除引用卫生外，本条**没有规格修改价值**（其句级 receipt 形状的教训属此前轮次，本次任务对该篇只要求仓库复查）。

### 6. TSD-AUDIT（arXiv:2609.19456）

**读到的层级：摘要＋方法概览（按任务要求）。** 来源：https://arxiv.org/abs/2609.19456 ；方法 §3 经索引摘录自其 PDF 文本（index 级）。

**身份先行（已证实）**：arXiv 列表题名是研讨会名《Beyond Private Training: The New Landscape of AI Privacy》（Culatana & Li，Atlassian，2026-09-16 提交，NeurIPS 2026 InfPriv workshop），论文内题名为《Beyond Output Filtering: Externally Auditable Erasure for Inference-Time RAG Memory》；TSD-AUDIT 是文中框架名，不是论文题名。

#### 「遍历安全删除证书」到底证书化什么（已证实）

证书化的是一个**逐查询的过程性质**：在给定的删除快照 D_t 下，这次 ANN 查询的遍历**从未对已删除向量计算过距离**。机制是 alive-before-scoring 不变量（alive bitset 在计距之前三处检查：入口点只取 live 节点、邻居入 frontier 前丢弃 deleted、pop 时复查）＋只用 live 候选的连通性修复。证书内容（§3 Certificate 段）：每个 checkpoint 发出 **deleted-set hash、抽样 expansion traces 的 hashes、以及 visited∩D_t 的 verdict**；独立 verifier 重算哈希并断言 no-intersection——即「scored/visited 集合与删除集不相交」可由第三方对删除快照核验。实证锚点：Faiss IndexHNSWFlat 原生过滤不减少距离计算数，70% 删除率下 trace-faithful replay 在 100/100 审计查询中检出 deleted-vector scoring；replay 保真度经验证（top-10 agreement 1.000、距离计数 0.98× Faiss 自带 ndis）；soft-delete 的证书全数失败，traversal-safe 与 clean-rebuild 通过（0 hash/count mismatches）。作者自我限定：贡献是「audit-not-algorithm」，enforcement 本身是已知的 predicate-aware ANN 过滤遍历；**at-rest 可恢复性不在其范围**（那是其所引 Ghost Vectors 的题目）。

#### 与回执成绩单的边界再确认（分析推断，基于上述已证实构成）

两者**互不蕴含、互补**：

| | TSD-AUDIT 证书 | ProvPurge 探针成绩单回执 |
|---|---|---|
| 证书化对象 | 检索**过程**：某查询的 scored 集 ∩ 删除集 = ∅ | 删除**事务的结果状态**：攻击探针全不过＋兄弟保全全过＋被删行缺席 |
| 时间粒度 | per-query、checkpoint、抽样 traces | per-deletion-transaction、全量探针 |
| 核验依据 | 删除快照哈希＋trace 哈希重算 | 探针成绩单签名＋缺席检查 |
| 不证明 | 内容不能从存活项重构、向量在盘上已抹除、行为不披露 | 检索过程从未触碰已删除向量 |

#### 具体修改建议

1. **【事务与回执｜改规格条款】「被删行缺席检查」必须分层定义。** 这是本次 TSD 复核唯一实质修改点：ProvPurge 现行表述只说回执验证含被删行缺席检查，但 TSD 证明「未出现在返回结果中」（output safety）与「遍历时未被计分」（traversal safety）是可分离的两回事——软删系统可以前者成立、后者全败（100/100 查询中招）。规格须写明缺席检查到底断言哪一层：(a) 存储层不存在；(b) 检索不返回；(c) 若系统含 HNSW/ANN 图索引，还须 (c1) 遍历不计分（要求 alive-before-scoring 或删除后重建索引）或 (c2) 明确把宣称限定在 (a)(b) 层。若 ProvPurge 的实现根本不经 ANN 索引（纯存储＋直喂探针），则退化为**改文档措辞**：一句「本方法的缺席指存储层与输出层，不含图遍历层，因系统不使用图 ANN 索引」即可封口；但这句必须写，不能留空让审稿人按 TSD 的区分来问。
2. **【宣称边界｜改文档措辞】术语隔离。** 不把 ProvPurge 回执称为 TSD 意义上的「删除证书 / deletion certificate」；两者并列引用时按上表一句话交代边界：TSD 证书证明「没碰过」，ProvPurge 回执证明「剩下的拼不出且兄弟还在」。

---

## 可修改点清单

| # | 改什么 | 依据（哪篇哪节） | 成本档位 | 不改的风险 |
|---|---|---|---|---|
| 1 | 探针成绩单改双通道：每条攻击探针同记解析判定、exact-target-anywhere、解析合同版本/哈希；两者冲突以 raw 为准拒绝提交 | AfterDeleteBench §3、§5.4、附录 E/G.1–G.2/I（legacy 下 raw 含目标 77–78/104 而官方 0；合同切换可使 E 77/104→0/104、write 效应反号 −0.224） | 改规格条款 | 探针门被解析器错位假阴性放行——删除未净却签发回执，是内核级失效 |
| 2 | 提交判定加组件级部分泄漏行：被删秘密的组件在探针回复中出现单独记账，不得并入「全不过」 | AfterDeleteBench §5.2、附录 G.3 Table 11（C3 完整恢复 0 而 region/mnemonic 出现达 104/104） | 改规格条款 | 对外宣称「探针全不过」时实际存在部分泄漏，宣称与事实不符 |
| 3 | BEMULA 评测补 never-exposed 配对对照；进阶补 exposed-and-retained 参照臂（子集即可） | AfterDeleteBench §3 配对设计（clean 0/200）；附录 J 自承缺 retained 参照 | 改实验设计 | Δ 无法归因于暴露；且错失一个相对该基准的现成评测增量点 |
| 4 | closure/闭包零恢复的措辞限定为「来源完备且已追踪通道内」；禁用其原始 compositional 0/50 论证组合型安全 | AfterDeleteBench §5.1/§5.2 对照、§6、附录 J | 改文档措辞 | 过度宣称语义擦除；被 C1 的 62–99% 直接反驳 |
| 5 | 候选生成：实体扩展不被相似度 top-k 截断；逐中间实体链式展开；回执记每实体 1-hop 覆盖数；参数优先级「扩覆盖 > 加预算」 | MemCut 附录 D（排名散至 141、E7 不入任何 pool、实体 1-hop 全覆盖）、B.3/B.4（budget 10→20 仅 −1.39 点且多删 9.95；pool 20→60 −12.50 点） | 改规格条款 | relation chain 型秘密系统性漏检（基线 Res 71.8%、S2 后仍 49–78% 可恢复），且多删伤兄弟 |
| 6 | Res≈13(8.2−k) 只作 pooled 风险先验与探针抽样优先级（按存活证据数，不按 system/model/类型分权）；禁止据此设分类型权重或反推删除充分性 | MemCut §6.1、附录 B.3（132 设置 pooled 拟合 R²=0.95；无分类型拟合；complete cut 时失效，minimal cut 均值 3.04） | 改文档措辞 | 伪造不存在的分类型参数；或误用线性律否定割/闭包路线的充分性 |
| 7 | 回执绑定删除前状态哈希＋删除快照哈希（预承诺）；相关工作中一句话区分两种「可验证遗忘」；不引入 MIA 阈值门与区块链 | VeriForgot §3.3 Phase 1 承诺、§3.2/§5.1（τ=0.57 与 FP 0.5660、事后调阈值）、结论（ZK 待实现、LLM 为 future work） | 改文档措辞（绑定字段若缺失则升为改规格条款） | 成绩单与状态的绑定可被质疑；或被误引为同类工作而混淆验证对象（参数 vs 记忆） |
| 8 | MemForm 进必引（形成端投毒基准），一句定位区分；不进基线、不改方法定位 | MemForm 摘要、§3–§4（五形成接口、1,016 案例） | 改文档措辞 | 相关工作缺口被审稿人点名；与形成端威胁的边界不清 |
| 9 | 删除后重建：确定性 consolidation 顺序＋顺序标识/重建后哈希入回执；兄弟探针在重建后状态上执行 | MemForm §4.1 Path 定义、Table 1（Path ASR 37.1–95.1%、保留集不变仅顺序变） | 改规格条款 | 兄弟保全结论随重建顺序漂移且不可复现，回执无法复核 |
| 10 | PolyGraph 引用注记改为「仓库地址见论文脚注，截至 2026-10-30 复查仍 404、公开不可达」，不得称代码已公开 | PolyGraph §3 脚注 11；2026-10-30 verified live 打开 https://github.com/VishiATChoudhary/polygraph 得 GitHub 404 | 改文档措辞 | 引用一条不可达的代码链接作可复现性依据，引用卫生问题 |
| 11 | 「被删行缺席检查」分层定义（存储/输出/遍历计分）；不用 ANN 则写明不含遍历层；回执不称 TSD 意义的 deletion certificate | TSD-AUDIT 摘要＋§3（output vs traversal safety；证书 = deleted-set hash＋trace hashes＋visited∩D_t verdict；soft-delete 证书 100/100 失败） | 改规格条款（不用 ANN 时为改文档措辞） | 「缺席」被按最强层级解读或质询，软删式反例（返回缺席但遍历计分）击穿宣称 |

无修改价值、明确不改的：VeriForgot 的 MIA oracle 与区块链存证（对象与设定均不合，见 §3）；MemCut 的分类型探针权重（数据源不存在，见 §2）；MemForm 的防御套件与 Scope 条款（与 F4 重叠且更粗，见 §4）；PolyGraph 的方法内容（本次仅复查仓库，见 §5）。

## Could not verify

- **VeriForgot 作者与 DOI**：Preprints.org 抓取正文未含作者行与提交日期行；二级来源（LinkedIn 文章）给出作者 Tulla & Chowdhury (2026) 与 DOI 10.20944/preprints202603.2081.v1，但本次打开的 manuscript 编号为 202603.2325，两编号关系未能确证。题名、平台、年份（© 2026）、内容与全部数字均以 202603.2325 正文为准。
- **MemCut 附录 B.3 的分系统拟合**：B.3 的 pooled 拟合覆盖三系统（Turn RAG、MemoryBank、LightMem 的 sweep 子集），Session RAG 与 Nemori 不在该拟合内；正文 Table 3/4 的分系统数字已读，但「分系统独立拟合参数」在已读文本中不存在，按「未提供」处理而非「不存在于全文任何角落」——附录 E（Additional Related Work）未读，与本任务无关。
- **TSD-AUDIT 方法细节深度**：按任务要求只读摘要＋方法概览；§3 的证书构成引自索引摘录的 PDF 文本（镜像站），未逐行读其全文实验节，故其 Recall@10 等数字只引摘要口径。
- **MemForm 附录**：本次为确认主题/方法/定位，只读到 §5.3，未读其附录 B 的分接口诊断与 updater 对照表；Path 易感性「取决于 updater」一句依据其摘要，未核到附录原表。
- **AfterDeleteBench 的代码仓库**：其复现声明称代码「accepted 后公开」，本次未去核验其仓库状态（不在任务清单内）。

## Sources

- AfterDeleteBench, arXiv:2610.31366, 全文 HTML（含附录 A–J）, https://arxiv.org/html/2610.31366 ，2026-10-30 读，index 全文。摘要页：https://arxiv.org/abs/2610.31366v1
- MemCut, arXiv:2610.21405v1, 全文 HTML（正文 §6、附录 A–D）, https://arxiv.org/html/2610.21405v1 ，2026-10-30 读，index 全文。
- VeriForgot, Preprints.org manuscript 202603.2325 v1, https://www.preprints.org/manuscript/202603.2325 ，2026-10-30 读，index 全文。作者/DOI 二级来源（index）：https://www.linkedin.com/pulse/playbook-generating-cryptographic-proofs-ai-mohit-sewak-ph-d--1pp7f
- MemForm, arXiv:2610.14431v1, 摘要页 https://arxiv.org/abs/2610.14431v1 与全文 HTML https://arxiv.org/html/2610.14431 ，2026-10-30 读，index。
- PolyGraph / Keep Your Receipts, arXiv:2610.17719v1, 全文 HTML https://arxiv.org/html/2610.17719 ，2026-10-30 读，index；其脚注仓库 https://github.com/VishiATChoudhary/polygraph ，2026-10-30 在研究窗口 verified live 打开，结果为 GitHub 404「Page not found」。
- TSD-AUDIT（载于 Beyond Private Training / Beyond Output Filtering, arXiv:2609.19456）, 摘要页 https://arxiv.org/abs/2609.19456 ，2026-10-30 读，index；方法 §3 摘录自 PDF 文本镜像 http://arxiv.cnoctave.top/pdf/2609.19456 ，index 级。
- 工作笔记：notes/source-01-afterdeletebench.md、notes/source-02-memcut-appendix.md、notes/source-03-veriforgot.md、notes/source-04-memform.md、notes/source-05-polygraph-repo.md、notes/source-06-tsd-audit.md
