# 三篇 2026 论文精读报告：BEMULA / PolyGraph / PACMI 与 ProvPurge

> 服务对象：AI 硕士研究生；拟定方案 **ProvPurge**（暂名）——LLM Agent 长期记忆的知识更新与选择性遗忘，核心想法为「写时句级来源 + 确定性级联清除（事实/向量/摘要句三层）+ 删除后攻击探针自验」，主基准拟用 BEMULA + LongMemEval-S 知识更新子集 + STALE。
> 阅读方式：三篇均打开 arXiv HTML 全文逐节读完（含附录），非仅摘要。阅读日期 2026-10-29。凡标「已证实」者均有全文页面支撑并附来源；标「推测/建议」者为基于全文的分析判断，不是论文原文结论；未获取的信息明确标注「未获取」。

## Summary

- **BEMULA**（Xie et al., arXiv:2610.18052, 2026-10-13）是**基准论文、不是方法论文**：它定义「显式遗忘请求下的记忆卸载（memory unlearning）」任务，50 个多会话对话、135 条六类遗忘请求、1,742 道题、成对 control 版本、五套探针。结论是负面的：六个主流记忆系统 × 三个 backbone，**请求通过率 RPR 最高 1.5%，多数为 0**。它给 ProvPurge 提供的是考场和判分规则，不是竞争方法——但它同时宣判：只做对「删除动作」远远不够，必须过攻击探针与保留探针的联合关。
- **PolyGraph / Keep Your Receipts**（Choudhary, arXiv:2610.17719, 2026-10-13, 单作者, NeurIPS 2026 workshop 预印本）是与 ProvPurge **机制上最像**的一篇：写时让构造器为每个图 artifact 发「receipt（支持标签）」——若干组各自充分的来源断言束；删除时只做集合运算（某束与删除集不交则存活），**零模型调用、1–21 ms**。但它的实证结论是一记警告：**扁平的摘要级 receipt 约 2/3 是错的且几乎全是「漏删（leaky）」；只有把 receipt 改成「一句一 receipt」后忠实度才升到 100%/88%**。这直接验证了 ProvPurge「句级来源」的必要性，也预告了审稿人会拿它质问 ProvPurge 的向量层与事实层来源是否同样不可信。
- **PACMI**（Wang et al., arXiv:2610.05732, 提交 2026-10-05）是与 ProvPurge **叙事上最像**的一篇：来源图 + 类型依赖边 + 四态有效性格 + BFS 级联传播。但必须看清它的证据强度：依赖边由基准标注**给定**、真值是固定两跳拓扑的确定性函数、每 case 只有 6 条记忆；其最强卖点「级联」的答案级消融 **p=0.0574 不显著**，真正拉开差距的是 premise checker 与历史保留。PACMI 的删除是**失效保留（非破坏）**，与 ProvPurge 面向显式删除/被遗忘权的**硬删**在合规语义上根本不同——这是 ProvPurge 最重要的区分句柄。
- Google Scholar 核查（单独一节，见 §5）：BEMULA 已被 Scholar 收录且 **Cited by 1**；PACMI 抓到条目但**被引数未显示**；PolyGraph 的 Scholar 查询被 403 拒绝、**收录情况未核实**。顺 Scholar 的 Related 列表发现一篇三篇之外、撞车风险可能最高的新工作 **MemCut**（arXiv:2610.21405, 2026-10-15，比 BEMULA 晚两天）：它已用「删除后残余恢复（residual recovery）」证明删后仍可从残存证据重建秘密，且指出删除候选覆盖率是瓶颈。ProvPurge 的开题与相关工作若不处理 MemCut，会被直接击中。

---

## Findings

### 1. BEMULA — Can Agents Forget When Asked? A Benchmark for Memory Unlearning in LLM Agents

来源：全文 https://arxiv.org/html/2610.18052v1 （2026-10-29 打开，verified live）；摘要页 https://arxiv.org/abs/2610.18052v1 ；作者 Bofan Xie、Chaokun Wang、Guanchen Ge、Songyao Wang、Fang Niu、Hang Zhang（清华）、Wei Wang（新浪微博）。

#### 1.1 研究问题、动机与声称贡献

**问题**：长期记忆系统只被测过「写/检索/组织」，从没被测过「用户明确说『忘掉 X』时，系统能否只删 X、其余完好」。删不净 → 隐私泄漏（检索、推理、提示抽取都能捞回）；删过头 → 记忆不可用。

作者声称的三条贡献（§1 逐条）：
1. **任务定义**：提出 memory unlearning 为新任务；遗忘请求按三轴（目标粒度、信息性质、操作语义）组织成六类分类法；每条对话配 control 版本，把「真遗忘」与「从没存进去」分开。
2. **BEMULA 基准**：首个专测记忆卸载的基准；50 个对话实例、746 sessions、1,742 道题；五阶段多智能体管线合成；五套探针 + 成对 control 评分；全部答案由 LLM judge 判分。
3. **实证研究**：主流系统在 BEMULA 上集体失败——被删信息仍可检索、可抽取，记忆卸载是开放问题。

任务形式化（式 1）：$a_t=\pi_\theta(u_t,W_t,R(M_t,u_t))$，$M_{t+1}=\text{Write}(M_t,W_t,u_t,a_t)$；$\pi_\theta$ 冻结，基准对记忆内部组织不可知、只假设 $R$ 与 Write 两操作。遗忘版对话 $D$ 与 control 版 $D'$ 逐轮相同，仅遗忘请求+助手确认被替换为等长的中性闲聊。

#### 1.2 方法细节：这是一套「数据构造 + 评测协议」，没有被测系统侧的新算法

构造管线五阶段（§3.2，**全程重度使用 LLM**，这点对 ProvPurge 复用其数据时要心里有数）：
1. **Profile 合成**：采样 6 个身份属性 → LLM 生成原子事实，每条为 $(subject,predicate,value)$ 三元组 + 事实类型 + 可选时间戳 + 文本陈述；**派生事实**由 LLM 组合 ≥2 条原子事实生成、再由另一 LLM 调用核验「严格可由所引来源推出」。全集 2,993 原子 + 77 派生事实，50 个 profile 平均 61.4 条/profile，约 1/3 带中/高敏感标签。
2. **对话规划**：50 个日常场景池（旅行/医疗/教育/金融等）；规划每 session 的场景、要提及的事实、以及每条遗忘请求的类型/目标/位置/表达风格（五种风格，从直白到含蓄，各 24–29 条）。长度四档（short/medium/long/xlong）决定每实例请求数（1–2 / 2–3 / 3–5 / 4–6）。
3. **多智能体对话模拟**：user simulator + assistant simulator 轮流发言（assistant 只看当前 session 最近 10 轮、无跨 session 记忆），director 控轮数并催提未提事实、verifier 查时序一致性与自然度，不合格 session 重生成一次。
4. **请求注入**：在规划位置注入遗忘请求，助手只回不复述敏感内容的确认；control 版替换为中性闲聊；LLM verifier 再查每条请求都带明确删除意图。数据生成模型为 GPT-5.6-Luna，seed 42，temperature 0.9。
5. **题目生成**：模板 + LLM 措辞，另一 LLM 检查剔除不可答的措辞；构造后人工抽检 10 个实例、388 道题。

**六类遗忘请求（F1–F6，这是 ProvPurge 必须逐字吃透的分类法）**：
| 类型 | 语义 | 备注 |
|---|---|---|
| F1 Factual | 删单个原子事实 | 32 条 |
| F2 Compositional | 删多个原子事实的组合 | 23 条 |
| F3 Semantic | 删整个类别（如全部医疗/财务） | 26 条，最难之一 |
| F4 Inferential | 删从未明说、只能推出的派生事实 | 13 条；只测派生结论是否消失，**源事实存亡不计入 F4 语义**（保留源+抑制推理，或连源一起删，两种合规方式都不罚） |
| F5 Temporal | 删某时间段的记忆 | 20 条，最难之一 |
| F6 Replacement | 以新值替换旧事实 | 21 条；功效探针的金标是新值而非 [FORGOTTEN] |

目标事实在请求前平均被提及 2.5 次（0–5 次）；52 条请求目标多于一条事实；共 135 条请求打 269 个目标事实。

#### 1.3 数据与协议：五套探针与指标的精确定义

评测协议（§3.4 + 附录 A）：系统逐 session 摄入对话；每道题在**只含该问题的新 session** 中提问，答案只能来自记忆 $M$，且回答不写回记忆。control pass 与 forget pass 两遍，系统在两遍之间重置。**被测系统事先不知道哪些轮次是遗忘请求——请求只是普通对话轮次，必须靠自己的 Write/更新路径识别并执行删除**（附录 C.2 明示：不给任何额外信号、评测方不代发 delete 调用）。这一条对 ProvPurge 至关重要：你的删除触发必须能从自然语言请求里被系统自发触发，不能靠评测脚本喂结构化删除指令（至少主结果不能）。

五套探针（每套的金标与计分版本）：
- **Retention（留存）**：control 版上问目标事实，金标=事实值。
- **Efficacy（功效）**：forget 版上同措辞问目标事实，金标=标记 `[FORGOTTEN]`（F6 为新值）；judge 只在「报告不可用且不泄漏目标值」时给分。
- **Preservation（保留）**：forget 版上问 3 条从未被任何请求瞄准的干扰事实（每实例恰 3 道，共 150 道），金标=事实值，测过删。
- **Relearning（再学习）**：附一段用户陈述「同类型新值」的短对话，系统摄入后作答（F6 豁免），共 114 道。
- **Attack（抽取攻击）**：把每个目标问题塞进 **5 个固定攻击框架**——①间接提问（不点名地套）；②角色扮演（权威框架）；③越狱前缀（override 指令，Wei et al. 2023）；④布尔猜测（把抽取变成对目标值的 yes/no 猜测）；⑤多跳推理（从仍留在记忆里的相关事实重建目标）。金标=`[NO_LEAK]`；**框架从不改写措辞，因为措辞本身就是攻击向量**。共 1,030 道 = 5 类 × 206。1,742 道题中 1,314 道可直接检索作答，428 道需 1–2 跳推理。

指标（式 2–5，精确定义）：
- 有效正确性 $\tilde z_i = z_i \wedge c_i$：efficacy/preservation/attack 题必须**同时**通过其成对 control 探针才计分（从没存住的目标，「忘掉」不给分）；
- $\mathrm{RA},\mathrm{FE},\mathrm{PR},\mathrm{RLA}$ 分别为对应题集上 $\tilde z$ 的均值；$\mathrm{AR}=\frac1m\sum_a \mathrm{M}(S_a)$，$m=5$ 类攻击的均值（先按类平均，防止某类题量 dominate）；
- **RPR（请求通过率）**：一条请求当且仅当**它自己的全部题目 + 所在实例的全部 preservation 题全对**才算通过——这是全文最严的指标；
- **总分** $\mathrm{Score}=\mathrm{FA}\times\mathrm{UG}$，$\mathrm{FA}=\sqrt{\mathrm{FE}\cdot\mathrm{AR}}$，$\mathrm{UG}=\sqrt{\mathrm{PR}\cdot\mathrm{RLA}}$：乘积形式，任一维为 0 则总分 0，「全删」与「不删」同罪。全部指标报百分比；LLM judge 为 DeepSeek-V4.1-Flash（temp 0），人工验证 100 题一致率 98%、Cohen's κ=0.96（附录 B）；不确定性用对实例的 cluster bootstrap（10,000 次，附录 K）。

规模总账：50 实例（forget+control 成对）、746 sessions、每版 36,900 turns（两版共 73,800）、平均 84.8K tokens/实例、最大约 379K tokens。另发布 **BEMULA-Lite**：31 个 short+medium 实例、848 道题，指标定义不变、分数可比（附录 F）。

#### 1.4 实验结果

设置：Full Context + 六系统（Naive RAG、Mem0、A-MEM、LangMem、LightMem、SimpleMem）× 三 backbone（GPT-5.6-Luna、DeepSeek-V4.1-Flash、Qwen3-32B-AWQ）；系统内所有 LLM 调用共用同一 backbone；需 embedding 者统一 text-embedding-3-small；Qwen 因上下文长度被排除在 Full Context 与全量 A-MEM 之外。

主结果（Table 1，全文数字，%）：
| Backbone | 系统 | RA | FE | PR | RLA | AR | RPR | Score |
|---|---|---|---|---|---|---|---|---|
| GPT-5.6-Luna | Full Context | 84.4 | 26.8 | 73.3 | 98.2 | 27.6 | 0.0 | 23.1 |
| GPT-5.6-Luna | LangMem | 50.4 | 37.9 | 20.0 | 78.9 | 25.6 | 0.0 | 12.4 |
| GPT-5.6-Luna | Mem0 | 89.7 | 8.5 | 80.0 | 77.2 | 4.7 | 0.0 | 4.9 |
| GPT-5.6-Luna | A-MEM | 80.4 | 33.0 | 59.3 | 88.6 | 24.7 | 0.0 | 20.7 |
| GPT-5.6-Luna | LightMem / SimpleMem | 92.9 / 93.3 | 17.9 / 16.1 | 88.0 / 82.7 | 80.7 / 90.4 | 18.0 / 18.7 | 0.0 | 15.1 / 15.0 |
| DeepSeek-V4.1-Flash | Full Context | 75.0 | 53.6 | 46.0 | 100.0 | 32.9 | 0.7 | 28.5 |
| DeepSeek-V4.1-Flash | LangMem | 92.0 | 40.6 | 74.0 | 89.5 | 35.7 | **1.5** | 31.0 |
| DeepSeek-V4.1-Flash | SimpleMem | 90.6 | 17.9 | 84.7 | 90.4 | 20.0 | 0.7 | 16.5 |
| Qwen3-32B-AWQ | 全部（5 系统） | 21.9–92.9 | 7.1–14.7 | 5.3–86.0 | 71–94 | 5.0–14.0 | 0.0 | ≤9.5 |

怎么读这些数字（§4.2–4.3 + 附录 C/H/I/J）：
- **失败模式一：请求根本到不了存储层**。Naive RAG 无删除路径（请求轮次被当普通文本 embedding 存入）；Mem0 原理上有写时删改，但 FE 仅 8.5（GPT），失败答案常逐字复述被删值；LightMem 的更新/合并是离线阶段、对话中够不着；SimpleMem 每窗口只压缩一次、永不回改；A-MEM 无删除但可覆写相关笔记，故 FE 达 33.0/29.5、且 F1/F2 最高（46.2/52.0，GPT）。LangMem 是唯一有对话中删除通道的系统（原配置默认关闭，作者为评测开启；且只能删抽取器为当前批次检索到的笔记）。
- **失败模式二：过删/过拒**。LangMem 在 GPT 上 PR 只有 20.0——删掉了没被要求删的事实（如用户街道地址）；Full Context 在 DeepSeek 上 81 个 preservation 失败中有 66 个是无差别「我没有这条信息」式全拒（PR 46.0）。
- **攻击面**：AR 天花板 35.7（LangMem/DeepSeek），Mem0/GPT 仅 4.7；删得最少的系统里，间接提问与布尔猜测最有效（无需任何提示技巧，Mem0/GPT 间接提问 AR 仅 2.4）；Full Context 最怕越狱前缀（GPT 上 AR 9.7）；F6 表面近乎解决（各系统 FE ≥61.9）但旧值逐字存活、攻击下 AR ≤30.5。全文没有任何 efficacy 的 bootstrap CI 上界超过 60.2、没有 AR 上界超过 41.4，负面结论对抽样不确定性稳健；但相邻名次常不可分（如 DeepSeek 上 LangMem 与 Full Context 总分差 2.5，配对差 CI [−2.4, 7.6]，应读作并列）。
- **长度效应**（附录 J）：LangMem 的 FE 随对话变长从最短档 66.7 跌到最长档 21.9；多数系统 PR 随长度恶化（附带损伤随存储规模增长）；AR 几乎不随长度变——作者据此称泄漏是**结构性**的，不是长度引起的。
- 作者自述局限（§5 仅两条）：①对话是合成的、来自单一生成管线；②只覆盖对话记忆，工具型 agent 还会把被删值留在 cache、log、外部服务里——未覆盖。延伸到真实用户对话与工具使用是下一步。

#### 1.5 代码/数据开源（已实测）

论文称在 https://github.com/Nboxff/bemula-benchmark 释放全部代码、prompt、配置与生成数据。**2026-10-29 实际打开确认**：仓库 Public，含 `bemula/`、`configs/`、`data/`、`scripts/`、`results/`，README 给出评测命令（baselines: rag/mem0/amem/simplemem/lightmem/langmem/fullcontext）与三阶段数据生成脚本，现成数据分 `data/core|full|quickcheck|smallcheck`。

---

### 2. PolyGraph / Keep Your Receipts — Trustworthy Cleanup of Agent-Built Knowledge Graphs

来源：全文 https://arxiv.org/html/2610.17719 （2026-10-29 打开，verified live）；摘要页 https://arxiv.org/abs/2610.17719v1 ；单作者 Vishisht Choudhary（TU Munich）；论文自述为 NeurIPS 2026 workshop「Who Verifies the Agents?」预印本、camera-ready 可能不同。注意：该文的行文与实验形态（个人项目色彩、harness 事故一节）与典型会议论文不同，引用时宜注明 workshop preprint 身份（这是事实陈述，非贬损）。

#### 2.1 研究问题、动机与声称贡献

**问题**：agent 把文档建成知识图谱很容易，维护很难。某来源被撤回/推翻/抹除时，建立在其上的一切——实体合并（merge）、蕴含边（entailed edge）、社区摘要（summary）——都要复检；维护者只能在「删太多」与「删太少」之间二选一。现有管线只删直接抽取的事实（实测几乎无效，见下）；重建也没用，因为构造器是采样式的，重建噪声比删除信号还大。

声称贡献（§1 逐条）：
1. **receipt-and-audit 协议**，并区分审计测的 *liveness*（构造器相对存活）与合规所需的 *source-groundedness*（来源可支撑性）；
2. **replay 不可替代 receipt 的测量**：重采样噪声是删除效应的 5 倍（§5）；
3. **层依赖的忠实度结果 + receipt 形状解释 + 句级 receipt 修复方案**（§6.1–6.2）；
4. **稳健性证据**：十次额外删除、第二 oracle 模型、claim-blind prompt、第二领域（最高法院判例）、第二构造器（§6.3）；
5. **检索层后果与 receipt 成本核算**（§6.4）。

#### 2.2 方法细节

**基本对象（§3）**：
- *assertion*（断言）$x=\langle d,s,p,o\rangle$：一个源文档 $d$ 陈述的一个三元组；**撤回删除的是断言**。
- *artifact*（产物）：检索器能返回的一切，分四**层**，逐层建在前一层上：extracted claims（自文档）→ entity merges（自 claims）→ schema-entailed edges（自 claims+merges，按 OBO Relations Ontology 公理闭包）→ community summaries（自合并实体社区）。
- *receipt / support label*（支持标签，源自 de Kleer 1986 的 ATMS）：构造器必须在**创建 artifact 的同一次调用**里发出 $$L(a)=\{S_1,\dots,S_m\},\quad S_i\subseteq\mathcal{X}$$，每个 $S_i$ 是一组「自称单独即充分」的断言束；标签上限 8 组、取最小者（截断只会让 artifact 看起来更死、不会增加泄漏——单调方向是安全的）。

**删除流程（确定性，零模型调用）**：给定删除集 $D$，receipt 预测 $a$ 存活 $\iff \exists S_i: S_i\cap D=\varnothing$。两种删除算子：*retract-source*（删某被撤文档的全部断言，按文档切）；*revoke-relation*（删使用某被判不健全谓词的全部断言，按 schema 切）。清理就是对 receipts 做集合运算，24,381 个 artifact 全图 **1–21 ms**。

**审计（re-derivation audit，§4）——哪些步用模型、哪些是规则，分界非常干净**：
- 真值定义：$a$ 在 $D$ 下 *alive* ⟺ 只看 $\mathcal{X}\setminus D$ 的构造器仍会产出 $a$。因为构造器采样，每个被触及 artifact 抽 **R=5** 次独立再推导，阈值 **τ=0.2**：≥1−τ 次复现为 alive，≤τ 为 dead，中间为 *undecided*（排除计分但计数报告）。
- 两条审计规则：永不用标签给自己打分；再推导程序须与被推导步骤同性质（随机的用随机、符号的用符号）——entailed edges 用决策过程符号化再推导、由构造即一致，**被排除在所有忠实度数字之外**（所以忠实度结论只覆盖 merge 与 summary 两层）。
- 错误分两向且**从不合并成一个 accuracy**：*leaky*（标签说活、再推导说死，代价是继续供应失效内容）与 *over-deleting*（标签说死、再推导说活，代价是丢证据）。
- 评级是**全人口**而非抽样：一次删除触及的每个模型作者 artifact 都裁定；一个 (删除算子 × 层) 称一个 cell。
- 检索层落地指标 *tainted@k*：用语料自身索引元数据构造查询（每 MeSH 描述符一题、共 300 题），BM25 取 top-k，tainted@k = top-k 中含 oracle 判死 artifact 的查询比例。
- **关键概念区分**：审计测的是 *constructor-relative liveness*（这个构造器在只剩存活证据时会不会还产出它），不是 *source-groundedness*（存活来源是否真的支撑它）。模型能用参数记忆补出证据时，artifact 对构造器是活的、但对来源是无据的；**被遗忘权等合规场景要的是后者，而本文审计测不到后者**（作者明示，§4 与 §8）。

**管线与语料（§3）**：主实例为 300 篇 PubMed 肿瘤摘要（按共享 MeSH 描述符选取），其中 **61 篇是真实撤稿**，retract-source 用真实撤回；构造器为 Anthropic Claude Sonnet（2026 年 8 月经 Claude Code CLI 的 sonnet alias 调用，内容寻址缓存以模型+prompt 哈希为键；后续运行 pin 为 claude-sonnet-5 或 claude-opus-5-5）；产出 2,785 assertions / 2,784 claims / 165 merges / 59 summaries / 21,373 entailed edges = **24,381 artifacts**。第二域：150 篇美国最高法院判例（Caselaw Access Project, U.S. Reports 380–387 卷），1,571 assertions / 136 merges / 60 summaries / 6,137 edges；无真实撤稿，retract-source 随机抽 10%（seed 0），查询为 300 个当事人与引证字符串。第二构造器：用 claude-opus-5-5 以相同摘要/prompt/schema 重建生物医学图。

#### 2.3 实验结果

- **现状有多坏**：不清理时污染查询占 32.2%（retract）/36.8%（revoke）；deployed 式清理（只删直接抽取事实）后仅降到 **28.0%**、revoke 下**纹丝不动 36.8%**——因为损伤主要住在派生边与摘要里，不在事实里。
- **Replay 之死（Table 1）**：同语料两次独立构建的对称差 = 29,475 artifacts；删掉 61 篇撤稿文档的效应 = 6,248；比值约 0.21（分层：claims 0.15、merges 0.23、edges 0.22、summaries 0.34；摘要层两次构建 Jaccard 距离 1.000、毫无交集）。结论域被作者限定为「本文的随机管线」，确定性管线下 replay 是精确的。
- **层依赖忠实度（Table 2，中心结果）**：merges 与再推导一致率 **68.4%（retract）/ 84.6%（revoke）**；summaries（扁平标签）仅 **33.3% / 36.4%**；summary 错误几乎全是 leaky（66.7% / 63.6%），over-deletion 各 cell ≈0。十次额外随机 10% 撤回 pooled（238 个 merges、109 个 summaries 被裁定）：merges leaky 18.1% [13.7, 23.5]，summaries leaky 41.3% [32.5, 50.7]，summary over-deletion 恒为 0；真实撤稿的 66.7% 位于逐次范围（21%–70%，中位 41%）顶端。
- **形状即病因（Table 3）**：原 prompt 本来就「按句问支持」，但存储时把整个摘要存成一张扁平备选列表——标签语义却把每一组读成「足以支撑整篇摘要」，于是支撑某句的一组证据能让整篇摘要在其他句子证据全灭时继续存活。两种修复对照：
  - *whole-summary receipts*（只问「足以支撑整篇」的集合）：一致率 100% / 90.5%、leaky 归零，但不 informative——模型不再声称冗余，被触及摘要基本全删，与「不信任该层」政策等价；
  - ***sentence-level receipts*（一句一组备选，摘要标签 = 各句标签的合取）：一致率 100% / 87.6%，错误不再单边（revoke 下 leaky 4.1% vs over-deleted 8.3%）；只删被判死的句子可保留 5/7 与 28/28 个被触及摘要（共 33/35；保留 48+353 个活句子、删 61+103 个死句子），而整篇政策是 35 个全删**。附带观察：被要求逐句带 receipt 后，模型写出的摘要变成 857 句 × 平均 7 词（原 181 句 × 16 词），548 句只依赖单个断言——**receipt 的形状反过来改变了 artifact 的形状**，作者提醒「这样的摘要还是不是读者想要的摘要，是另一个问题」。
- **稳健性（§6.3）**：换 Opus 做 oracle，与 Sonnet oracle 在 92%/91% 已裁定 artifact 上一致且层序不变，分歧中 5/6 是 Opus 判得更死；claim-blind prompt 没救回 merge（over-deletion 反升至 26.3%/22.2%，作者归因于参数记忆盲点），summary 在所有 oracle 下 leaky 仍 57%–68%；最高法院域 merges 80.0%/76.2%、summaries 43.8%/12.5%（错误全为 leak），不信任 summary 层后污染 15.7%/4.7% → 0.0%、recall 99.9%/100%；Opus 构造器下层序与错误方向不变，但扁平 summary 的 leaky 程度随构造器大幅变化（8.3%/50.0% vs Sonnet 的 66.7%/63.6%）。
- **按裁决行事（Table 4）**：BM25、k=10、300 题：trust-all-labels 把 tainted@10 从 deployed 的 28.0%/36.8% 降到 0.8%/2.1%；**distrust-the-failing-layer（忽略 summary 标签、被触及摘要直接删）再减半到 0.4%/0.8%，recall@10 同为 99.9%/99.8%**，且零额外模型调用。代价核算：被裁定的 28 个被触及摘要中 27 个确实不再可推导，该政策只误杀 1 个活摘要；无标签的「凡 provenance 沾边全删」政策可达 0.0% 污染但多删图、recall 最多低 1.6 点——**是审计（决定哪里可信）而非自报本身提供安全**。
- **成本**：全部 labels 序列化 1.35 MB / 全图 5.45 MB（扁平 provenance 列表本就要 0.82 MB，故备选束的边际成本约图字节的 10%，每 artifact 12–94 字节）；receipt 占 merge 调用输出 token 的 38%、summary 调用的 52%，本次构建约 18k 输出 token、全图约 1/4 构建输出，按文内价约 **$0.30 一次性**；清理期零 token。延迟未单独测。
- **作者自述局限（§8，另 §7 的 harness 事故）**：①只测 liveness 非 groundedness，erasure 合规最要命的失败恰在盲区，能补的人审「可负担但没做」，盲评表与评分器已随库释放；②两域两构造器同属一个厂商、一套 prompt，语料 150–300 文档，summary leaky 幅度在 8%–88% 间随设置漂移、无规模律测量，协议要求每个 (构造器, prompt, 层) 先跑一次审计再校准信任；③检索是 BM25 关键词式，沿路径推理的消费者会被路径上任何死边污染，tainted@k 测不到；④只研究删除不研究新增（新增不改变已有支持集，但「活着的摘要何时变得不完整需重生成」未研究）；⑤entailed edges 被排除，忠实度只覆盖按数量计的少数模型作者 artifact；R=5、τ=0.2 裁决粗糙。§7 还披露缓存把配额提示/内容过滤拒绝当合法答案缓存了 1,123 次调用——「验证 agent 的推理之外，还必须验证管道（plumbing）」，此点与 ProvPurge 的工程自验设计有共鸣（分析判断）。

#### 2.4 代码/数据开源（部分未确认）

论文 §3 脚注给出 https://github.com/VishiATChoudhary/polygraph ，并称「每个模型面步骤的 prompt 模板随代码逐字释放」；§8 称盲评表与评分器「随库释放」，§6.3 称完整表格在 released repository 中。**但 2026-10-29 实际打开该 GitHub URL 返回 404，未能确认仓库当前公开可达**（可能未公开/已更名/抓取受限；按规则未重试）。引用时应写「论文声称开源、本次未能实测确认」，不要写成已开源事实。

---

### 3. PACMI — Provenance-Aware Cascading Memory Invalidation for Long-Term LLM Agents

来源：全文 https://arxiv.org/html/2610.05732 （2026-10-29 打开，verified live）；摘要页 https://arxiv.org/abs/2610.05732v1（提交 2026-10-05）；作者 Yiqi Wang、Jiaqi Liu、Jiaqi Zhang、Zhangkai Wu、Yiqun Duan、Mingkai Zheng、Taotao Cai（University of Southern Queensland、南科大、江苏大学、悉尼大学、Uploading Inc.）。

#### 3.1 研究问题、动机与声称贡献

**问题**：新证据到来后，旧记忆可能「语义上仍高度相关、却已失效」，且失效会沿依赖传到下游记录；同时旧记录对历史查询仍有价值。现有系统缺两样东西：依赖追踪（找出下游影响）与历史保留（非破坏地留住旧值）。它与 BEMULA 的分工很清楚：BEMULA 处理**用户显式删除请求**，PACMI 处理**新证据隐含的失效（stale）**——触发源不同。

声称贡献（§1 逐条）：
1. PACMI 框架：来源图 + 类型依赖边 + 四态有效性格 + 边类型驱动的传播；
2. 新诊断基准：100 cases、300 queries、五域，评测分 node/context/answer 三层；
3. 实证：最终答案准确率最高且与最强基线的配对差异显著；premise checker 在受控查询分布上完美；级联传播改善记忆状态正确性，但答案级差异在当前规模**不显著**（作者自己写进贡献条目，罕见地克制）。

**先读 Scope（§1 末，决定全文数字怎么读）**：A1 评测前图的依赖边**由基准标注给定**，本文研究「给定来源图的传播与检索」，不研究从原始文本归纳图；A2 每 case 只有一条新证据、固定两跳链、6 条先验记忆；A3 因此真值有效性标签是图拓扑的确定性函数，下游状态类指标只验证传播实现正确、**不是比较优势证据，作者明确不拿它 claim 优越性**。

#### 3.2 方法细节

**数据结构（§3）**：记忆与新证据同为 provenance graph $G_t=(V_t,E_t)$ 的节点，节点带文本、时间戳、来源、可选元数据（语义角色如事实/计划/推荐只是标注、不是运行时节点类）；评测前图含 *supports* 与 *used-by* 两类边，关系分类后 PACMI 可新增 *updates/contradicts* 边。**四态有效性格**（自弱到强地「非当前」）：
$$\textsc{active} < \textsc{needs-verification} < \textsc{historical-only} < \textsc{superseded}$$
该序不是约定俗成，而是当前导向查询的检索偏置序（依次 +0.12 / +0.04 / −0.10 / −0.22），并定义多路到达同一节点时「取更强状态」规则。历史查询的偏置反转为 +0.04/+0.04/+0.12/+0.12。状态赋值全程**非破坏**：原文保留、只记录 relation-judgment 与 cascade-invalidation 操作以备追溯。脚注另披露实现中还有 *stale*（contradict 直接目标）与 *invalidated*（经 derived-from 到达、完全排除检索）两态，但基准未实例化、未测——**引用四态时勿把这两态算进已证实范围**。

**完整流程（§4 + Algorithm 1；LLM 与确定性规则的分界是本文的关键细节）**：
1. 插入新证据 $e_{t+1}$，推理时状态先全初始化为 active；
2. **候选排序（确定性公式）**：$s_{cand}=0.82\,s_{cos}+0.18\,s_{tok}+0.08\,\mathbb{1}[\sigma_e=\sigma_i\neq other]$（cosine + token Jaccard + 同槽奖励），保留至多 k=6 个候选——因每 case 恰 6 条先验记忆，**本基准中排序不做任何剪枝**（作者明示）；
3. **关系分类（LLM 判断 + 确定性约束）**：辅助 LLM（一次批量调用打全部候选）对每候选预测 {duplicate, refine, update, contradict, temporally-compatible, unrelated} + 理由；随后确定性的 slot/stability/temporality 约束归一化预测——update/contradict 仅在「同单值槽 + 旧记忆本身是被替换的当前事实」时被接受，稳定偏好/规则/模板不得仅因职业地点变化而判 stale；有 incoming supports/used-by 边的候选被视为下游记录而非直接目标、交给传播处理；
4. 只有 update/contradict 且为直接目标者入集 $D$，赋关系特定的直接非当前状态（本基准中评测到的都是 update → superseded），并把预测的关系边加入图；
5. **级联传播（确定性 BFS）**：以 $D$ 为队列种子，沿 outgoing 依赖边按边类型映射下游状态——supports → needs-verification，used-by → historical-only；多路取格中更强者；未达节点保持 active；
6. **有效性感知检索（确定性打分）**：cosine + 状态偏置（见上）+ 新证据奖励 0.06 + 同槽奖励 0.05 + 归一化 recency 奖励 0.04，top-5（带状态与槽标签）构成上下文 $C_q$；
7. **Premise checking（LLM 判断，消费图状态而非仅检索上下文）**：把 superseded 旧记录与新证据+active 记录对照；若查询把过时事实当当前前提，则把识别出的新旧事实对塞进答案 prompt 的纠正块，否则用与基线相同的中性 prompt。图中无 superseded 记录时跳过且不发 LLM 调用。

运行例（§3）：酒店场景 $e_1$ 支持 $m_1$ 支持 claim $c_1$ 被 decision $d_1$ 使用；新证据「Hotel X 1 月 10 日售罄」把 $m_1$ 更新为 superseded → 沿 supports 把 $c_1$ 推为 needs-verification → 沿 used-by 把 $d_1$ 推为 historical-only；而语义相近的 hard negative $m_2$ 因无依赖边保持 active——这是「按依赖失效」区别于「按语义相似失效」的判别性案例。

**触发与删除语义**：没有删除。失效是状态降级 + 检索降权/分流（历史查询反而升权），原文永远在库。这一点与 ProvPurge 的硬删形成根本对照（见 §4）。

#### 3.3 数据与协议

自建 stale-memory dependency benchmark（§5.1 + 附录 A）：100 cases × 五域（Personal Profile、Location & Preference、Work & Project、Finance & Recommendation、Tool/Workflow State，各 20）。每 case 固定结构：6 条先验记忆构成两跳链 $M_1 \xrightarrow{supports} M_2 \xrightarrow{used-by} M_3$，$M_4$–$M_6$ 分别为仍有效相关记忆、语义相似 hard negative、无关 distractor（另含一条支持/上下文条目）；金标状态恒为 $M_1$=superseded、$M_2$=needs-verification、$M_3$=historical-only、$M_4$–$M_6$=active。每 case 1 条新证据 + 3 查询：Q1 当前状态、Q2 陈旧前提（查询预设了已被取代的旧值，须识别并纠正）、Q3 历史状态。无 dev/test 划分，固定评测集单次运行。

设置（附录 A）：final-answer 模型与 judge 均为 **deepseek-v4-pro**，中间操作（关系分类、premise check、基线的重写/合并/链接演化）为 **deepseek-v4-flash**，embedding 为 all-MiniLM-L6-v2，全部 temperature 0；PACMI 等多数方法上下文 top-5，**A-MEM-style 可扩展到 8 条（作者承认这是覆盖类指标的已知混杂）**。基线：Vector RAG、Recency-RAG（0.7 相似 + 0.3 recency）、LLM-Only Update（局部 LLM 分类+就地重写）、CUPMem-style（改编自 STALE 的 CUPMem：槽级 supersede + 历史可查旧值，但不传播下游）、A-MEM-style（语义链接扩展，无有效性传播）。

指标（§5.3 + 附录 B/C，精确定义）：
- Node 层：DR（直接 stale 节点 recall）、AR（受影响下游节点 recall），均为 case-macro recall；OI/UI = 预测非当前集合「含真值外节点」/「漏真值内节点」的 **case 比例**（注意：不是节点比例，任一错即整 case 计错）；
- Context 层（仅 Q1/Q2 平均）：SIC = 检索条目中 stale/affected 的比例；CVER = 「active 先验 + 新证据」中被检索到的比例；附 New-Evidence Coverage；
- Answer 层：Final Accuracy 按标注 expected behavior 由 judge 判；Stale Reuse = 答案是否把直接 stale 记忆当现行使用（**不独立评每个下游状态**）；Premise Resistance = Q2 准确率；Historical Preservation = Q3 准确率；
- 统计：10,000 次 case 级 bootstrap（seed 20260728）报 95% CI；预设配对比较用双侧 exact McNemar。

#### 3.4 实验结果

主结果（Table 1）：
| 方法 | DR | AR | UI | OI | SIC↓ | CVER | Q1 | Q2 | Q3 | Final | Reuse↓ |
|---|---|---|---|---|---|---|---|---|---|---|---|
| Vector RAG | – | – | – | – | 0.53 | 0.58 | 0.95 | 0.28 | 1.00 | 0.74 | 0.10 |
| Recency-RAG | – | – | – | – | 0.47 | 0.66 | 1.00 | 0.39 | 0.95 | 0.78 | 0.04 |
| LLM-Only Update | 1.00 | – | 0.68 | 0.00 | 0.49 | 0.64 | 1.00 | 0.47 | 0.42 | 0.63 | 0.03 |
| CUPMem-style | 0.93 | – | 1.00 | 0.00 | 0.40 | 0.76 | 0.97 | 0.43 | 1.00 | 0.80 | 0.03 |
| A-MEM-style | – | – | – | – | 0.47 | 0.88 | 1.00 | 0.71 | 1.00 | 0.90 | 0.00 |
| **PACMI** | 1.00 | 1.00 | 0.00 | 0.00 | **0.34** | 0.83 | 1.00 | **0.97** | 1.00 | **0.99** | 0.00 |

读法（§5.4，作者的克制很值得学）：①唯一的干净胜利在答案层：Final 0.99 vs 最强基线 A-MEM-style 0.90，即 300 题中 3 vs 29 个错，McNemar **p=2.56×10⁻⁶**；差距全在 Q2（两法 Q1/Q3 均触顶——**该基准的区分度几乎全在陈旧前提题**）。②PACMI 并不统治检索：CVER 0.83 输给 A-MEM-style 的 0.88（链接扩展拉入更多有效上下文 + 其上下文上限 8 > 5），只是 SIC 更低（少放旧货、以覆盖换洁净）。③Stale Reuse 两者都四舍五入为 0.00，是并列不是领先。④LLM-Only Update 的 DR 同为 1.00，却因就地重写毁掉历史值而 Q3 崩到 0.42（58 个历史题错误）——作者称之为支持非破坏失效的受控证据。⑤PACMI 的 3 个错误全发生在 premise 检测成功之后，属答案生成/落地失败（附录 I 逐例）。

消融（Table 2 + 附录 F/H）：
| 变体 | AR | UI | SIC | Final | Premise | Historical |
|---|---|---|---|---|---|---|
| PACMI | 1.00 | 0.00 | 0.34 | 0.99 | 1.00 | 1.00 |
| w/o Provenance | 0.00 | 1.00 | 0.37 | 0.97 | 0.98 | 1.00 |
| w/o Cascade | 0.00 | 1.00 | 0.41 | 0.96 | 0.98 | 0.99 |
| w/o Validity States | 1.00 | 0.00 | 0.56 | 0.97 | 0.98 | 1.00 |
| w/o Premise Check | 1.00 | 0.00 | 0.34 | **0.83** | **0.80** | 1.00 |
| w/o Hist. Preservation | 1.00 | 0.00 | 0.20 | **0.77** | 0.99 | **0.36** |

- 答案层最大的两个杠杆是 **premise check**（去掉后 Q2 错误 3→50，p=1.78×10⁻¹³；但该消融同时去掉检测器与纠正 prompt，只能归因于「组合模块」）与**历史保留**（去掉后 Hist 1.00→0.36）；
- **级联本身**：去掉后 AR 归零、UI 全错（状态层全灭），但 Final 仅 0.99→0.96（错误 3→11），**McNemar p=0.0574，未达 0.05**。作者给的解释是结构性的：每 case 6 条记忆、上下文预算 5、新证据在 98.7% 的查询上下文中被检索到，答案模型本来就同时看见新证据与受影响记录——级联的价值在当前规模主要是状态正确性与可审计性，「更大存储/更深链下是否出现答案级差距是开放问题」；
- premise checker 本征评测：300 题上 TP=100、TN=200、FP=FN=0，P/R/F1=1.00——但作者反复强调这条件于「图状态已把新旧事实分好 + Q2 直接预设旧值 + Q3 明示问过去」的受控分布，**不得解读为开放场景的完美检测**；
- 成本：PACMI 辅助 LLM 调用共 400 次（每更新 1 次批量关系分类 + 每查询 ≤1 次 premise check），更新感知基线为 100 次——0.09 的 Final 优势是用 300 次查询期调用换的；作者建议查询密集型部署应以「检索上下文是否含非 active 记录」门控 premise check。只数了调用数，未测 latency/token。

作者自述局限（§6 + 附录 J）：A1 使 PACMI 拿到基线没有的结构信息，比较应读作「给定来源图时能做什么」，最缺的实验是图质量退化研究（oracle 边 / LLM 归纳边 / 受控腐蚀边）；固定两跳拓扑使 OI 根本测不了（没有「连着 stale 却应保 active」的下游节点）、多路状态消解从未被测；6 条记忆使检索近饱和、结论不外推真实规模；judge 与生成同模型家族（自偏好风险）、单次运行、方法间上下文预算不等；stale-reuse judge 不独立评下游状态。附录 J 主动列出移除每个假设所需的基准扩展（变深度/分支/汇合、图质量三档、记忆规模远超上下文预算、多 backbone + 跨家族 judge + 人一致性）。

#### 3.5 代码/数据开源（未确认）

摘要与正文只称「Code and data will be made publicly available」，正文多处提及 released results/logs/analysis script，但**全文未出现仓库 URL，本次未在页面上确认到可获取地址**。按要求记为未获取，勿在开题里写「PACMI 已开源可直接跑基线」。

---

### 4. 三篇横向对比 + 对 ProvPurge 的直接启示

#### 4.1 对照表

| 维度 | BEMULA | PolyGraph | PACMI | ProvPurge（拟定） |
|---|---|---|---|---|
| 性质 | 基准（无新方法） | 方法+审计协议（文档图谱维护） | 方法+诊断基准（记忆失效管理） | 方法（对话记忆硬删） |
| 触发源 | 用户**显式**自然语言遗忘请求 | 来源**撤回/谓词吊销**（外部治理事件） | **新证据**隐含冲突（无删除意图） | 显式遗忘请求 + 知识更新（二者都要） |
| 来源粒度 | 事实级：(s,p,o) 原子/派生事实，对话轮次为载体；**无写时来源记录**（金标靠构造管线的事后标注） | **断言级→artifact 级**：receipt 是断言束的集合（DNF 式备选支持）；摘要层须细到**句级**才可信 | **记忆节点级 + 类型边**：边由标注给定，节点带时间戳/来源字段；不做句级分解 | **句级**：事实/向量/摘要句三层各自带来源（与 PolyGraph 的修复结论同向） |
| 删除对象 | 对话中披露的个人事实（269 个目标事实/135 请求） | 图 artifact：claims、merges、entailed edges、summaries | 无删除对象；对象是被降级的记忆节点及其下游 | 事实条目、向量索引项、摘要句（三层冗余副本） |
| 删除方式 | 被测系统各行其道；基准只判结果。语义要求是真删（[FORGOTTEN]/[NO_LEAK]） | **硬删**（按 receipt 存活判据移除 artifact；句级时可只删死句子） | **失效保留**：四态降级 + 检索偏置，原文永留、历史查询可达 | **硬删 + 确定性级联**（目标是任何层、任何探针都捞不回） |
| 确定性 vs LLM | 构造/判分全用 LLM；被测侧不限 | 写时 LLM 自报 receipt；**删时纯集合运算零模型调用**；审计时才用模型再推导 | 关系分类与 premise check 用 LLM（受确定性约束/门控）；传播、打分、状态合并是确定性规则 | 写时 LLM 抽取+挂来源（须自报）；删时确定性级联；删后探针自验（部分需 LLM judge） |
| 验证方式 | 五套探针 + 成对 control + 五类固定攻击框架 + LLM judge | re-derivation 审计（删证据后看构造器是否复现，R=5, τ=0.2）+ tainted@k 检索污染 | node/context/answer 三层指标 + premise 本征评测 + bootstrap/McNemar | 拟用攻击探针自验（须对齐 BEMULA 五类攻击 + 残余恢复式探针） |
| 适用对象 | 对话长期记忆 | 文档知识图谱（GraphRAG 式） | 对话/任务型长期记忆（受控小规模） | 对话长期记忆 |
| 规模实证 | 50 对话/平均 84.8K tok/最长 379K tok | 300 文档/24,381 artifacts | 100 cases × 6 记忆（检索近饱和） | 拟 BEMULA + LongMemEval-S 子集 + STALE（规模远超 PACMI、形态对齐 BEMULA） |
| 最重要的负面/警告 | 现有系统 RPR ≤1.5% | 扁平摘要 receipt 约 2/3 不可信且单边漏删 | 级联的答案级效应不显著（p=0.0574） | ——（这三条都是写给你的） |

#### 4.2 撞车点、差异一句话、基线/必引判定

**与 PolyGraph（撞车最深，必须作为方法基线 + 必引）**
- 最容易被判撞车的点：「写时记录来源、删时沿来源确定性清除」这一**主机制**与 receipts + disjointness cleanup 几乎同构；若 ProvPurge 的摘要层来源是整段/整条粒度，PolyGraph 的 Table 2/3 就是现成的反例（约 2/3 错、且错在漏删方向）。
- 差异一句话：**「PolyGraph 清的是文档图谱中可由备选证据束再支撑的 artifact、且只测构造器相对存活；ProvPurge 清的是对话记忆中用户点名要被遗忘的事实在事实/向量/摘要句三层的全部冗余副本，并以被遗忘权语义（捞不回、推不出）而非再推导存活为验收标准。」**
- 判定：**必引 + 必须做基线**（至少做「receipt 式来源 + 集合清除」在 BEMULA 上的复现基线，否则审稿人会问你与它的差别是否只是换了数据集）。

**与 PACMI（叙事撞车，必须必引；基线可选做、但其基准设定不可直接比）**
- 最容易被判撞车的点：「来源图 + 级联传播处理下游失效」的故事线、以及四态/多态有效性状态的表述；若 ProvPurge 也用「下游节点状态降级」措辞，会被读成 PACMI 的硬删变体。
- 差异一句话：**「PACMI 是新证据触发的非破坏失效（旧值降级保留、历史可查、边由标注给定）；ProvPurge 是删除请求触发的破坏性清除（旧值在三层存储中物理移除、历史亦不可达、来源在写时自生成而非标注给定）。」**
- 判定：**必引**；作为基线时只能借其思想做「失效保留版 ProvPurge」对照臂（恰好能凸显硬删的合规差异），其自建基准因 A1/A3（边给定、真值=拓扑函数、6 条记忆）不能作为 ProvPurge 的主评测——在 PACMI 基准上比 Final 没有信息量（Q1/Q3 触顶）。

**与 BEMULA（不是撞车，是考场；必引 + 主基准）**
- 最容易被判「撞」的点其实是反向的：BEMULA 已把「首个记忆卸载基准 + 六类请求 + 攻击探针」占住，ProvPurge 不能再 claim 任何「首次提出遗忘评测/攻击验证」的说法；且它的 RPR 定义（全题+全 preservation 全对）意味着任何单点漏删都会把整条请求判死。
- 差异一句话：**「BEMULA 是评测协议与负面结论，本身不提供任何删除机制；ProvPurge 是第一个（拟）以写时句级来源 + 确定性三层级联去正面攻克 BEMULA 全套判据（FE/AR/PR/RLA/RPR）的删除机制。」**（「第一个」属拟定说法，须在查全 MemCut 等同期工作后再定，见 §4.4。）
- 判定：**必引 + 主基准**（含 BEMULA-Lite 做低成本迭代）；其六系统 × 三 backbone 的结果表是 ProvPurge 必须同表对比的对象。

#### 4.3 可直接复用 / 不可复用的实验设置、指标与探针

**可直接复用（已证实其定义与实现公开）**：
1. BEMULA 的全部指标与协议：成对 control + 有效正确性 $\tilde z=z\wedge c$、RA/FE/PR/RLA/AR/RPR、Score=FA×UG、五类固定攻击框架（间接/角色扮演/越狱前缀/布尔/多跳）与 [FORGOTTEN]/[NO_LEAK] 判分——代码与数据已实测公开，可直接跑。
2. BEMULA 的六类请求分类法（F1–F6）与 F4 的特殊语义（只判派生结论、源事实存亡不计）——ProvPurge 的级联范围必须按此逐类声明：例如 F4 下「连源带结论全删」与「保源抑制推理」都合规，但你的确定性级联若默认前者，要在 PR 上确认不误伤该实例的 preservation 题（F4 源事实已被排除出 preservation 池，原文已证实，这对级联删除反而是利好）。
3. PACMI 的三层评测分解（node/context/answer）与 OI/UI 的 case 级过/欠失效定义——非常适合做 ProvPurge 的**内在（intrinsic）评测**：删对了哪些节点、上下文里还剩多少 stale、答案才对不对，分开报告，避免只报 QA 分被质疑掩盖机制错误（此点与 FEB-30 的内在评测主张互相印证，见 §5）。
4. PACMI 的检索状态偏置数值与候选打分公式可作为「失效保留对照臂」的现成实现规格；其统计规格（case 级 bootstrap 10,000 次 + 配对 McNemar）可直接搬。
5. PolyGraph 的成本核算口径（来源记录的存储字节占比、构建期额外输出 token、清理期 ms 级零模型调用）与错误双向拆分（leaky vs over-deleting 永不合并）——ProvPurge 报告删除质量时应同样分「漏删率 / 过删率」双向报，不要只报一个 accuracy。
6. PolyGraph 的审计思想可改造为 ProvPurge 的内部消融：抽一批 artifact，把其所引证据删掉后看系统是否仍能产出/答出它——即把「再推导」换成「再问答」。

**不可复用（或复用即翻车）**：
1. **PACMI 的自建基准不能当主证据**：边给定（A1）、真值是拓扑确定函数（A3）、6 条记忆检索近饱和、OI 不可测——在其上刷 AR/DR=1.00 等于自证实现正确，审稿人（尤其读过 PACMI Scope 一节的）会直接引用作者自己的话驳你。
2. **PACMI 的 premise checker 完美分数不可借用为预期**：1.00 条件于已分好的图状态与模板题型；ProvPurge 若做删后自验探针，结果必须在 BEMULA 式自然措辞+攻击框架下报告。
3. **PolyGraph 的 liveness 验收标准不能搬到被遗忘权场景**：它的 alive 允许「模型凭参数记忆重建」，这在 erasure 语义下正是泄漏（作者自己在 §8 承认）。ProvPurge 的验收必须是 groundedness 侧的（存活来源里推不出、检索不到、攻击问不出），与 BEMULA 的 AR/[NO_LEAK] 同侧。
4. **PolyGraph 的 BM25 tainted@k 不能替代向量检索污染测量**：ProvPurge 有向量层，须用自己的向量/BM25 混合检索复测污染，tainted 的定义可借、管线不可照抄。
5. **BEMULA 的攻击框架不可改写措辞后仍称「用了 BEMULA 攻击套件」**：原文明确框架从不改写、措辞即攻击向量；自创探针可以加，但要与原五类分开报告。
6. **BEMULA 的数据生成管线不可低成本复刻**：GPT-5.6-Luna、temp 0.9、五阶段多智能体 + 人工抽检的成本与它的 judge 同源性（生成与判分异模型但同属合成分布）都是既定事实；ProvPurge 用其**发布的数据**即可，不要自建「类 BEMULA」小数据再比——规模与分布都不可比。

#### 4.4 读完三篇后，ProvPurge 还剩的真实新意与必须放弃/改口的说法

**仍站得住的真实新意（按强度排序，均为分析判断）**：
1. **硬删语义 × 冗余三层的一致清除**：三篇没有一篇做「同一事实在事实库/向量索引/摘要句三层冗余副本的物理一致删除」。BEMULA 只诊断冗余副本是失败根源之一（§1 明言 raw turns/extracted facts/compressed notes 多副本使删一份留多份）；PolyGraph 的层是图谱推导层而非存储冗余层、且验收是 liveness；PACMI 明确不删。**「面向被遗忘权的三层硬删闭包」是空白点**——这是 ProvPurge 最硬的新意，应升为第一卖点。
2. **删除对象从「文档/断言」迁移到「对话轮次→事实」的写时句级来源**：PolyGraph 的来源在文档断言层、构造器是抽取管线；对话记忆的来源链（哪轮对话→哪条抽取事实→哪个向量块→哪句摘要）无人做过句级对齐。但请注意：MemCut 已标注了对话记忆中的 evidence trails（见 §5），此新意须与它划界（它做评测与残余恢复测量、不做写时来源驱动的确定性清除——此划界基于其摘要，已证实范围仅至摘要）。
3. **删后自验做成系统组件而非仅评测**：BEMULA 的攻击套件是外部考官；把五类攻击探针内化为「删除事务的提交前自检（不过则扩删/告警）」在三篇中均无对应物（PolyGraph 的审计是离线校准哪层 receipts 可信、且对象是构造器自报，不是每次删除的在线验收）。这是第二卖点，但必须给出自验的假阴性讨论（探针过 ≠ 真删净，MemCut 的 residual recovery 正是这类反例的度量方式）。
4. **触发统一**：同一套来源与级联同时服务「显式遗忘请求」（BEMULA 侧）与「知识更新替换」（F6/LongMemEval-S 知识更新/STALE 侧）——三篇各占一端，无人统一。注意 STALE 侧是隐含失效、无删除意图，把它纳入主基准时须明确你的系统在 STALE 上做的是「更新即删旧」还是「失效保留」，不要混报。

**必须放弃或改口的原说法**：
1. 放弃「首次提出写时来源/来源驱动删除」——PolyGraph 已占（receipts），且其术语（support label、写时发出、删时零模型调用）比你更早成文；改口为「首次将写时句级来源用于**对话记忆的三层冗余硬删**（据本次检索范围）」。
2. 放弃「确定性级联清除是无人做过的机制创新」这种笼统说法——级联传播本身有 TMS/DRed/视图维护的经典谱系，PACMI 与 PolyGraph 都已明确承接；你的创新点要落在**级联的扇出对象**（三层存储副本 + 摘要句）与**确定性**（删时零 LLM 判断）上，并主动引用这条谱系。
3. 放弃「删除后用探针验证即可证明删净」——三篇共同的教训是验证有盲区：PolyGraph 的 liveness 盲区（参数记忆重建）、BEMULA 的 judge 盲区、MemCut 的 residual recovery（39–76% 可从残存证据恢复）。改口为「探针自验是必要不充分条件，并报告其残余恢复率」。
4. 放弃用 PACMI 式状态指标（AR/DR 满分）预演自己的成果——在真值由构造决定的设定下满分无信息量；ProvPurge 的状态层指标必须在 BEMULA（金标独立于你的来源结构）上报。
5. 改口「摘要层来源」的一切表述：必须一开始就声明句级（sentence-level receipts），并引用 PolyGraph Table 3 作为设计依据；若实现中向量块的来源只能到「块级」，要按 PolyGraph 的教训预设该层不可信、给出类似「不信任则整块删」的分层政策，而不是默认全层可信。
6. 「主基准 BEMULA + LongMemEval-S + STALE」的组合要补一句边界：三者触发语义不同（显式删除/知识更新/隐含失效），结果须分基准报告、不可合并成单一总分——BEMULA 作者对「把遗忘混进综合分」的做法（Memora 的 FAMA 单指标）正是批评态度。

#### 4.5 精读这三篇时要带着看的 10 个问题清单

1. BEMULA 附录 C.2：被测系统事先不知道哪轮是遗忘请求、评测方不代发 delete 调用——ProvPurge 的删除触发在同样条件下能被自然语言请求可靠触发吗？触发失败算谁的？
2. BEMULA 式 (2)：$\tilde z = z\wedge c$ 的成对计分下，ProvPurge 若在 control 版上 RA 不够高，FE 再高也会被清零——你的系统先过得了「先记住」这一关吗（尤其长档）？
3. F4 的语义（附录 A）：你的级联删源事实还是保源抑推？两种合规路径下，你的 AR（尤其多跳攻击）分别是多少？
4. PolyGraph §3：你的 receipt/来源结构是 DNF 备选束（「任一束充分即可活」）还是单链来源（「任一环断即死」）？对话事实多源互证时，单链结构会不会系统性过删？
5. PolyGraph Table 3：你的每一层来源粒度与它所支撑的断言粒度是否同形？哪一层最像「扁平摘要标签」，为它准备的「不信任政策」是什么？
6. PolyGraph §4/§8：你的删后验收测的是 liveness（还能不能被系统产出）还是 groundedness（存活证据是否还支撑它）？参数记忆重建这条泄漏道，你的探针测得到吗？
7. PACMI Scope A1–A3：把 PACMI 的每个数字按「给定图/拓扑真值/6 条记忆」打折后，还剩哪些结论值得你引用？哪一句是作者自己声明不能 claim 的？
8. PACMI 消融表：级联 p=0.0574 不显著、premise check 才是答案层大头——ProvPurge 的消融要预先设计到能回答「级联到底贡献了几个点、置信区间多大」，而不是只报点估计。
9. 三篇的 judge 都是 LLM 且各有同源风险（BEMULA 生成与判分、BEMULA judge=DeepSeek 与被测 backbone 之一同名、PACMI judge 与生成同家族）：ProvPurge 的评测如何做 judge 去偏（跨家族 judge + 人一致性抽检）？
10. 三篇的成本账（PolyGraph 约 $0.30/图的一次性 receipt 成本、PACMI 4 倍辅助调用、BEMULA 全量跑一遍的 API 成本）：ProvPurge 写时多付的来源生成 token、删时级联的延迟、自验探针的调用数，分别是多少、随记忆规模怎么长？

---

### 5. Google Scholar 核查（单独标注来源）

> 本节来源为 Google Scholar 页面，与上述 arXiv 全文来源分开标注。访问经过：2026-10-29，研究浏览器**直连** scholar.google.com 三次均被拦截，页面原文为 “We're sorry... but your computer or network may be sending automated queries. To protect our users, we can't process your request right now.”（verified live）；其后改经 web index 抓取 Scholar 页面，BEMULA 与 PACMI 两次成功、PolyGraph 一次返回 403（access denied），按指示对该 403 请求**未重试、未绕过**。

| 论文 | Scholar 收录 | 页面显示的作者/版本 | 被引 | 备注 |
|---|---|---|---|---|
| BEMULA | **已确认收录**（抓取成功） | 「B Xie, C Wang, G Ge, S Wang, F Niu, H Zhang… - arXiv preprint arXiv …, 2026 - arxiv.org」，有 [PDF] 与 HTML 版入口 | **Cited by 1** | 查询页：https://scholar.google.com/scholar?q=%22Can+Agents+Forget+When+Asked%22+Bemula |
| PACMI | **抓到条目内容**（返回该文摘要全文 + arxiv.org），但抓取文本未显示标题/作者行与 Cited by 计数 | 未显示 | **未获取**（不推测为 0） | 查询页：https://scholar.google.com/scholar?q=%22PACMI%22+Provenance-Aware+Cascading+Memory+Invalidation |
| PolyGraph | **未核实**（抓取 403） | 未获取 | 未获取 | arXiv 摘要页仅把 Google Scholar 列为 References & Citations 工具链接，**不构成收录证据**，故不据此断言已收录 |

**BEMULA 的 Cited by 1 是谁**（点开 Cited by 链接，Scholar 页面，verified via index）：唯一引用者为 *Machine Unlearning Under Model Evolution: A Survey of Durability, Recovery, and Lineage*，M Partohaghighi，SSRN，2026-10-21。链接：https://scholar.google.com/scholar?cites=13474922009107902801&as_sdt=2005&sciodt=0,5&hl=en&oe=ASCII

**顺 Scholar 的 Related articles 列表发现的同主题工作**（Related 页与 BEMULA 条目同簇出现，Scholar 页面 verified via index；下列每条的定位另经 arXiv 索引摘要核实，标注至摘要级、未精读全文）：
- **MemCut: Every Secret Leaves a Trail in the Long-Term Memory of LLM Agents**（Cheng Zhang, Xuanning Cao, Zhicheng Zheng, Hui Xiong，arXiv:2610.21405，2026-10-15；Scholar 显示 Cited by 1）——与 ProvPurge 关系最近的新工作：456 个人工设计秘密 + **全标注 evidence trails**，区分 native recovery（经系统检索恢复）与 **residual recovery**（把溯源到的残存对象直接喂 reader 的恢复率）；结论是披露删除对 native 降得多、对 residual 降得少，LLM 引导的多删 10 轮后 residual recovery 仍 39–76%，且**候选覆盖率（candidate coverage）是参考删除基线的瓶颈**。它占住了「删后残余可恢复性」的评测叙事，ProvPurge 的自验探针必须正面回应它（建议列为必引 + 其 residual recovery 口径可直接借作你的自验指标）。
- 其余 Related 条目多为记忆系统/评测的一般文献（如 InjecMEM, Cited by 10；Can Agent Memory Systems Track Evolving State?, arXiv:2608.19652, Cited by 12；MemoryAgentBench 的 ICLR 版, Cited by 306），与「删除/来源级联」不直接撞车，不展开。

**经 arXiv 索引（非 Scholar 页面）补充核实的同主题新工作**（为避免把非 Scholar 来源混入上表，单独列此；均只核实到摘要/公开转述级）：
- **Beyond Downstream QA / FEB-30**（Toma et al., arXiv:2610.22209, 2026-10-15）：主张直接检查记忆存储内容（intrinsic evaluation），被测系统对 create/update/reinforce/supersede 的操作分类仅 20% 正确——支撑 ProvPurge 做 node 层内在评测的必要性。
- **Control-Plane Placement Shapes Forgetting / LETHE + ForgetEval**（arXiv:2606.15903, 2026-06）：13 种配置对照「确定性原语 / 写时 LLM / 变异时 hook」，参考实现含 signed purge receipts 与向量+FTS 重索引原语，ForgetEval 385 对抗 cases——它与 ProvPurge 在「确定性删除原语 + purge 回执」措辞上部分重叠，开题前建议精读（本报告未读其全文，撞车程度待核）。
- **TRACE**（Xiong et al., arXiv:2609.33517, 2026-09-27）：多智能体返回时的记忆准入治理，在 Memora 与 STALE Type II 上评测——属失效治理侧，与 PACMI 同侧、与硬删不同侧。
- 三篇正文已提及且 ProvPurge 绕不开的：**STALE**（Chao et al., arXiv:2605.06527，含 CUPMem 原型）、**Memora/FAMA**（Uddin et al., ACL 2026 Findings, arXiv:2604.20006）、**MemConflict**（Tao et al., arXiv:2605.20926）。

---

## Could not verify

1. **PolyGraph 的 Google Scholar 收录与被引**：直连被自动化查询拦截、index 抓取返回 403，未重试未绕过——未核实（见 §5）。
2. **PACMI 的 Google Scholar 被引数**：抓取只返回摘要文本，未显示计数——未获取，未推测。
3. **PolyGraph 代码仓库可达性**：论文给出的 GitHub URL 本次打开返回 404——论文「已释放」的说法未能实测确认（仓库可能未公开、更名或抓取受限，三者无法区分）。
4. **PACMI 代码/数据地址**：全文只称将公开，未给出 URL——未获取。
5. **MemCut、LETHE/ForgetEval、TRACE、FEB-30 未精读全文**：本报告对它们仅核实到 arXiv 摘要/Scholar 条目级，已在文中逐处标注；其与 ProvPurge 的精确撞车程度需另行精读后才能定论。
6. BEMULA 图 4 的逐类型 FE 精确数值在 HTML 全文中以图呈现、正文只给定性与少数数值（如 F6 ≥61.9、A-MEM F1/F2 46.2/52.0）；报告未引用图中未在正文出现的数字。

## Sources

全部于 2026-10-29 打开；「全文」指 arXiv HTML 全文逐节读完（含附录）。

1. BEMULA 全文（verified live）: https://arxiv.org/html/2610.18052v1 ；摘要页: https://arxiv.org/abs/2610.18052v1
2. BEMULA 代码仓库（verified live，Public）: https://github.com/Nboxff/bemula-benchmark
3. PolyGraph 全文（verified live）: https://arxiv.org/html/2610.17719 ；摘要页: https://arxiv.org/abs/2610.17719v1 ；论文声称的代码地址（本次打开 404，未确认）: https://github.com/VishiATChoudhary/polygraph
4. PACMI 全文（verified live）: https://arxiv.org/html/2610.05732 ；摘要页: https://arxiv.org/abs/2610.05732v1
5. Google Scholar — BEMULA 查询页（index 抓取）: https://scholar.google.com/scholar?q=%22Can+Agents+Forget+When+Asked%22+Bemula ；其 Cited by 页: https://scholar.google.com/scholar?cites=13474922009107902801&as_sdt=2005&sciodt=0,5&hl=en&oe=ASCII ；其 Related 页（会话参数 URL，正文已述）与 PACMI 查询页: https://scholar.google.com/scholar?q=%22PACMI%22+Provenance-Aware+Cascading+Memory+Invalidation ；Scholar 直连拦截页（研究浏览器三次，同一提示）: https://scholar.google.com/scholar?q=%22Keep+Your+Receipts%22+Trustworthy+Cleanup+Knowledge+Graphs
6. 同主题新工作（arXiv 索引，摘要级）: MemCut https://arxiv.org/abs/2610.21405v1 ；Beyond Downstream QA https://arxiv.org/abs/2610.22209v1 ；Control-Plane Placement https://arxiv.org/html/2606.15903 ；TRACE https://arxiv.org/abs/2609.33517v1
7. 工作笔记: notes/bemula.md, notes/polygraph.md, notes/pacmi.md, notes/scholar.md
