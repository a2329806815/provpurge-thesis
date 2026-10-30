# ProvPurge 撞车精读报告：MemCut / LETHE（重点精读）+ FEB-30 / TRACE（核查）

> 对象方案 **ProvPurge**（拟定）：写时句级来源 + 确定性级联清除（事实/向量/摘要句三层）+ 删除后攻击探针在线自验 + 删除回执；主基准 BEMULA + LongMemEval-S 知识更新子集 + STALE；三条贡献：(1) 三层冗余副本的句级来源链；(2) 带在线自验的硬删事务；(3) 三协议联合评测。
>
> **证据纪律**：下文标【原文】的为实际打开全文/仓库页面读到的事实（来源见文末 Sources，标注 index 全文 / verified live）；标【推断】的为本报告基于原文的分析判断，不是任何论文的结论。获取不到的一律明写未获取。

## Summary

- **四篇全部真实存在且全文可读**：MemCut（arXiv:2610.21405v1，2026-10-15）与 LETHE（arXiv:2606.15903，题 *Control-Plane Placement Shapes Forgetting*）HTML 全文逐段读完（含全部附录）；FEB-30（arXiv:2610.22209v1）与 TRACE（arXiv:2609.33517v1）读到正文方法、实验、局限全量（超「摘要+方法概览」要求），仅附录未逐项读。
- **一句话判决**：ProvPurge 的「回执」被 LETHE 几乎全额占用（且是密码学签名的完成版）；「来源链+级联清除」的机制内核被 MemCut 的 turn 级来源记录 + closure 删除先行实现（在评测协议里）；「联合评测」被 LETHE、TRACE 的跨基准实践压成常规操作。**真正没被占住的只剩一个内核：把 MemCut 式的残余恢复探针做成删除事务的在线提交门（probe-gated commit），探针不过就不签回执、自动扩候选再删**——详见「对 ProvPurge 的判决」。
- **一个对 ProvPurge 叙事最危险的事实组合**【原文】：MemCut 证明残余风险的主体不是「副本没删干净」，而是**可推导性**——零衍生的逐字存储（Turn RAG）经 LLM 引导删除后 residual recovery 仍 41–46%；LETHE 证明纯确定性删除在规范化类攻击上 ≤5%/0%。ProvPurge 的「来源链管副本 + 确定性级联」这两根支柱，分别被这两篇各打穿一半。

---

## Findings

### 1. MemCut（Zhang et al., arXiv:2610.21405）全文精读

【原文】作者 Cheng Zhang、Xuanning Cao、Zhicheng Zheng、Hui Xiong（HKUST 广州 / USTC），2026-10-15 提交。**定位先说清：MemCut 是一篇基准论文，不是删除方法论文**——它提出的三条删除基线是刻意「最小适配复用现有组件」的 reference baselines。这一点决定了它对 ProvPurge 的威胁方式：它占的是*问题定义、评测协议与来源/closure 机制*，不是*删除算法*。另注意：MemCut 的参考文献已经引用了 LETHE（2606.15903），两篇在同一引用网络内。

#### 1.1 研究问题与任务定义

- 问题：用户要求遗忘一个敏感事实后，该事实是否仍可从长期记忆中**被推导恢复**——包括从从未直接陈述该事实的普通记录中联合推出。
- 形式定义（§3.1）：设 disclosure 为 $D$、evidence 为 $E$、minimal sufficient sets（足以推出事实的极小证据组合）为 $\mathcal{S}$。删除集 $X$ 构成一个 **cut** ⟺ $D\subseteq X$ 且 $X$ 与**每一个** $S\in\mathcal{S}$ 相交；minimal cut 是最小的这种 $X$。即：遗忘 = 打断全部推导路径，而不是删掉全部相关记录。
- deleter 接收自然语言删除请求并从 store 删对象；之后用**四选一问题**（可拒答，错误选项只差一个细节，且**两种选项顺序下都答对**才算恢复）测可恢复性。

#### 1.2 数据结构与构造（标注粒度是这篇的核心资产）

| 项 | 数值/做法【原文】 |
|---|---|
| 秘密总数 | 456 个，手工设计（作者设计事实结构，GPT-6 只渲染对话措辞，逐 turn 人工核查修订） |
| 规模 | 15 个 episodes，43,402 turns / 490 sessions / 1.74M tokens；每 episode 均 2,893 turns、跨 27–86 天、约 30 个事实；人工 ≥12 小时/episode |
| 每事实 evidence | 均 8.1 turns（6–10），跨 8.0 sessions；与 disclosure 同 session 的仅 1.8% |
| minimal sufficient sets | 均 15.7 个/事实（共 7,148），每 set 2–5 turns（均 2.7），**无 size-1**（单个证据永不泄露） |
| minimal cut | 均 3.04 turns，max 5（清理预算 10 的一半） |
| distractors | 均 26.6 个/事实（形似而不支持该事实） |
| 四种推导类型 | fact assembly（同实体属性拼接）/ relation chain（跨实体链）/ aggregation（数值聚合，删任一值即断）/ temporal state（时序状态），各 113–115 个 |
| 标注可靠性 | 双标注初始一致 97.6%；minimal cuts 由枚举 + 独立动态规划两算法对全部 456 个交叉验证一致 |

校验（§3.3，Qwen3.8-Flash 作独立 reader）：组合 evidence 恢复 96–99%；最佳单个 evidence turn 恢复 **0%**；删 disclosure+evidence 后 0%；无记忆答题 0%。即评测测的确实是「联合推导」，不是猜测或单条泄露。

#### 1.3 两条恢复指标与 hidden-risk 的精确定义（ProvPurge 必须照此口径说话）

- **Native recovery**：用事实的问题经**系统自身检索**提问，同一 reader 模型作答——测「助手还会主动说出什么」。
- **Residual recovery**：**绕过检索**，把所有溯源到该事实 disclosure∪evidence 的**存活对象直接全量喂给同一 reader**——测「记忆里还存着什么可推导的东西」。三条配套设计：①同一模型答两边（reader 固定）；②清理与 native 提问用同一检索器、分别以删除请求和问题为 query；③**删一个 turn 即删其全部衍生对象（closure）**，residual 只看 deleter 留下的东西。
- **Hidden-risk share** $= (\mathrm{Res}-\mathrm{Nat})/\mathrm{Res}$。附录 B.1 证明它是「residual 成功中被 native 漏掉的份额」$|R\setminus N|/|R|$ 的**下界**（因 native 独立作答会多答对少数 residual 答错的，实测低估 0–12 点，S2 后最大）。

#### 1.4 删除方法：三条 reference baselines 全流程（哪些步用 LLM、哪些确定性）

| 基线 | 流程【原文 §4.1】 | LLM 参与 |
|---|---|---|
| S0 | 删 disclosure 及其全部衍生对象（closure） | 无（按标注/来源确定） |
| S1（retrieval-ranked cleanup） | 以删除请求 $q$ 对系统原生检索器**单次查询**，取 **20 个 turn 作候选池 $P_q$**，删排名 **前 10** | 无 LLM 判断；检索为稠密相似度 |
| S2（LLM-guided cleanup） | **同一 20 候选池不变**，由 write model 按存储文本重排，删前 10 | LLM 只做池内重排/选择 |

- 失败二分：**Reach 失败**=证据没进池；**Choice 失败**=进了池没被选。S1/S2 均不见任何标注。预算论证：每事实 evidence ≤10 turns、minimal cut ≤5，故预算 10「够用」的前提是证据**够得着且被选中**——论文就是要分离这两者。
- **统一协议（§4.2，与 ProvPurge 来源链正面对撞，见判决节）**：所有系统以 turn 为统一单位——**写入时记录每个对象的 source turns**，池与预算按 turn 计，删除按 closure 执行（删一个对象=删其源 turn 牵连的全部对象）；系统只需支持 list / search / delete / snapshot 即可接入，作者称已 release 该 harness。
- 五个记忆设计：Turn RAG、Session RAG（逐字存储、零衍生）+ MemoryBank（原文 + daily summaries）、Nemori（segment 重写为 narratives）、LightMem（离线 consolidated summaries，不保留原文）。四个 write models（DeepSeek-V4.1-Flash、Qwen3.8-Flash、GLM-5.3-Flash、MiMo-V2.5），**同一模型兼任写入、候选打分、reader**；每事实在 store 的 fresh copy 上独立评测。

#### 1.5 主要结果数字

- 删除前 residual recovery 94–100%（逐字库 100%）——残余不是写入丢失造成的。
- **S0**：native 降 12–50 点（Turn RAG 91–95%→57–60%），但 residual 仍 **85–99%**。只删披露≈只降可见性。
- **S1**：residual 仍 **48–86%**；注意 Turn RAG（零衍生）仍 59–62%——**残余不需要摘要/改写副本存在，逐字原文的分散证据本身就够**。
- **S2**：再降 7–17 点，residual 仍 **39–76%**（即任务书已知线索的出处：LLM 从 20 候选中再删 10 turns 之后）。同系统内四个 write model 的 residual(S2) 差异 ≤约 5 点（Session RAG ≤10 点）——结论对模型选择不敏感。
- **Hidden-risk 随删除上升**：Turn RAG 由删除前 5–9% → S0 后 39–42% → S2 后 **76–82%**；20 个配置全部上升，S2 后 14/20 超过一半；Session RAG S2 后 83–92%（检索粒度越粗，藏得越深：其 3 个 session 只含 1.9 个 evidence turns，Turn RAG 16 turns 含 4.8 个）。
- 分类型（S2 后 residual 均值）：relation chain **71.8%** 最高（hidden-risk 87.1%），fact assembly 60.2%，temporal 58.4%，aggregation 24.3% 最低。衍生强度与 residual 无一致关系：衍生最强的 LightMem（39–43%）≈Turn RAG（41–46%），衍生最轻的 MemoryBank 反而最高（72–76%，因其候选覆盖最差）。

#### 1.6 消融：候选池/预算扫描——「覆盖率瓶颈」的定量形态（§6.1、附录 B.3，144 事实子集 EP01–05、三系统两模型）

- **加宽池子有效**：预算固定 10，池 20→60 使 S2 residual 降 **12.5–23.9 点**。
- **加预算近乎无效**：池固定 20，预算 10→20（删光全池）residual 只再降 **≤6.5 点**；删全池后 Turn RAG/LightMem 仍 43–51%、MemoryBank 仍 66–72% 可恢复。
- **一条近乎定律的拟合**：132 个 sweep 设置上，$\mathrm{Res}\approx 13\times(8.2-k)$（$k$=平均删掉的 evidence turns 数），$R^2=0.95$；给定 $k$，系统+模型+策略合计位移 ≤4 点。外推到 $k=8.2$（≈平均事实的全部 8.1 条 evidence）才归零。
- **池内证据含量**：池 20 时 Turn RAG/LightMem 含 4.5 条、MemoryBank 仅 3.0 条（共 8.1）；池 60 时升至 5.8 / 5.9 / 4.5——**加宽到 60 仍够不着全部证据**。
- **选择精度**：S1 的 10 个删除中只有 3.0 个是 evidence，S2 为 3.6 个 → off-target 约 **70% / 64%**；LLM 判断相对检索排名只多删中 **0.7 条** evidence/事实。分系统 S2 删中 evidence：MemoryBank 2.9、Nemori 3.4、LightMem 4.7；分类型：relation chain 仅 **2.4**，assembly 3.8、temporal 3.9、aggregation 4.6。
- **判断的价值随 reach 增长**：S2 相对 S1 的增益，池 20 时 5–15 点，池 60 时 **20–32 点**——先有覆盖，才谈得上选择。
- **partial cut 不够**：池 60 时 S2 已平均打断 92–93% 的 sufficient sets（Turn RAG/LightMem），residual 仍 31–40%——必须 complete cut（打断**全部**推导组合）。
- **成本对照（Table 11，DeepSeek–Turn RAG）**：池20/选10 → off-target 5.54、Res 52.08%；池20/选20 → off-target 15.49、Res 50.69%（多删 9.95 条非证据只换 −1.39 点）；池60/选10 → off-target **4.50**、Res **39.58%**（降 12.50 点且还少删 1.04 条非证据）。即扩池同时改善遗忘与附带损伤，加预算则相反。

#### 1.7 案例研究（附录 D，ProvPurge 候选扩展可直接抄的灵感出处）

关系链事实「公司雇的 firm 属于 CFO 之妻」：4 条 link、每条 2 个 evidence turns，删空任一 link 的 2 条即成 cut（minimal cut 仅 2）。证据对删除请求的相似度排名散在 **3–141**；池 20 只含 E1、E3（属不同 link，删了也不成 cut）；池 30 时 E2 入池却未被 S2 选中（它「读起来只是例行雇佣记录」）；池 50 才凑成 cut；E7 在所有池宽下从未入池。作者明写：**每个 evidence turn 都与请求提到的实体（Ravi / Arden / Nina）1-hop 相连，entity expansion 本可在一个 10-turn 池内凑齐 complete cut，但「its effectiveness as a deletion strategy remains untested」**——这是作者亲手留的空位。

#### 1.8 作者自述局限

①受控构造：部署中 evidence trail 的真实频率未知，需部署研究；②reference baselines 只复用现有检索/写入组件，结论限于「该复用策略」，MemCut 本身支持评测专门的 erasure 方法（扩覆盖、改索引）；③只研究文本记忆，多模态与参数记忆不在范围；④source-level 附带损伤度量不捕获 closure 删掉的无关内容，也不测下游效用损失（retained task performance 是独立维度）。

#### 1.9 代码是否公开且可打开

【原文】正文两处表述：§4.2「we release its harness」、Ethics「The benchmark will be released for research use」——但**全文未给任何仓库 URL**。2026-10-29 本报告两轮定向搜索（含 arXiv 编号 + GitHub 关键词）**未找到 MemCut 的公开仓库**。结论：**未获取**——按「已承诺释放、截至核查日未验证可打开」处理，ProvPurge 若要复现 S1/S2 只能按论文参数自研（参数已足够完整，见判决节）。

---

### 2. LETHE / Control-Plane Placement（Yang, arXiv:2606.15903）全文精读

【原文】单作者 Dongxu Yang（DeepLethe）。**版本注意**：arXiv 版题为 *Control-Plane Placement Shapes Forgetting*（13 配置研究）；其 GitHub 仓库内还有一版更早的同系列论文 PDF（2026-05-13，题 *ForgetEval: Benchmarking the Forgetting Axis of Agent Memory Systems*，23 页）——**签名回执的字段级规格只在仓库版论文 §5.4 与仓库代码里，arXiv 版正文只在附录 A 提了一句**。下文凡回执细节均注明出处层级。

#### 2.1 控制面/数据面划分是什么意思

论文的术语是 **recall plane vs control plane**（即任务书说的控制面/数据面之分）：

- **Recall plane（召回面）**：只读热路径，按查询检索已存事实——现有基准（LongMemEval 的 R@k、LOCOMO 等）全在测这一面。
- **Control plane（控制面）**：对记忆施加变更的操作面——**supersede（取代）、release（释放/软删）、purge（按标识符硬删）**。论文核心问题不是「要不要 LLM」，而是 **LLM 坐在哪个位置**：inscribe-time（写入摄入时做规范化/抽取）、mutation-time（执行 supersede/release/purge 时做意图判断）、还是不放（纯确定性原语）；召回热路径在所有配置下保持 LLM-free。
- 两面非对偶的实证：MemPalace 在召回面可达 Memora 60/150，却在 ForgetEval 对抗集 **0/385**——召回好完全不蕴含可被命令遗忘。

**它做哪一层的遗忘（任务书点名问题）**：论文 §2 明写「we target the memory-store layer alone, **no parameter access**」。即：**外置记忆存储层**——不做参数 unlearning/编辑（那是它明确划走的另一支文献），也不治理 agent 的执行态/上下文（其 event log 是存储层自带的审计日志，不是运行态）。其 store 内部的三张子表（文本/向量/FTS）同属一个系统、一把 rowid，见 2.3。

#### 2.2 ForgetEval：任务与指标

- 规模：**1000 案模板套件**（确定性生成，seed=42，每案 4 个 distractors）+ **385 案对抗层**（132 手工 core + 253 由 DeepSeek-V3 起草、经两阶段准入：结构过滤 + Qwen-2.5-72B 独立 judge 按 oracle 规则验证；10 标注者 100 案 IAA Fleiss κ=0.958；另有 77 案外部作者盲写子集）。
- 五个结构族：supersession（新事实胜出、旧事实退出 top-k）/ decay（TTL/OTP 释放后不回 top-k）/ amnesia（删光某实体的全部事实、兄弟实体存活——难点是宽度控制）/ **purge（按 identifier 硬删，GDPR Art.17；语义相似是错的原语）** / drift（取代链只有最新可达）。
- 用例即一个 dataclass：`setup_facts`（含 distractors）、`mutations`（supersede/release/purge 调用）、`final_query`、`must_contain`、`must_not_contain`。**通过条件：must_contain ⊆ top-10 召回文本 且 must_not_contain ∩ top-10 = ∅；确定性子串匹配，评测环路无 LLM judge**（LLM judge 只用于数据准入）。
- 对抗 10 类：substring_trap、prefix_collision、paraphrase_supersession、negation_trap、temporal_qualifier、shared_attribute、compound_fact（需 partial-edit 原语，作者自认是 primitive-existence 测）、identifier_obfuscation、cross_lingual_identifier、recursive_supersession，每类约 36–40 案。
- **Adapter Protocol**：3 个必需方法（reset / inscribe / recall_texts）+ 3 个可选变更方法（supersede / release / purge，未实现抛 NotImplementedError 记 **N/A**，区分「没提供」与「提供了但失败」）；行为契约——只暴露 add+delete 的后端可以用组合实现 supersede 并通过同样的测试；适配器约 130 行。

#### 2.3 参考实现 Lethe 的数据结构与方法全流程

- 单个 SQLite 文件：`memory`、`memory_vec`、`memory_fts` 三子表共用 rowid，外加 **append-only `event` 日志表**与 `supersession` 边表，无外部服务。
- **单标量 `depth` 编码全部遗忘状态**：+∞ pinned；1.0 surface；(0,1) sinking；0 submerged but logged；**void erased**。每个状态转移往 event 表写一行（depth_before/depth_after），由此支持 time-travel `recall(query, at=T)` 与签名回执。
- 检索双原语：`recall(query)` = 向量 + BM25 的 RRF 融合；`recall(query, lexical=True)` = 纯 BM25。**purge 走纯 lexical**——按 identifier 精确查找，而非语义相似（这是它 prefix_collision 82% 而 naive SQL LIKE 只有 28% 的机制；附录 T：naive DELETE-LIKE 基线总分 40.5% vs Lethe 63.4%）。
- **purge 是真硬删**【仓库版论文 Prop.3 原文】：行从 memory、memory_vec、memory_fts **三表全删**，AUTOINCREMENT 保证 rowid 不复用（防已删行被新写入冒名、破坏回执对历史状态的唯一指认）；event log 只保留 purge 事件作审计、没有任何操作能逆转 purge。release 则是软删（depth 压至 submerged，TTL/OTP 语义）。
- LLM hook（mutation-time）：三个窄 JSON 契约 prompt——supersede planner、purge match、release match（arXiv 版附录 B 只列标题，正文未展开全文）；hook 只在三类变更操作时触发，$0.17/385 案、约 2.3 s/案（确定性 64–191 ms/案）。

#### 2.4 主要结果与消融

| 结果【原文】 | 数字 |
|---|---|
| 模板集 1000 案 | Lethe 99.3%（993）、LangGraph InMemory 99.5%、Mem0 88.8%（amnesia 70%、purge 75% 塌陷）、MemPalace 0% |
| 对抗集 385 案（确定性） | Lethe 63.4%、Mem0 68.3%、LangGraph 62.9%——「63–68% 带」，Wilson CI 互叠，Lethe vs LangGraph 的 McNemar p=0.724（聚合分无显著差异） |
| +mutation hook | Lethe+DeepSeek-V3 **91.7%**（去 compound_fact 93.3%），LangGraph+同 hook **93.2%**（94.2%）——提升跨后端一致，归因于 hook 模式而非存储引擎 |
| 三种放置的互补性 | 确定性：词法/时间类 ≥95%，但 identifier_obfuscation **≤5%**、cross_lingual **0%**；inscribe-LLM：规范化 **100%**，但 prefix_collision / compound_fact **0%**；mutation hook：compound 78–85%、prefix 79%。非 primitive-existence 的 345 案：确定性 70.1–70.7% → hook 93.3–94.2%（**+22.6~24.1 pt**） |
| 跨评测 | Memora-weekly：Lethe 31% / LangGraph 45% / MemPalace 40%（轴翻转证据）；FactConsolidation 400 题：单跳 100%、多跳 17–37%，hook 变体与确定性同分（该基准不触发变更，hook 不点火）；LongMemEval-S（附录 Q，session 粒度）：Lethe R@5=**93.8%**，knowledge-update 子类 0.987【注：仓库 README 的 R@5=97.4% 是 sub-document QA 另一口径，引用时必须注明口径，勿混用】 |
| 消融 | 换多语 embedder 对 cross_lingual 无效（purge 不走向量）；hook 换 Qwen-2.5-72B 只 +13 pt，Llama-3.1-70B 因 JSON 契约解析失败回落确定性——提升幅度随 LLM 的 JSON 跟随能力缩放；Mem0 infer=True（写入时 LLM 路由）总分 68.3%→**43.6%**（规范化 +45~53 点、删除精度类 −31~92 点，LINK-AND-KEEP 之害）；外部 77 案子集上确定性带跌至 28–33%，且 Lethe+LLM 在 cross_lingual 反为 0/8 |

#### 2.5 签名删除回执：签什么、怎么验（任务书点名问题）

**先纠正定位**【原文】：在 arXiv 版论文里，回执**不是贡献、不是实验对象**——附录 A 仅一句：每次状态转移写一行 append-only event 表，「enabling time-travel queries and **signed purge receipts**」；正文与附录 B–V 没有任何回执评测。仓库 README 的 Roadmap 甚至把「Receipt-verification benchmark family（系统能否产出可审计的删除证明）」列为**下一步待做**。字段级规格在仓库版论文 §5.4 与代码里，如下：

- **签发**【仓库版论文 §5.4 原文】：`purge_with_receipt()` 发出 `PurgeReceipt`，内容含四项：①被删的 **row ids**；②删除**之前** purged texts 的 **SHA-256 哈希**；③对应 purge 事件的 **event-log IDs**；④签发时刻**全 event log 的 Merkle root**。以该 Lethe 实例的 **Ed25519 私钥**签名，签名对象为 **canonical CBOR payload**；签发/验证复杂度 O(log n)（对 event log 规模），亚毫秒级。
- **第三方能验三件事**（无需接触已删数据）：(a) 到底删了哪些行；(b) 签发之后 event log 未被篡改（Merkle root 对不上即失败）；(c) 回执确由合法运营者签发。
- **验证流程（代码实证）**【verified live，recipes/02_gdpr_purge_receipt.py，72 行】：`generate_key()` 生成密钥 → `Lethe(..., signing_key=private_key)` → inscribe → `purge_with_receipt([alice_id])` → 回执 `to_dict()` 打印键为 `purged_ids / purged_text_hashes / event_log_merkle_root / signature` → 两种验法：`verify_receipt(receipt, agent)` **带 DB 验**（重算 Merkle root + 验签，返回 `valid`）；`verify_receipt(receipt, lethe=None)` **仅验签**（返回 `signature_valid`，供监管方无 DB 访问时用）。【注：该脚本 docstring 称回执为「Ed25519-signed **JSON** document」，与仓库版论文的「canonical **CBOR** payload」表述不一致——文档/版本漂移，引用时以代码行为为准并注明】
- **回执证明的边界**（这是判决节的关键）：它证明的是「**某次 purge 事件真实发生、删的是这些行、日志事后没被改**」——一个**密码学事件证明**。它**不包含任何删除后行为验证结果**（删完之后还能不能问出来/推导出来，不在回执内、也未被 ForgetEval 测过）；它锚定的对象是**单个 Lethe store 内部按 rowid 指认的行**，其级联范围就是该 store 的三张子表，不涉及跨系统衍生副本、更不涉及 MemCut 式的可推导证据。

#### 2.6 作者自述局限

子串评分有盲点（50 案审计中 30% 的确定性失败是 scorer artifact，集中在 prefix_collision 与条款级 supersession，但承载主结论的规范化三类零 artifact）；英文为主、跨语言每脚本样本薄（1–3 案的语系无法区分系统）；compound_fact 是原语存在性测而非遗忘能力测；253 案由 LLM 起草 + 单 judge 准入，与 hook 同为 DeepSeek 系，循环性「不能完全排除」（以手工 core、跨家族 judge 审计、外部子集三重对冲）；Lethe 聚合分并不显著优于 LangGraph 裸存储，参考实现自我定位为「anchor 而非竞争系统」；Zep 因 docker 约束未实测；63–68% 只是该套件上的经验饱和带、非上限（外部子集已跌至 28–51%）。

#### 2.7 代码是否公开且可打开

**是，且已实开核验**【verified live，2026-10-29】：https://github.com/deeplethe/lethe —— Public，86 commits（最新 2026-06-16），含 `lethe/`（实现）、`bench/`、`paper/`、`recipes/`、`tests/`，MIT 许可，PyPI 名 `pylethe`，README 自述 v1.0.0-alpha、pytest 14 passed，MCP 暴露 11 个工具含 signed-purge receipts。arXiv 版附录 V 亦给出该仓库为正式发布地址（「released under MIT」）。

---

### 3. Beyond Downstream QA / FEB-30（Toma et al., arXiv:2610.22209）核查

【原文】读取层级：**全文正文**（§1–8、指标公式、结果表、局限），附录未逐项读。AWS / Cluj 团队，2026-10-15 提交。

- **主张**：现有 agent 记忆基准全用下游 QA（extrinsic）当记忆质量的代理，把记忆、检索、生成三者的功过混在一起：假阳性（记忆是错的，但强 LLM 靠推理补偿答对）与假阴性（记忆是对的，检索没取到却怪记忆）。提出 **intrinsic evaluation**：每步摄入后**直接检查记忆库内容**与已知 ground truth 的一致性（类比软件工程的单元测试 vs 端到端测试）。
- **FEB-30 构造（paradigm inversion）**：先定义 30 条事实演化 trajectory（每条含 states、原子 claims、每步新信息的 docClaims、期望动作 create/update/reinforce/conflict/supersede），再由 trajectory **生成**文档——由此得到确定性 ground truth、免人工标注。6 诊断类（C1 LLM 补偿 / C2 检索瓶颈 / C3 分类 / C4 合成 / C5 时间正确性 / C6 多作者）× 3 难度各 10 条；共 147 文档（127 状态文档 + 20 噪声）；生成用 6 模型轮转（单个 ≤30%）+ 3 judge 一致验证（蕴含∧无幻觉），L3 把 claims 注入真实无关文档（约 90% 真实文本）；人工复核修正 11% 内容、28% 去脚手架。
- **指标**：内在 = claim coverage（embedding 相似 >0.8 后再经 LLM entail 判定）、fact accuracy、update detection recall、false update rate、fact count、confidence MAE；外在 = 诊断 QA（YES/PARTIAL/NO）；分歧 = MAD（两者绝对差均值）与 DivRate（差 >0.15 的比例）。
- **被测系统**（代表性 extract-embed-classify-synthesize 管线，作者明说系统本身不是贡献）：LLM 分类动作、temporal guard（过期 observation 的 update/supersede 自动降级为 reinforce）、confidence 步进、**每个 fact 保存全历史并链接其 supporting observations**（一种 observation→fact 的来源链接）、BM25(0.3)+向量(0.7) 混合检索。
- **主要结果**（11 次运行）：内在 60.6% vs 外在 48.8%，**MAD 0.168±0.025、DivRate 43.6%**（摘要约数 16 点 / 44%）；pass/fail 结论不一致 21±4%、κ=0.118、r=0.423（r²≈0.18，两套评测近乎独立信号）；**动作分类准确率仅 20.1%**（L1 33.8%→L3 10.1%），false update rate 22.2%，update detection recall 73.6%；C6 多作者内在分最低 41.5%、claim miss 87.5%。四种配置（系统×语料×骨干）下 MAD 0.158–0.181，结论稳健。
- **作者自述局限**：合成文档效度待真实多作者语料验证；30 条只够诊断、不足细粒度统计；评分依赖 embedding 阈值 + LLM judge 有测量噪声；不测 episodic/procedural 记忆与单轮回忆。
- **代码**：论文脚注称发布于 https://github.com/amazon-science/feb-30-benchmark ——**2026-10-29 研究窗口实开返回 GitHub 404**【verified live】。结论：论文声称释放，**截至核查日实际打不开**（未登录状态下不存在/不可见），按未获取处理。
- **与 ProvPurge 的关系**【推断】：它不做删除方法、不做回执，对 ProvPurge 的直接占用小；但它给 ProvPurge 的评测叙事埋了两个雷：①「只看下游 QA 会同时产生假阳/假阴」与 MemCut 的 native/residual 之分互相印证，ProvPurge 若只报 QA 类指标会被双重引用夹击；②它示范了「内在检查记忆库状态」已是成文范式，ProvPurge 的自验若只是「查库里还有没有该事实」，新意叙述须与 intrinsic evaluation 划界（ProvPurge 的探针应是*行为/推导*探针，不是状态检查）。

### 4. TRACE（Xiong et al., arXiv:2609.33517）核查

【原文】读取层级：**全文正文**（§1–7：形式化、方法四阶段、实验、消融、局限全读），附录未逐项读；代码仓库已实开。上海交大 / MemTensor / UC Berkeley，2026-09-27 提交。

- **问题（先划清：TRACE 不做删除）**：**temporal memory admission**——一个 agent 离开团队一段时间后返回，哪些保留记忆还可被*采取行动*。记忆可以检索得到、与任务相关、忠于来源，却已不可采（inadmissible），如出发前存的行程里酒店已被团队换掉。它实现的是 **functional forgetting**：被拒状态可以留在 audit store 里，只是不准作为新工作负载的前提——与 ProvPurge 的硬删是**相反的处置哲学**，引用时勿混。
- **方法**（training-free 层，夹在记忆后端与 agent 之间）：①**捕获 return-cycle checkpoint**：离队时记录该 agent 可见状态 + **source receipts** + epoch，私有 episodic bank 被隔离、每条经验**绑定其源 work-state item**；返回时 $C = D_a \oplus \Delta$（离队检查点⊕缺席期增量）。②**失效解析**：显式失效（epoch/有效期/supersession/revocation 链接——确定性规则）+ 隐式失效（高召回 scan 提出候选依赖对 → grounded verifier 逐对用 LLM 判定，只依据 checkpoint 证据、不用基准标注）。③**四门验证** $E=\{m: A(m)\wedge T(m)\wedge U(m)\wedge P(m)\}$：authorization / temporal / applicability / **provenance（P 门要求 referenced source state 有 bound receipts；只记录来源派生，不担保来源为真）**，理由编码形成审计轨迹。④**义务覆盖选择**：在计数与字符双预算下贪心近似集合覆盖（覆盖率 ≥γ、关键依赖 ≥ρ），**覆盖不了就 reset/block——fail closed**。⑤**签名 Return View**：序列化内容含 return-cycle id、principal、role、task、epoch、expiry、**item digests**、coverage report；暴露前重算 digest 并复查全部谓词；私有经验仅当其绑定的 work-state item 仍在签名视图内才被重新接纳。
- **数据结构**：work-state item 携带 content、**source receipt**、epoch 与 validity interval、principal/role scope、task purpose、dependency links、supersession status。
- **评测**：统一五 agent return 协议（1 个返回 + 4 个活跃），三个 actor 模型（Qwen3.5-122B-A10B、Gemini-3-flash、DeepSeek-v4-flash）；Memora（显式替换）、**STALE Type II**（隐式失效）、ManBench-Return（由 ManBench 衍生、传播冲突）。指标：Overall（各基准原生口径）、VIA（有效信息可用率）、IIR（无效信息拒绝率）、WSAR=1−IIR；STALE 上 LS 要求同一 episode 三探针全过。
- **主要结果**：ManBench-Return 上 TRACE 是唯一双高：VIA **92.6–98.3%**、IIR **98.4–99.5%**（Restore 保留一切、Reset 拒绝一切，两者各有一项在对侧条件下触 0）；STALE Type II 上 Overall 超最强对比策略 **+22.3（Qwen）/+18.5（Gemini）/+27.5（DeepSeek）点**，Qwen 的 LS 由 18.0%→39.0%，代价约 **2.3× token**（499,659/episode）；写时整合管线 CUPMem 在 Qwen 子集上更准（Overall 80.56% vs TRACE 58.89%）但烧 TRACE 的 3.99× token；**Memora 上 TRACE 无一致增益**（FAMA 40.14/42.80/42.58 vs MemTX 42.14/44.12/44.72），作者明说显式替换场景下该层不必要。
- **消融**（STALE、Qwen）：去掉隐式失效模块 Overall **−25.83 点**、IIR −31.00 点（最大单项）；义务覆盖换成 relevance 排序 → Overall 38.67%；去掉 complete-view gate → 47.33%；去掉 source-bound 私有重接纳 → 42.00%（全量 52.33%）。4–32 个活跃 agent 下波动 ≤0.36 点；四后端（ScopedMem/Mem0/Memobase/A-MEM）可移植（IIR +0.39~6.42 点、VIA −0.05~−3.03 点）。
- **作者自述局限**：假设 authenticated return、离队 checkpoint 与 source receipts 被保留；隐式失效依赖候选覆盖与 verifier 校准（漏检放行陈旧、过宽误杀有用）；缺席越长性能越差（STALE Early 73.33%→Late 43.33%，token/episode 316,750→664,860）；ManBench-Return 是衍生集；token 分析只有总量、无延迟结论。
- **代码**：**公开且实开核验**【verified live，2026-10-29】：https://github.com/xiong-wenjun/TRACE —— Public，含 `src/trace`、`data`、`results`、`tests` 与复现文档，MIT；但注意只有 2 个 commits（最新 2026-10-01），成熟度有限。
- **与 ProvPurge 的关系**【推断】：TRACE 占了三样 ProvPurge 叙事周边的东西——①「source receipt」一词已被它用作*来源*凭证（与 LETHE 的*删除*回执是同名异物，ProvPurge 写回执时必须先做术语切割）；②「签名 + 覆盖报告 + fail-closed 才放行」这套**事务式门控**形态已被它用于准入（签名对象是 Return View 而非删除事件），ProvPurge 的「硬删事务」若只讲 fail-closed 与签名，形态上不新；③它已在 **STALE Type II** 上立了强基线且证明「返回/使用时再验」可行，这会反过来质疑 ProvPurge 为何一定要删时验、以及 STALE 在 ProvPurge 三基准中的定位。

---

## 对 ProvPurge 的判决

> 本节全部为【推断】（基于上文【原文】事实的分析），占用比例为本报告的估计，不是任何论文的数字。

### A. 逐条贡献对照：被占多少、还剩多少

| ProvPurge 原贡献 | MemCut 占了什么【原文】 | LETHE（+TRACE/FEB-30）占了什么【原文】 | 占用估计 | 还剩什么 |
|---|---|---|---|---|
| (1) 三层冗余副本的句级来源链 | **写入时记录每个对象的 source turns + 按 closure 删除**（§4.2）已是同一机制，只是粒度在 turn 而非句、且实现于评测 harness 而非被提出的系统；其五系统恰好含「原文+摘要」「重写叙事」「整合摘要」三类衍生层 | LETHE 的 event log + supersession 边表是另一套溯源/审计结构；purge 按 rowid **同时删文本/向量/FTS 三表**——「多层副本一并清除」在其单库内已是既成事实。TRACE 的 item 级 source receipt + dependency links、FEB-30 系统的 fact↔supporting observations 链接，进一步把「来源链接」变成 crowded 的通用件 | **约 60–70%** | ①**句级**粒度（比 turn 更细，使摘要句可单独切除而不整对象陪葬——MemCut 自认 closure 的附带损伤未测，这正是句级能打的点）；②来源链作为**写时一等数据结构服务于删除事务本身**（预计算闭包/最小割），而非评测锚定；③**跨异构层**（独立向量库与摘要层不在一个 SQLite 里）时来源链的一致性维护 |
| (2) 带在线自验的硬删事务 | 探针的**测量学**被它定义完：native vs residual、同一 reader、closure 后测量、hidden-risk 下界——但那是**离线评测**，其基线删完不复检、不迭代 | LETHE 占了「硬删」本身（purge 三表全删、不可逆 Prop.3）与事务的密码学收尾（回执）；TRACE 占了「验证不过就不放行（fail closed）+ 签名才生效」的事务形态（用于准入）。但**没有任何一篇把行为探针放进删除回路**：LETHE 的验是验日志/签名，TRACE 的验是验准入资格，都不是「删完立刻用攻击问题试着把事实问/推导出来，恢复得了就继续删」 | **约 40–50%**（硬删与事务形态被占，探针测量学被占；**回路本身未被占**） | **probe-gated commit**：探针套件预承诺 → 删除 → 在线跑 native+residual 式探针 → 任一恢复成功则自动扩候选再删 → 全不过才提交并签回执。这是四篇里唯一没人做的闭环，也是 ProvPurge 还能立住的那一条（见 D 节） |
| (3) 三协议联合评测（BEMULA + LongMemEval-S KU + STALE） | MemCut 本身就是一套更贴删除的协议，且其 Table 1 已把 LongMemEval-S、ForgetEval 等摆在同一比较面 | LETHE 已做 ForgetEval + Memora + FactConsolidation + LongMemEval-S 四面交叉评测并提出「轴翻转」；TRACE 已做 Memora + **STALE Type II** + ManBench-Return 三面联合；FEB-30 已做内在+外在双轨 | **约 70–80%**（作为「贡献」基本不成立） | 只剩**执行层面的组合价值**：同一删除事务在三套协议下用**同一残余口径**报告。作为贡献应降级为实验设计；且名单必须更新——不加 MemCut，这条会被审稿人当场击毙（见 C 节） |
| 附：删除回执（原计划卖点之一） | 未涉及回执 | **LETHE 几乎全额占用**：Ed25519 签名、Merkle 锚定、四项内容、双路验证、GDPR 叙事、MCP 工具、现成代码（详见 B 节）。TRACE 另有 source receipt 与签名 Return View 两个近亲 | **约 85–90%** | 只剩「回执内容绑定**验证结果**」一条缝（B 节给方案）；「做一个删除回执」本身已不可声称 |

### B. 特别回答：LETHE 的 signed purge receipts 与 ProvPurge 的「删除回执 + 自验」是不是同一件事？

**结论：回执部分是同一件事，而且 LETHE 的版本更完整；自验部分不是同一件事——这正是 ProvPurge 唯一的缝。**

- **同**：两者都是「删除动作完成后，向第三方出示一份可独立验证的凭证」。LETHE 的凭证在密码学维度已经封顶：签名（Ed25519）+ 防篡改锚（event log 的 Merkle root）+ 内容指认（row ids + 删除前文本 SHA-256 + purge 事件的 log IDs）+ 两种验证路径（带 DB 重算 root、无 DB 仅验签）。ProvPurge 若按原计划做「删除了 X、时间 T、签名 S」式回执，是 LETHE 的**弱化复刻**，且对方有可运行代码与 GDPR Article 17 的现成叙事。**「首个/可审计的删除回执」类表述必须放弃。**
- **不同**：LETHE 回执证明的是**事件真实性**（purge 确实执行、删的是这些行、日志没被改），**不证明删除的有效性**（删完之后事实还能不能被问出或推导出来——按 MemCut 的数据，这恰恰是经常还能）。LETHE 的自家 Roadmap 也承认回执尚无任何评测（「Receipt-verification benchmark family」列为待做）。ProvPurge 的「自验」若成立，验的是**行为有效性**（攻击探针恢复不了）。两者一个验「做没做」，一个验「成没成」——**不是同一件事，但如果 ProvPurge 把回执和自验分开写，两者会分别被 LETHE 与 MemCut 各吃掉一半**。

**若 ProvPurge 还想保留回执，必须做出的实质区别（缺一不可）**：

1. **回执内容从「事件清单」升级为「闭包 + 验证」双清单**：除删除对象清单（含三层各自的对象 ID 与删除前内容哈希、来源闭包的句级清单）外，必须签入——候选池快照的承诺哈希（删前生成，防事后挑池）、**探针套件的预承诺哈希**（探针在删除前登记，防「删完挑软探针」）、每轮探针的结果摘要（native 探针、residual 式直喂探针、推导探针各自的恢复/未恢复）、扩候选轮次记录、以及兄弟事实保全探针（must-not-break）的结果。即回执要能让第三方核验的不只是「删了这些行」，还有「**删后按预登记的攻击面测过、测的是这些题、结果是恢复不了、且没误伤邻居**」。
2. **自验与回执强联动：回执是探针的函数，不是删除的函数**。具体语义三选一并写死：探针全过才签「verified」回执；探针未过则**不签发或只签发 failed/unverified 状态的回执**且事务不提交（或回滚/进入下一扩删轮）；回执中含探针成绩单的哈希，使「没测就签」在密码学上不可伪装。换句话说，把 LETHE 回执的 Merkle 锚从「事件日志」扩展到锚定「**探针执行记录**」——这是 LETHE 现有回执结构里**没有**的东西，也是唯一能让审稿人承认区别的改法。
3. **覆盖范围声明必须比 LETHE 大一圈且如实**：LETHE 回执只锚其单库 rowid；ProvPurge 的回执若声称覆盖「事实层+向量层+摘要句层」的跨层闭包，须把每层被删对象的哈希分别签入（向量层签 embedding 条目 ID+索引版本号，摘要层签被切除的句 ID+其来源句链）。做不到跨层就不要声称——否则与 LETHE 同范围而更弱。

做不到 1+2 的话，建议**直接删掉回执卖点**，把 LETHE 回执当现成组件引用（其代码 MIT 且可打开），只把新意压在自验回路上。

### C. MemCut 能否直接当 ProvPurge 的基线或组件？候选扩展可照抄什么？

**能，且应该——分三层用：**

1. **作基准（最优先）**：MemCut 是目前与 ProvPurge 问题定义（用户请求遗忘 + 可推导残余）最贴合的基准，比原计划三基准都更对口。建议**把 MemCut 升为第一主基准**，主指标直接用它的 native / residual / hidden-risk 三件套。注意其数据/代码截至核查日未获取（见 1.9），排期上要预留「等释放或按其构造法自建子集」的风险。【推断】
2. **作基线（现成配方，无需等代码）**：S0/S1/S2 的定义完整到可直接复刻——S1 = 以删除请求单次查原生检索器、池 **20 turns**、删 top-10；S2 = 同池、由 write model 按存储文本重排删 top-10；统一以 turn 计池/预算、closure 删除、每事实 fresh copy。 ProvPurge 的方法必须至少在这三条上同表对比；打不过 S2 的「扩池版」（见下）就没有发表意义。
3. **作组件（协议层）**：其 §4.2 的接入协议（写入时记录 source turns、list/search/delete/snapshot 四能力、以 turn 锚定一切操作）可以直接借作 ProvPurge 的实验 harness 规范——但引用时必须承认 ProvPurge 的句级来源是把这一 turn 级锚定细化，而非首创锚定本身。

**候选覆盖结论中可直接照抄的参数与做法**【原文数字】：

- **预算别加、池子要加宽**：主设置从池 20 起步做 **20→60 的池宽扫描**（预算固定 10）；文献值：池 20→60 使 residual 降 12.5–23.9 点，而预算 10→20 只降 ≤6.5 点且 off-target 从 5.54 涨到 15.49/事实。ProvPurge 的消融表应原样包含这两条曲线，否则会被要求补。
- **报道 $k$（实际删掉的证据/目标相关 turns 数）**：用 $\mathrm{Res}\approx 13(8.2-k)$（$R^2=0.95$）作健全性标尺——若 ProvPurge 报告的 residual 降幅远超其 $k$ 能解释的范围，审稿人会按此式质疑；反之可用它证明「我们删得少但断的是 cut」（minimal cut 均 3.04、max 5，远小于 8.1 条 evidence 全删——**以 cut 为目标的选择器在原理上能用 ≤5 次删除做到全删证据做不到的事，前提是 reach 完整**，这是 ProvPurge 相对 S2 最可辩护的机制优势）。
- **实体扩展（entity expansion）作为候选生成的第二通道**：附录 D 案例中每个 evidence turn 都在请求实体的 1-hop 内，作者明说此法未测。ProvPurge 可直接把「来源链实体图上的 1–2 hop 扩展」做成候选扩展模块，并在 MemCut 的 relation chain 子集（residual 71.8%、池 60 也只够着 4.4/8.3 条 evidence 的最难类型）上验证——这是把 MemCut 的诊断结论变成方法增益的最短路径。
- **池内选择用「打断 sufficient sets」的组合目标而非相似度/LLM 印象分**：池 60 时 S2 已打断 92–93% 的 sets 仍剩 31–40% residual，说明优化目标必须是 complete cut（hitting set）而非「尽量多删相关项」；LLM 判断的增益在池 20 只有 5–15 点、池 60 才 20–32 点——**先扩覆盖、后上判断**的工程顺序可直接照抄。
- **评测卫生**：探针/恢复题用「可拒答 + 双选项顺序都对才算恢复 + 错误选项只差一细节」的构造防猜测；附带损伤用 off-target fraction 报，且须与 residual 同表（作者亦自认还缺下游效用维度，ProvPurge 应补 sibling-preservation 探针，正好与 B 节回执的保全项呼应）。

### D. 需要改口、加强或放弃的说法清单 + 必引名单更新

**放弃（再说会被原文直接证伪）**：

1. 「首个删除回执 / 可审计删除证明」——LETHE 已有 Ed25519+Merkle 的完整实现与代码（占用见 A/B 节）。
2. 「首次区分『表面删掉』与『实际还能恢复』」——MemCut 的 native/residual/hidden-risk 已成体系，且 FEB-30 的 intrinsic/extrinsic 假阳假阴是平行证据。
3. 「首次用来源链做级联删除」——MemCut 的 source-turn 记录 + closure 删除、LETHE 的 purge 三表同删，均在先。只能说「句级」与「跨异构层」。
4. 「三协议联合评测是贡献」——LETHE（四面交叉）、TRACE（三面联合，含 STALE）已做；降级为实验设置。
5. 「确定性清除即可靠」这类无条件表述——LETHE：确定性在 identifier_obfuscation ≤5%、cross_lingual 0%；MemCut：零 LLM 的 S1 后 residual 48–86%。确定性只能修饰**执行**（给定候选集，删什么、怎么删是确定性规则），不能修饰**发现**。

**改口（收窄到可辩护版本）**：

6. 「句级来源链」→「把 turn 级来源锚定（MemCut 协议）细化到**句级**，使摘要层可按句切除、避免整对象 closure 的附带删除，并以来源图预计算删除闭包与最小割」。新意点从「有来源」移到「粒度 + 用途」。
7. 「删除回执」→「**验证绑定回执**（verification-bound receipt）」：按 B 节 1+2 条实现后再用此名；实现不了就删此卖点、改引 LETHE 回执为组件。
8. 「攻击探针自验」→ 明确写成「将 MemCut 的 residual recovery 测量**在线回路化**：探针集删前预承诺、删后即测、恢复成功自动触发扩候选再删、全过才提交」。探针本身的发明权归 MemCut，ProvPurge 只主张回路与事务语义。
9. 基准叙事：主基准名单改为 **MemCut（主）+ STALE + LongMemEval-S 知识更新子集**，BEMULA 保留与否须另行核查其口径（本轮未核查 BEMULA，见 Could not verify）；且 STALE 的引用方式要处理 TRACE 的先行使用（TRACE 在 STALE Type II 上的数字是 ProvPurge 必须同表交代的对照）。

**加强（不做会被审稿人补刀）**：

10. 主指标必须以 **residual recovery** 为首，native/QA 类指标只能作陪衬；同时报 hidden-risk 与 off-target fraction，并补兄弟事实保全/下游效用一项（MemCut 自认缺此维度，是可占的评测空位，但只能作配菜）。
11. 消融必须含**池宽扫描（20/30/40/50/60）与预算扫描**两条曲线 + 每设置的 $k$ 值（删中目标数），与文献拟合式对照。
12. 候选生成必须含**实体 hop 扩展**通道并在 relation chain 类上单列结果；否则方法在最难类型上必然复现 S2 的 2.4/8.1 删中率。
13. 与 LETHE 的对照实验：至少把 Lethe（MIT 代码可直接跑）作为一个被测/基线系统纳入，或在 ForgetEval purge/amnesia 两族上报告 ProvPurge 的成绩——回避它比输给它更伤。

**新增必引名单（更新）**：

| 文献 | 为何必引 |
|---|---|
| MemCut, Zhang et al., arXiv:2610.21405 | 问题定义、residual 口径、来源/closure 协议、S0–S2 基线——全案的锚 |
| Control-Plane Placement (LETHE), Yang, arXiv:2606.15903 + 其仓库 deeplethe/lethe | 控制面框架、ForgetEval、签名回执的先行实现 |
| Beyond Downstream QA / FEB-30, Toma et al., arXiv:2610.22209 | intrinsic vs extrinsic、QA 假阳/假阴——评测叙事的平行证据 |
| TRACE, Xiong et al., arXiv:2609.33517 | source receipt、签名视图、fail-closed 门控、STALE Type II 强基线 |
| STALE, Chao et al., arXiv:2605.06527（据 TRACE 参考文献）【本轮未读原文，仅经 TRACE 转述其 Type II 定义】 | ProvPurge 原计划基准之一，且 TRACE 的使用方式构成对照 |
| Memora / From Recall to Forgetting, Uddin et al., arXiv:2604.20006（据 LETHE 参考文献）【未读原文】 | FAMA、过时记忆复用——与知识更新子集叙事相邻 |
| MemLeak, Wang & Zhang, arXiv:2606.29788（据 MemCut 参考文献/正文）【未读原文】 | 相关线索恢复泄露，与 residual recovery 相邻，related work 需切割 |

### E. 最小可发表内核（直说）

【推断】读完四篇后，ProvPurge 原「三条贡献」的成色是：**贡献 (3) 基本清零（降为实验设计），回执卖点基本清零（除非按 B 节改造），贡献 (1) 只剩粒度与用途的窄边，贡献 (2) 是唯一还有完整形态的一条。** 新意被压缩掉约三分之二。

**剩下的最小可发表内核（一句话）**：

> **Probe-gated 硬删事务**：以写时句级来源图给出跨三层副本的精确闭包与最小割候选，再以实体 hop 扩展补足可推导证据的覆盖；删除不以「执行完」为提交条件，而以**预承诺攻击探针集（native + residual 式直喂 + 推导型）在在线回路中全部无法恢复、且兄弟保全探针全过**为提交条件；未过则自动扩候选再删，提交时签发的回执同时锚定闭包清单与探针成绩单。在 MemCut 上以 residual recovery 为主指标，打穿 S2 的 39–76% 残余带（尤其 relation chain 的 71.8%），同时用 off-target 与保全指标证明不是靠滥删。

这个内核的每一块都有明确出处与空位：探针测量学来自 MemCut（且其作者明说 entity expansion 未测、专门 erasure 方法待人做），回执的密码学外壳可直接站在 LETHE 肩上（引而不用其名），fail-closed 事务形态与 TRACE 划界（它管准入不管删除、且是功能遗忘不硬删）。对硕士论文而言，这一条系统贡献 + 一套对齐 MemCut 口径的扎实评测，足够成立；再往上叠任何「首个回执/首个来源链/首个联合评测」的表述，都会在审稿环节被这四篇中的某篇原文当场拆掉。

---

## Could not verify

- **MemCut 代码/数据**：论文称 release harness、benchmark 将释放，但全文无仓库 URL，2026-10-29 两轮搜索未找到公开仓库——**未获取**，其实测可打开性无法确认；ProvPurge 复现 S1/S2 暂只能按论文参数自研。
- **FEB-30 代码**：论文脚注给出的 https://github.com/amazon-science/feb-30-benchmark 于 2026-10-29 实开为 GitHub **404**——声称释放与实际可达不一致，按未获取处理（不排除日后上线或权限变化）。
- **BEMULA**：ProvPurge 原计划基准之一，本轮任务未要求核查、亦未读到其原文——其与 MemCut/STALE 的口径关系**未核查**，基准名单调整（D 节第 9 条）中涉及 BEMULA 的部分属待定。
- **STALE 原文、Memora 原文、MemLeak 原文**：仅经 TRACE / LETHE / MemCut 的转述与参考文献条目获知（arXiv 号见必引表），未读原文，相关表述已在文中标注转述层级。
- **LETHE 回执的 CBOR vs JSON 不一致**：仓库版论文 §5.4 写 canonical CBOR payload，仓库 recipe 脚本 docstring 写 Ed25519-signed JSON document——已按「以代码行为为准」处理，但回执 payload 的确切序列化格式**未逐行读 `lethe/receipt.py` 源码确认**（本轮只读到 recipe 层与论文层）。
- **LETHE LongMemEval-S 两个 R@5 数字**（论文附录 Q 的 93.8% session 粒度 vs README 的 97.4% sub-document 口径）系不同口径并存，非矛盾，但引用时必须注明口径。

## Sources

1. **MemCut** — Zhang, Cao, Zheng, Xiong, *MemCut: Every Secret Leaves a Trail in the Long-Term Memory of LLM Agents*, arXiv:2610.21405v1（2026-10-15）。全文 HTML：https://arxiv.org/html/2610.21405v1 ；摘要页：https://arxiv.org/abs/2610.21405v1 。2026-10-29 读，**index 全文**（正文+附录 A–E 全读）。笔记：notes/source-memcut.md
2. **LETHE / Control-Plane Placement** — Dongxu Yang, *Control-Plane Placement Shapes Forgetting: An Architectural Study of Agent Memory Across Thirteen System Configurations*, arXiv:2606.15903。全文 HTML：https://arxiv.org/html/2606.15903 。2026-10-29 读，**index 全文**（正文+附录 A–V 全读）。
3. **LETHE 仓库版论文（回执 §5.4 出处）** — Yang, *ForgetEval: Benchmarking the Forgetting Axis of Agent Memory Systems*（2026-05-13，仓库内 PDF）：https://github.com/deeplethe/lethe/raw/refs/heads/main/paper/paper.pdf 。2026-10-29 读，**index 全文关键节**（§5.4/5.5、Prop.3、实验节）。
4. **LETHE 代码仓库** — https://github.com/deeplethe/lethe （主页、recipes 目录、https://github.com/deeplethe/lethe/blob/main/recipes/02_gdpr_purge_receipt.py ）。2026-10-29 读，**verified live**（研究窗口实开）。
5. **FEB-30** — Toma, de Kernier, Ply, Matei, *Beyond Downstream QA: Rethinking How We Evaluate Agent Memory*, arXiv:2610.22209v1（2026-10-15）。全文 HTML：https://arxiv.org/html/2610.22209v1 ；摘要页：https://arxiv.org/abs/2610.22209v1 。2026-10-29 读，**index 全文正文**（附录未逐项）。其声称的代码地址 https://github.com/amazon-science/feb-30-benchmark 于同日 **verified live 实开为 404**。
6. **TRACE** — Xiong et al., *TRACE: Governing Memory Validity in Evolving Multi-Agent Systems*, arXiv:2609.33517v1（2026-09-27）。全文 HTML：https://arxiv.org/html/2609.33517v1 ；摘要页：https://arxiv.org/abs/2609.33517v1 。2026-10-29 读，**index 全文正文**（附录未逐项）。代码仓库 https://github.com/xiong-wenjun/TRACE 同日 **verified live** 实开（Public，2 commits，最新 2026-10-01）。
7. 综合笔记：notes/source-lethe.md、notes/source-feb30-trace.md（均在研究目录内）。
