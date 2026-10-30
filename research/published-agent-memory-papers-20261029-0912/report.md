# LLM Agent 长期记忆的知识更新与选择性遗忘（ProvPurge）文献调研报告

> 对象：一名人工智能硕士研究生，拟定方案暂名 **ProvPurge**（按名称与方向理解为：基于溯源/provenance 的记忆净化——当某条记忆的来源失效、被撤回、被更新或被要求删除时，沿溯源链选择性清除该记忆及其派生记忆，并验证清除效果。此理解是本报告的工作假设，任务文本未给出 ProvPurge 的更细定义，见 §6.4 注记）。
> 调研日期：**2026-10-29**。检索路径：Google Scholar 网页索引（`site:scholar.google.com` 检索无结果返回，Scholar 页面在本环境不可直接读到被引数字）、ACL Anthology、OpenReview、AAAI OJS、NeurIPS.cc、PMLR、ACM DL、CCF 官网、DBLP/Semantic Scholar 风格的二手索引与各论文官方代码仓库交叉核对。
> **诚实声明（贯穿全报告）**：①凡写“已核实”者，均指本轮实际打开或经网页索引看到官方页面（ACL Anthology、出版社、会议官网、官方仓库）；凡写“索引显示/未核实”者，不得当作已证实事实引用。②**Google Scholar 被引量：本轮全部论文均记“未获取”**——Scholar 直连受限、索引检索无被引数字返回，按用户要求不编造、不以 Semantic Scholar/Liner 等其他平台数字冒充 Scholar 数字。③CCF 类别仅以实际打开的 CCF 官网页面为准（人工智能类页面已打开核实，见 §1.2）。

---

## 摘要（先给结论）

1. **先综述、后论文的读法成立且必要**：该方向至少有 4 篇可核实的已发表综述（TOIS 2025 的 agent 记忆综述、CSUR 2023 的机器遗忘综述、CSUR 2025 的知识编辑综述、Artificial Intelligence Review 2025 的 LLM unlearning 综述）搭好了坐标系；另有 5 篇影响很大的综述**只有预印本**（含 *Memory in the Age of AI Agents*、*Memory for Autonomous LLM Agents*、*Rethinking Machine Unlearning for LLMs*），本报告单独标注、绝不冒充已发表。
2. **具体论文入选 14 篇**（全部正式发表、同行评审）：Generative Agents、Reflexion、MemoryBank、ExpeL、LoCoMo、MemoryLLM、WISE、TOFU、LongMemEval、A-MEM、Mem0、Selective Forgetting (SEUL)、Memory-R1、Memora。其中 **Memora 是 ACL 2026 Findings，单独标注**（部分评价体系不计 Findings）。
3. **明确排除**：MemGPT（仅 arXiv:2310.08560，未核实到正式主会/期刊版本）、Zep（仅 arXiv:2501.13956/CoRR）、Who's Harry Potter 原稿（仅 arXiv 与 workshop 相关记录，不计主会长文）。三者可作思想源头引用，但**不能列入“已发表代表作”清单**。
4. **与 ProvPurge 最接近的五篇**：Memora（遗忘感知评测 FAMA）、Memory-R1（学出来的 DELETE 操作）、Mem0（生产级 ADD/UPDATE/DELETE 流水线）、MemoryBank（遗忘曲线式衰减）、SEUL/TOFU（参数侧选择性遗忘与可验证指标）。ProvPurge 的划界点应落在：**对象是外置 agent 记忆而非模型权重；触发信号是“来源失效/溯源”而非时间衰减或 RL 奖励；删除对象包含派生记忆（摘要、反思、图边、向量索引）；且给出可审计的删除验证**——这四点没有任何一篇已发表工作同时占全。
5. 报告 §5–§6 给出该方向论文的通用写法、审稿人买账的叙事、以及一份可直接套用的 ProvPurge 二级标题大纲（含每节写什么、放哪张图/表）。

---

## 一、入选标准与核查方法

### 1.1 标准
- 只收**正式发表、同行评审、有明确会议/期刊与年份**的论文；arXiv-only、workshop、demo、technical report 不入正选清单（Findings 例外单独标注，因其经同行评审但评价口径特殊）。
- 与主题的相关性排序：记忆的**更新/冲突处理 > 选择性遗忘/删除 > 可验证删除**优先；纯检索、纯长上下文工作除非是领域基准（LoCoMo、LongMemEval）否则不收。

### 1.2 CCF 核实情况（本轮实际打开）
- 已打开 CCF 官网人工智能类页面（https://www.ccf.org.cn/Academic_Evaluation/AI/ ，2026-10-29）：**A 类会议含 AAAI、NeurIPS、ACL、ICML、CVPR、ICCV、IJCAI（共 7 个，该页 A 名单未见 ICLR）；B 类含 ECAI、EMNLP、COLING、NAACL 等**。
- 已由索引看到 CCF 数据库/数据挖掘类页面将 **TOIS 列为 A 类期刊**；TOIS 综述作者主页亦标注 “TOIS 2025 (CCF-A)”（https://zeyu-zhang.cn/publication.html ）。
- **未核实**：UIST（人机交互类页面本轮未打开）、ACM Computing Surveys、Artificial Intelligence Review、COLM（很可能不在 CCF 目录内，但本报告只写“未核实”）、ICLR（见上：CCF 人工智能 A 名单未见，**请勿自行标 ICLR 为 CCF-A**，如学校另有认定以学校文件为准）。
- CCF 目录说明（索引所见）：目录所指会议论文为 Full/Regular paper，Workshop 等不计入考虑——这正是 Who's Harry Potter workshop 记录不能等同主会的原因。

---

## 二、综述怎么说（先综述）

### 2.1 已正式发表的综述（可放心引用）

#### 综述 S1｜Zhang et al., *A Survey on the Memory Mechanism of Large Language Model based Agents*, ACM Transactions on Information Systems (TOIS), 2025
- **发表信息（已核实）**：Zeyu Zhang, Xiaohe Bo, Chen Ma, Rui Li, Xu Chen, Quanyu Dai, Jieming Zhu, Zhenhua Dong, Ji-Rong Wen. *ACM TOIS* 43(6): 1–47, 2025，DOI 10.1145/3748302（图书馆记录标注 Peer Reviewed；作者主页标注 TOIS 2025、CCF-A）。预印本 arXiv:2404.13501。配套仓库：https://github.com/nuster1128/LLM_Agent_Memory_Survey （论文摘要与作者页提及，仓库本身本轮未单独打开，记索引显示）。
- Scholar 被引：未获取。CCF：TOIS 为 A（见 §1.2）。
- **分类体系（本文的骨架，ProvPurge 应优先借它）**：
  - **记忆来源**：inside-trial（单次试验内）、cross-trial（跨试验积累）、external knowledge（外部知识）。
  - **记忆形式**：textual memory（显式文本：全历史/近期历史/检索历史/外部知识） vs parametric memory（隐式参数：微调、知识编辑）。
  - **记忆操作**：**Writing（写）— Management（管：merging 合并、reflecting 反思/总结、forgetting 遗忘无关或过时信息）— Reading（读：检索）**。这就是业内常说的“写–管–读”三分。
  - 其后按设计、评测、应用组织全文，末尾分析局限与未来方向。
- **对开放问题的指向**：遗忘在该框架里只是 Management 的一个子动作，且多被实现为“基于时间/重要性的衰减”，**没有“按来源失效精确删除并验证”的操作定义**；评测部分也以回忆/任务成功为主，删除正确性不是一级指标。

#### 综述 S2｜Xu et al., *Machine Unlearning: A Survey*, ACM Computing Surveys, 2023
- **发表信息（已核实）**：Heng Xu, Tianqing Zhu, Lefeng Zhang, Wanlei Zhou, Philip S. Yu. *ACM Computing Surveys* 56(1), Article 9, 2023 年 8 月（期刊 PDF 页眉经索引核实：https://par.nsf.gov/servlets/purl/10520867 ）。CCF：未核实。Scholar 被引：未获取。
- **分类体系**：先给形式化定义与三项 desiderata——**consistency（遗忘后与“从未学过”的一致性）、accuracy（保留集效用）、verifiability（可验证性）**；方法按 **data reorganization（数据重组，如分片重训练 SISA 路线） vs model manipulation（模型操纵，如影响函数、梯度类近似遗忘）** 划分，另设 verification 机制专章（实验型 vs 理论型验证）。
- **对 ProvPurge 的价值**：它提供了“删除不是把分数变低，而是要可验证地接近重训练模型”这一**规范语言**。Agent 外置记忆的删除虽比权重遗忘容易审计，但已发表 agent 论文几乎无人借用这套 verifiability 语言——这是 ProvPurge 可以claim 的方法论空位。
- 姊妹篇提示：Nguyen et al., *A Survey of Machine Unlearning*（arXiv:2209.02299）流传更广，其框架含 definitions/requirements/techniques/verification 与请求粒度（item/feature/class/task/stream）；本轮 ACM DL 索引另显示该题名有 ACM TIST 条目（DOI 10.1145/3749987），**但本轮未打开 ACM 原页核实年份与卷期，故记“索引显示已发表、细节未核实”**，引用时请再开原页确认。

#### 综述 S3｜Wang et al., *Knowledge Editing for Large Language Models: A Survey*, ACM Computing Surveys, 2025
- **发表信息（索引核实充分）**：Song Wang, Yaochen Zhu, Haochen Liu, Zaiyi Zheng, Chen Chen, Jundong Li. *ACM Comput. Surv.* 57(3), Article 59, 1–37（2024 年 11 月在线/2025 年卷期），DOI 10.1145/3698590（多篇正式论文参考文献一致著录）。CCF：未核实。Scholar 被引：未获取。
- **分类体系**：按“新知识从哪里进入”分三类——**External Memorization（外置记忆：不改权重、用外部记忆/上下文影响行为）、Global Optimization（全局优化：受约束微调）、Local Modification（局部修改：定位-再编辑 locate-then-edit，如 ROME/MEMIT 一系；该综述原文类名与后续文献的 memory-based/meta-learning/locate-then-edit 三分略有出入，引用时以原综述为准）**。评测围绕 reliability、generalization、locality 等编辑性质展开。
- **对开放问题的指向**：连续/终身编辑下的**编辑间冲突、旧编辑被新编辑覆盖、locality 随编辑次数退化**被反复指出；这与 agent 记忆的“新事实覆盖旧事实”是同一问题的参数版表述。

#### 综述 S4｜Le-Khac & Truong, *A Survey on Large Language Models Unlearning: Taxonomy, Evaluations, and Future Directions*, Artificial Intelligence Review (Springer), 2025
- **发表信息（索引核实）**：Uyen N. Le-Khac, Vinh N. X. Truong. *Artificial Intelligence Review* vol. 58，2025-10-29 在线，DOI 10.1007/s10462-025-11376-7（Crossref 索引：https://api.crossref.org/works/10.1007%2FS10462-025-11376-7 ）。CCF：未核实。Scholar 被引：未获取。
- **要点**：在常规的遗忘有效性/效用/效率之外，**首次系统地把 robustness（对再学习攻击、知识泄漏的鲁棒性）列为 unlearning 目标**；指出当前方法对黑盒模型适用性有限、易受对抗攻击与知识泄漏、效率不足；未来方向为综合评测基准、鲁棒遗忘、可解释遗忘与治理。
- **对 ProvPurge 的价值**：提醒“删了还能被问出来/再学回来”不是边角料——Memora 的 adversarial 类、TOFU 的后续攻击研究都证明审稿人认这一点。ProvPurge 的实验应至少有一项**删除后诱导恢复测试**（paraphrase/multi-hop/派生摘要泄漏）。

### 2.2 只有预印本的综述（单独标注，**不要冒充已发表**）

| 综述 | 状态（本轮核实） | 分类体系（一句话） | 与 ProvPurge 最相关的论断 |
|---|---|---|---|
| Hu et al., *Memory in the Age of AI Agents: A Survey — Forms, Functions and Dynamics*, arXiv:2512.13564（2025-12，2026-01 修订） | **仅 arXiv，未核实到正式发表** | Forms: token/parametric/latent；Functions: factual/experiential/working；Dynamics: formation–evolution（consolidation & forgetting）–retrieval | 把 forgetting 明确写进 evolution 环节；前沿含 trustworthiness、记忆自动化、RL 化管理 |
| Du et al., *Rethinking Memory in LLM-based Agents: Representations, Operations, and Emerging Topics*, arXiv:2505.00675 | **仅 arXiv，未核实到正式发表** | Representations: parametric vs contextual；**六原子操作：Consolidation, Updating, Indexing, Forgetting, Retrieval, Condensation**；四话题：long-term/long-context/parametric modification/multi-source | 对 ProvPurge 最有用的一套“操作词表”：可直接把 ProvPurge 定义为 Forgetting 操作的一种可验证实现，并与 Updating/Consolidation 的边界讲清 |
| *Memory for Autonomous LLM Agents: Mechanisms, Evaluation, and Emerging Frontiers*, arXiv:2603.07670 | **仅 arXiv（2026-03），未核实到正式发表** | **Write–Manage–Read 闭环** + 三维分类：temporal scope / representational substrate / control policy；五机制族：context-resident compression、retrieval-augmented stores、reflective self-improvement、hierarchical virtual context、policy-learned management | 开放挑战原文点名：**continual consolidation、contradiction handling、learned forgetting、privacy governance、write-path filtering**——几乎就是 ProvPurge 的选题清单 |
| Wu et al., *From Human Memory to AI Memory*, arXiv:2504.15965 | **仅 arXiv，未核实到正式发表** | 以人类记忆为参照，三维（object, form, time）八象限分类 | 提供“为什么要像人一样忘”的认知叙事，但工程可操作性弱于 Du et al. |
| Liu et al., *Rethinking Machine Unlearning for Large Language Models*, arXiv:2402.08787 | **仅 arXiv/CoRR（多索引均如此著录），未核实到 ACL 等正式版本** | 提出 unlearning scope 等概念，讨论 LLM 遗忘与传统 MU 的错位 | 常被误引为已发表综述，引用时务必注 arXiv |

> 使用建议：**相关工作章节的“已发表锚点”用 S1–S4；P 系列只能以“近期预印本综述亦指出……”的口吻引用**，且在参考文献中如实标 arXiv。这是审稿人一眼能看出的学术诚信点。

### 2.3 综述共同指出的开放问题（四条主线）

1. **知识更新与冲突**：新事实到来时，旧事实应被覆盖、并存（带时间戳）还是合并？已发表系统多用启发式（Mem0 的 UPDATE 判定、A-MEM 的 evolution），综述一致认为**缺乏冲突的形式化定义与专门评测**；LongMemEval 把 knowledge-update 单列一类、Memora 把“复用已失效记忆”做成扣分项，正是对这一空缺的回应。
2. **选择性遗忘**：现有“遗忘”有三义且常被混用——(a) 容量管理式衰减（MemoryBank 遗忘曲线）、(b) 正确性驱动的失效（事实已过时，Memora/Memory-R1 的 DELETE）、(c) 权利/安全驱动的删除（unlearning，TOFU/SEUL）。综述的共识是：**(b)(c) 都缺“删什么、连带删什么、何时算删干净”的操作规范**。
3. **可验证删除**：unlearning 综述（S2、S4）把 verifiability/robustness 立为一级目标，并指出近似遗忘常被再学习/改述攻击击穿；而 agent 记忆综述（S1 及 P 系列）几乎不谈删除验证——**两个社区之间的这条裂缝就是 ProvPurge 的立题空间**：外置记忆明明有显式条目与日志，反而更应做到可审计，却无人系统做。
4. **评测错位**：基准长期以 recall 为中心（记得越多分越高），直到 LongMemEval 的 abstention、Memora 的 FAMA 才开始**惩罚“该忘的没忘”**。综述共同预言：下一代基准会把 update/forget 与 recall 并列计分。

### 2.4 综述之间的分歧（写相关工作时要主动点破）
- **分界线划在哪**：S1 按“来源×形式×操作”组织、把参数记忆（微调/编辑）纳入 agent 记忆；P 系列（Hu et al.）用 forms/functions/dynamics 更细，但同样跨参数与外置；unlearning/编辑综述（S2–S4）则只谈权重内的知识，默认“记忆=参数”。**ProvPurge 须在 §2 第一页就声明自己站在哪一侧**（建议：外置显式记忆为主，参数侧 unlearning 只作方法论借鉴）。
- **遗忘是功能还是缺陷**：认知派（Wu et al.、MemoryBank 的叙事）把遗忘当健康机制；安全派（S2/S4）把遗忘当合规义务；效率派（Mem0/A-MEM）把遗忘当容量与成本手段。三种动机对应三种指标，混写会被审稿人挑。
- **谁来决定忘**：启发式规则（时间/重要性） vs 学出来的策略（Memory-R1 的 RL、P 系列预言的 learned forgetting） vs 外部请求/来源信号（删除请求、来源撤回）。综述未有定论——ProvPurge 的“溯源信号驱动”正是第三路线，目前最少被占。

---

## 三、具体已发表论文（14 篇）

> 每篇统一格式：书目信息（含 CCF、Scholar 被引）→ 研究问题/方法/数据/结果 → 代码 → **在综述分类中的位置 + 填补的开放问题** → **写作套路（3–5 句）**。
> 总览映射表（先看此表，再读细节）：

| # | 论文（ venue/年） | 综述分类格（主） | 填补的开放问题 |
|---|---|---|---|
| 1 | Generative Agents (UIST 2023) | S1: cross-trial × textual；写-管（reflection）-读 | 长期记忆架构的“可运行原型”；未解更新/删除 |
| 2 | Reflexion (NeurIPS 2023) | S1: cross-trial × textual episodic；管=反思写入 | 经验型记忆如何跨 trial 更新；未解事实冲突 |
| 3 | MemoryBank (AAAI 2024) | S1: textual；管=forggetting-by-decay | 选择性遗忘（衰减义）第一个代表作 |
| 4 | ExpeL (AAAI 2024) | S1: cross-trial × textual experiential | 经验的 consolidation（成败轨迹→洞见） |
| 5 | LoCoMo (ACL 2024) | 评测：S1 的 evaluation 格 | 非常长期对话记忆的可测化（无 update/forget 专项） |
| 6 | MemoryLLM (ICML 2024) | S1/S3: parametric × 自更新 | 参数侧持续注入与抗遗忘（近百万次更新） |
| 7 | WISE (NeurIPS 2024) | S3: local modification 的终身版 | 编辑冲突的形式化（不可能三角）与分片消解 |
| 8 | TOFU (COLM 2024) | S2: unlearning 评测格 | “忘得像没学过”如何造数据、定指标 |
| 9 | LongMemEval (ICLR 2025) | 评测格 | **knowledge updates + abstention 首次单列** |
| 10 | A-MEM (NeurIPS 2025) | S1: textual 图式组织；管=evolution | 记忆组织随新记忆演化（邻居属性被改写） |
| 11 | Mem0 (ECAI 2025) | S1: textual (+graph)；写-管-读生产流水线 | UPDATE/DELETE 的工程化判定；效率指标进主表 |
| 12 | SEUL (AAAI 2025) | S2: model manipulation，细粒度版 | **span 级选择性遗忘** + 敏感信息专用指标 |
| 13 | Memory-R1 (ACL 2026) | P 系列: policy-learned management | **DELETE 成为可学习操作**；小数据 RL 学管记忆 |
| 14 | Memora (ACL 2026 Findings*) | 评测格 | **遗忘感知计分 FAMA**：复用失效记忆要扣分 |

### 3.1 Generative Agents: Interactive Simulacra of Human Behavior — UIST 2023
- **书目**：Joon Sung Park, Joseph C. O'Brien, Carrie J. Cai, Meredith Ringel Morris, Percy Liang, Michael S. Bernstein. *Proc. 36th ACM UIST*, 1–22, 2023，DOI 10.1145/3586183.3606763（索引核实）。CCF：**未核实**（HCI 类页面本轮未打开）。Scholar 被引：未获取。
- **问题/方法/数据/结果**：问题是 LLM 角色如何长期保持可信行为。架构三件套——**memory stream（按 recency×importance×relevance 检索）、reflection（周期性把低层记忆综合成高层结论并写回）、planning**；在 Smallville 沙盒中放 25 个 agent 自主生活两天。结果以涌现行为与可信度人工评测立论（只给一个 agent “办情人节派对”的种子，消息能经记忆链传播并促成到场）；消融表明三组件缺一都会显著降低行为可信度（定性结论，项目页/论文口径）。
- **代码（已打开核实）**：https://github.com/joonspk-research/generative_agents （Apache-2.0，本轮打开时约 22.3k stars）。
- **分类位置/空缺**：S1 的 cross-trial textual 记忆原型；它证明“写-管-读”能跑通，但管环节只有 reflection，**没有更新冲突与删除**——后续所有工作（含 ProvPurge）都可以说“自 Generative Agents 以来，记忆只进不出的问题未解”。
- **写作套路**：引言用一个具体而戏剧化的场景（一句话种子→一场派对）代替抽象动机，这是系统论文最讨巧的开场。贡献按“架构 + 沙盒环境 + 涌现发现 + 消融洞见”四点列，每点都可在正文找到对应章节。方法先给架构总图再逐组件讲检索打分公式，实验则以定性展示为主、人工评测与消融为辅，局限坦承成本与伦理/拟真边界。模仿要点：**先让读者看见现象，再解释机制**。

### 3.2 Reflexion: Language Agents with Verbal Reinforcement Learning — NeurIPS 2023
- **书目**：Noah Shinn, Federico Cassano, Edward Berman, Ashwin Gopinath, Karthik Narasimhan, Shunyu Yao. *Advances in NeurIPS 36*, 2023（proceedings 著录经索引核实）。CCF：**A（已核实，CCF 人工智能类页面）**。Scholar 被引：未获取。
- **问题/方法/数据/结果**：问题是 agent 不改权重如何从试错中学习。方法把反馈转成自然语言反思、存入 **episodic memory buffer** 供下一 trial 读取（Actor/Evaluator/Self-Reflection 三模块）。数据：ALFWorld（决策）、HotpotQA（推理）、HumanEval/LeetcodeHardGym（编程）。标志结果：**HumanEval pass@1 达 91%，超过当时 GPT-4 的 80%**（摘要，索引核实）；ALFWorld、HotpotQA 亦有大幅提升。
- **代码（已打开核实）**：https://github.com/noahshinn/reflexion （MIT；含各任务 runs 与日志）。
- **分类位置/空缺**：experiential memory 的奠基作；它更新的是“教训”而非事实，**旧反思与新证据矛盾时无处理机制**，正好衬托 ProvPurge 的事实型更新/删除。
- **写作套路**：引言先把“传统 RL 要样本、要微调”与“语言反馈便宜”对立，提出 verbal reinforcement 这一可命名的新概念。贡献列法极简：框架 + 三类任务 SOTA + 消融洞见。实验按任务分节、每节同一模板（基线→主结果→消融），分析专节讨论何时反思会失败。局限写得短而诚实（依赖评测信号质量）。模仿要点：**给机制起一个可被引用的名字，并用一句定义钉死它**。

### 3.3 MemoryBank: Enhancing Large Language Models with Long-Term Memory — AAAI 2024
- **书目**：Wanjun Zhong, Lianghong Guo, Qiqi Gao, He Ye, Yanlin Wang. *Proc. AAAI 38*: 19724–19731, 2024（OpenAIRE/出版社著录，索引核实）。CCF：**A（已核实）**。Scholar 被引：未获取。
- **问题/方法/数据/结果**：问题是 LLM 没有长期记忆、无法在持续陪伴中演化。方法三部分：记忆存储（对话/事件摘要/用户画像分层）、检索、以及受 **Ebbinghaus 遗忘曲线**启发的记忆强度更新——随时间衰减、被回忆则强化，实现“选择性保留”。载体系统 SiliconFriend（长程 AI 陪伴，心理对话数据 LoRA 微调）。评测为真实用户对话定性分析 + ChatGPT 模拟多性格用户的定量分析（仓库含 100 道人工 probing questions，中英双语）。
- **代码（已打开核实）**：https://github.com/zhongwanjun/MemoryBank-SiliconFriend （MIT；含 memory_bank 实现与评测数据）。
- **分类位置/空缺**：**“选择性遗忘”在 agent 侧的第一代表作，但遗忘=时间衰减**，不看来源、不看事实是否已被推翻、删后不可验证——这正是 ProvPurge 与之划界的第一句话。
- **写作套路**：引言从应用痛点（陪伴、咨询、秘书）切入，再借认知科学理论给机制背书，使“遗忘曲线”既有故事又有公式。贡献按“机制 + 系统 + 评测”三段列。方法按存储/检索/更新三节与架构图一一对应，实验定性先行、定量殿后。局限与未来工作放在系统体验层面。模仿要点：**用一条经典理论曲线把启发式包装成原理**，但 ProvPurge 须注意：审稿人现在会追问衰减之外的正确性证据。

### 3.4 ExpeL: LLM Agents Are Experiential Learners — AAAI 2024
- **书目**：Andrew Zhao, Daniel Huang, Quentin Xu, Matthieu Lin, Yong-Jin Liu, Gao Huang. *Proc. AAAI 38*(17): 19632–19642, 2024，DOI 10.1609/aaai.v38i17.29936（AAAI OJS 已由索引核实）。CCF：**A（已核实）**。Scholar 被引：未获取。
- **问题/方法/数据/结果**：问题同 Reflexion 但强调跨任务：离线从训练任务的成败轨迹池中**抽取自然语言 insights + 保留成功轨迹作示例**，推理时双路召回。数据：HotpotQA、ALFWorld、WebShop，并做 HotpotQA→FEVER 迁移（项目页表格：迁移设定 FEVER 成功率约 70% vs ReAct 约 63%，索引所见）。主结论是性能随经验积累单调提升且可迁移。
- **代码（索引核实，未单独打开仓库首页）**：论文脚注与 alphaXiv 均指向 https://github.com/LeapLabTHU/ExpeL ；项目页 https://andrewzh112.github.io/expel/ 。
- **分类位置/空缺**：cross-trial experiential 的 consolidation 代表；**经验只增不删**，错误洞见一旦抽出便长期污染——ProvPurge 可引它论证“派生记忆（insights）必须可溯源、可连带清除”。
- **写作套路**：引言用 Mitchell 的经典学习定义开篇，把“经验学习”锚进 ML 传统。贡献强调“无需参数更新”这一约束下做成三件事（收集、抽取、召回）。方法用一张离线/在线双阶段图讲清，实验除主表外专设迁移与定性涌现能力两节。局限讨论洞见质量与任务分布。模仿要点：**把系统拆成离线学习期与在线使用期，是经验记忆论文最清晰的叙事骨架**。

### 3.5 Evaluating Very Long-Term Conversational Memory of LLM Agents (LoCoMo) — ACL 2024
- **书目**：Adyasha Maharana, Dong-Ho Lee, Sergey Tulyakov, Mohit Bansal, Francesco Barbieri, Yuwei Fang. *Proc. ACL 2024 (Long Papers)*: 13851–13870，DOI 10.18653/v1/2024.acl-long.747（**ACL Anthology 已打开核实**：https://aclanthology.org/2024.acl-long.747/ ）。CCF：**A（已核实；ACL 主会）**。Scholar 被引：未获取。
- **问题/方法/数据/结果**：问题是既有长期对话评测最多约 5 个 session。方法为 machine–human 生成管线（persona + 时序事件图驱动 agent 对话，人工校验）。数据 LoCoMo：Anthology 摘要称每段对话平均约 600 turns、16K tokens、最多 32 sessions（注意：项目页另称平均 300 turns/9K、最多 35 sessions，两处口径不一致，引用数字时以所用版本为准并注明——公开的 10 对话版本与论文 50 对话全量亦不同）。任务含 QA（single/multi-hop、temporal、open-domain、adversarial 五类）、事件摘要、多模对话生成。结论：长上下文与 RAG 有 22–66% 改善但仍落后人类约 56%、temporal 落后约 73%（项目页口径）。
- **代码/数据（项目页已打开核实）**：https://snap-research.github.io/locomo/ （含数据与评测脚本入口）。
- **分类位置/空缺**：评测格的基石，但**没有 knowledge-update 与 forgetting 专项**——它测“记不记得”，不测“改没改对、忘没忘净”。ProvPurge 若只在 LoCoMo 上涨分，审稿人会问删除指标在哪。
- **写作套路**：基准论文范式：引言先量化旧基准的不足（≤5 sessions），再给新基准的规模对比表（Table 1 式）。贡献列为数据管线、数据集、三任务评测、系统性发现四条。实验不提新模型，而是把已有模型族（短上下文/长上下文/RAG）摆成矩阵找规律。局限坦承合成数据属性。模仿要点：**基准论文卖的是“新坐标系 + 反直觉发现”，不是 SOTA 数字**。

### 3.6 MEMORYLLM: Towards Self-Updatable Large Language Models — ICML 2024
- **书目**：Yu Wang, Yifan Gao, Xiusi Chen, Haoming Jiang, Shiyang Li, Jingfeng Yang, Qingyu Yin, Zheng Li, Xian Li, Bing Yin, Jingbo Shang, Julian McAuley. *Proc. ICML 2024*, PMLR 235: 50453–50466（**PMLR 官网索引核实**：https://proceedings.mlr.press/v235/wang24s.html ）。CCF：**A（已核实）**。Scholar 被引：未获取。
- **问题/方法/数据/结果**：问题是部署后模型静止、新知识难注入。方法在 Transformer 隐空间内嵌**固定大小的 memory pool**，自更新时只改每层一部分记忆槽，使旧知识“缓慢相位退出（slowly phase out）”。评测：模型编辑基准 + 自设长期保持评测 + 长上下文基准；摘要标志结论为**近百万次（nearly a million）记忆更新后未见性能退化**。
- **代码（已打开核实）**：https://github.com/wangyu-ustc/MemoryLLM （MIT；PMLR 摘要亦给此链接）。
- **分类位置/空缺**：parametric 自更新的代表，与 ProvPurge 的外置路线互为对照；其“旧知识缓慢退出”是**不可控遗忘**，没有按条目删除与验证——可在相关工作中用一句话完成对照。
- **写作套路**：引言把问题压缩成一个工程矛盾：全量微调贵、RAG 不进参数。贡献以架构创新为首、保持能力证据为辅。方法先讲 memory pool 的数据结构再讲自更新规则，实验刻意设计“压力测试”（近百万次更新）制造记忆点。局限隐含在固定容量假设里。模仿要点：**一个极端规模的压力实验胜过十个常规表格**——ProvPurge 可借此设计“万级条目连续 purge 后保留集不退化”实验。

### 3.7 WISE: Rethinking the Knowledge Memory for Lifelong Model Editing of LLMs — NeurIPS 2024
- **书目**：Peng Wang, Zexi Li, Ningyu Zhang, Ziwen Xu, Yunzhi Yao, Yong Jiang, Pengjun Xie, Fei Huang, Huajun Chen. *Advances in NeurIPS 37*, 2024（NeurIPS.cc 虚拟会场页索引核实：https://nips.cc/virtual/2024/poster/94912 ；卷页 53764–53797 见多篇引文）。CCF：**A（已核实）**。Scholar 被引：未获取。
- **问题/方法/数据/结果**：问题是终身编辑中更新知识“住在哪里”。先证**不可能三角**：只改长期记忆（参数）损 reliability/locality，只用工作记忆（检索激活）损 generalization。再提 WISE：**主记忆（预训练知识）+ 侧记忆（编辑知识）双参数记忆 + router 路由 + knowledge-sharding（不同编辑分片存不同子空间再无冲突合并）**。在 QA、幻觉、OOD 三类终身编辑设定、GPT/LLaMA/Mistral 多架构上优于既有编辑方法（NeurIPS 摘要口径，未在本轮核到具体数值表，不编造数字）。
- **代码（索引核实）**：论文称代码在 https://github.com/zjunlp/EasyEdit （WISE 作为其中方法实现；本轮经搜索索引与 DeepWiki 索引确认该仓库含 WISE 实现，未单独打开仓库首页）。
- **分类位置/空缺**：S3 的终身编辑格；它把“冲突”从口号变成了**可画成三角形的三指标权衡**。ProvPurge 应学这一手：把删除问题画成自己的权衡图（如 削除彻底性 × 保留效用 × 派生一致性）。
- **写作套路**：教科书级的“先立 impossibility、再给绕行方案”结构：引言提出根本问题（知识住在哪），第 2 节用实验把三角坐实，方法才登场且每个组件对应三角的一角。贡献第一条不是方法而是发现（impossible triangle）。实验按三性质分表、跨架构验证。局限讨论 router 与分片的规模上限。模仿要点：**命名一个“不可能三角”式张力，等于给整篇论文装上记忆锚点**。

### 3.8 TOFU: A Task of Fictitious Unlearning for LLMs — COLM 2024
- **书目**：Pratyush Maini, Zhili Feng, Avi Schwarzschild, Zachary C. Lipton, J. Zico Kolter. *First Conference on Language Modeling (COLM)*, 2024（作者主页与多索引一致著录；arXiv:2401.06121）。CCF：**未核实**（COLM 为 2024 年新会议，本轮未在 CCF 目录中核到）。Scholar 被引：未获取。
- **问题/方法/数据/结果**：问题是 unlearning 号称“像没学过”，却无干净办法检验。方法即基准设计：**200 个虚构作者 × 每人 20 组 QA**，先微调让模型真学会，再划 forget set（forget01/05/10）要求遗忘，配 retain set 与成套指标（forget quality、model utility 等）并与“只在 retain 上重训练的 oracle”比分布。结论（摘要）：**所测基线无一实现有效遗忘**。
- **代码（已打开核实，现行仓库）**：https://github.com/locuslab/open-unlearning （仓库自述：已替代原 locuslab/tofu，统一 TOFU/MUSE/WMDP 与 13+ 遗忘方法）。
- **分类位置/空缺**：S2 评测格的 LLM 版基石。它的虚构数据诀窍——**用合成事实避开“真数据到底学没学过”的不可知**——可直接移植到 ProvPurge：用合成人物/事实注入 agent 记忆，再要求按来源 purge，天然有 ground truth。
- **写作套路**：基准论文的另一种范式：引言先攻击评测前提（无法知道预训练里有什么），再用“虚构”一招拆掉前提。贡献全在数据集与指标设计，基线结果故意“全军覆没”以证明问题难、基准有区分度。实验主表是多方法×多指标矩阵，分析节专谈指标会互相矛盾。局限承认虚构分布与真实知识的差距。模仿要点：**当真实场景不可测时，设计一个可控的合成孪生场景，并明说它与真实的距离**。

### 3.9 LongMemEval: Benchmarking Chat Assistants on Long-Term Interactive Memory — ICLR 2025
- **书目**：Di Wu, Hongwei Wang, Wenhao Yu, Yuwei Zhang, Kai-Wei Chang, Dong Yu. *ICLR 2025*（OpenReview 页眉 “Published as a conference paper at ICLR 2025”，索引核实）。CCF：**未核实**（CCF 人工智能 A 名单未见 ICLR，见 §1.2）。Scholar 被引：未获取。
- **问题/方法/数据/结果**：问题是长期记忆缺统一考卷。基准 **500 道人工精编题**，嵌在可伸缩的 timestamped 聊天史中，五能力：**information extraction、multi-session reasoning、temporal reasoning、knowledge updates、abstention**；两档规模 LongMemEval-S（约 115K tokens/题）与 -M（500 sessions、约 1.5M tokens）。标志结论：**商用助手与长上下文 LLM 在持续交互下准确率下降约 30%**（摘要）；论文另把记忆系统拆成 indexing/retrieval/reading 等阶段并给出 session 分解、fact-augmented key expansion、time-aware query expansion 三项优化，显著提升 recall 与 QA。
- **代码（已打开核实）**：https://github.com/xiaowu0162/LongMemEval （MIT；数据在 HuggingFace，仓库含评测与记忆系统实验代码）。
- **分类位置/空缺**：评测格中**第一个把 knowledge updates 与 abstention 立为一级能力**的工作——ProvPurge 的主实验应至少含 LongMemEval 的 knowledge-update 子集 + 自建 purge 子集，否则选题与评测会对不上。
- **写作套路**：引言用“能力清单缺项”立论（旧基准不测更新与拒答），贡献是基准 + 统一框架 + 三项可复用优化的三层结构。方法节先形式化记忆流水线阶段，再逐项优化对应一阶段——这种“阶段—优化”对照表极易被审稿人接受。主实验先给难度证据（-30%），再给优化增益，消融按阶段拆。局限谈合成史与评测成本。模仿要点：**把系统拆成公认阶段、每阶段只改一处，是增量创新最稳的包装**。

### 3.10 A-MEM: Agentic Memory for LLM Agents — NeurIPS 2025
- **书目**：Wujiang Xu, Zujie Liang, Kai Mei, Hang Gao, Juntao Tan, Yongfeng Zhang. *Advances in NeurIPS 38*, 2025（NeurIPS.cc 2025 poster 页索引核实：https://neurips.cc/virtual/2025/loc/san-diego/poster/119020 ；arXiv:2502.12110 的 Comments 亦标 NeurIPS 2025）。CCF：**A（已核实）**。Scholar 被引：未获取。
- **问题/方法/数据/结果**：问题是既有记忆系统只有存取、组织僵化。方法借 **Zettelkasten 卡片盒**：新记忆生成带上下文描述/关键词/标签的结构化 note，与历史 note 建链；新 note 加入时可触发**memory evolution——改写邻居 note 的上下文与属性**，使记忆网持续自组织。评测主战场 LoCoMo，**六个基础模型**上优于既有 SOTA 基线（NeurIPS 摘要口径；二手总结称 multi-hop 大幅提升且记忆操作 token 大降，本轮未在官方页核到确切倍数，故正文不写具体倍数）。
- **代码（已打开核实）**：https://github.com/WujiangXu/A-mem （MIT；仓库自述为 NeurIPS 2025 论文代码，另有官方系统版 A-mem-sys 入口）。
- **分类位置/空缺**：S1 的 management 格明星——evolution 本质是“新记忆改写旧记忆”，**但改写无版本、无溯源、不可回滚**：一条坏记忆可以污染一串邻居。ProvPurge 的溯源链正是对 evolution 的安全补丁，这是极好的相关工作切入句。
- **写作套路**：引言先贬“只有 storage/retrieval 的仓库式记忆”，再用一个跨学科隐喻（Zettelkasten）撑起方法身份。贡献按 note construction、link generation、memory evolution、retrieval 四步列，与方法四小节同构。实验以 LoCoMo 分题型主表为核心，配多骨干与 token 效率分析。局限少谈错误传播——这正是后来者（包括 ProvPurge）的机会。模仿要点：**借一个成熟的人类实践作隐喻时，必须落到可执行的数据结构与触发规则，否则只是修辞**。

### 3.11 Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory — ECAI 2025
- **书目**：Prateek Chhikara, Dev Khant, Saket Aryan, Taranjeet Singh, Deshraj Yadav. *ECAI 2025 — 28th European Conference on AI*, Frontiers in Artificial Intelligence and Applications vol. 413: 2993–3000, IOS Press，DOI 10.3233/FAIA251160（多篇正式论文引文一致著录，索引核实）。CCF：**B（已核实，CCF 人工智能类 B 名单含 ECAI）**。Scholar 被引：未获取。
- **问题/方法/数据/结果**：问题是固定上下文窗口下跨 session 一致性与生产可用性。方法两阶段：**extraction（从对话抽显著事实）+ update（用工具调用对已有记忆做 ADD/UPDATE/DELETE/NOOP 式合并，处理冗余与冲突）**；变体 Mem0g 把记忆组织成图。评测仅 LoCoMo，对六类基线（记忆系统、RAG 多粒度、全上下文、开源/专有记忆等）：摘要标志结果——**Mem0 的 LLM-as-a-Judge 相对 OpenAI 记忆高 26%，Mem0g 总分再高约 2%；p95 延迟比全上下文低 91%、省 token 90%+**。
- **代码（已打开核实，注意版本）**：https://github.com/mem0ai/mem0 （Apache-2.0）。**注意**：现仓库是持续演进的产品代码（2026 年已换新算法、自述改为 ADD-only 单遍抽取，与 ECAI 论文的 update 流水线并不等同），引用“论文方法”时勿以现仓库 README 为据。
- **评测争议（引用时须知）**：Mem0 与 Zep 曾公开互相质疑对方 LoCoMo 口径（索引所见的复盘文章有详细时间线，本轮未逐一打开双方原文核实，记**未核实细节**）——教训是：**凡报 LoCoMo 分数，必须写清类别口径、judge 模型与检索预算**，ProvPurge 尤其如此。
- **分类位置/空缺**：S1 写-管-读的生产化标杆，DELETE 在它这里是**冲突合并的副产物**，不是面向“来源失效/删除请求”的可验证操作，也无溯源链。
- **写作套路**：标题直给定位词（Production-Ready、Scalable），引言把学术问题翻译成部署三难（一致性、延迟、成本）。贡献列法是系统 + 图变体 + 全面对比 + 效率证据。实验主表按问题类别分列、第二主表专测延迟与 token——**把效率升为主结果而非附录**是它区别于学院派的关键。局限谈得少、偏产品叙事。模仿要点：ProvPurge 若主打可用性，必须有延迟/存储/调用成本表；若主打正确性，则别学它淡化局限。

### 3.12 Selective Forgetting: Advancing Machine Unlearning Techniques and Evaluation in Language Models (SEUL) — AAAI 2025
- **书目**：Lingzhi Wang, Xingshan Zeng, Jinsong Guo, Kam-Fai Wong, Georg Gottlob. *Proc. AAAI 39*: 843–851, 2025，DOI 10.1609/aaai.v39i1.32068（Crossref/Mendeley 索引核实；arXiv:2402.05813 注 Accepted to AAAI 2025）。CCF：**A（已核实）**。Scholar 被引：未获取。
- **问题/方法/数据/结果**：问题是整体反向训练的遗忘会伤生成能力、且“忘整条样本”粒度太粗。方法 **SEUL**：只对**敏感 span** 施加遗忘目标（而非整句/整样本），配在线（语言概率打分）与离线（两阶段 LLM 标注）两种自动敏感 span 标注法；新指标 **S-EL（sensitive extraction likelihood）与 S-MA（sensitive memorization accuracy）** 专测敏感信息忘没忘。结论（摘要口径）：相比全量反转目标，细粒度遗忘对模型能力（尤其生成）的负面影响更小。
- **代码**：**未确认**（本轮检索未找到与论文对应的官方仓库，不编造链接）。
- **分类位置/空缺**：S2 的 model manipulation 格，但把粒度从样本推进到 span——**“选择性”的粒度竞赛**正是 ProvPurge 要接的棒：agent 侧的对应粒度应是“事实条目 + 其派生条目”，而非整段对话。
- **写作套路**：引言先立粒度问题（忘得太粗→伤能力），贡献三件套严格对应问题三面：方法（span 级）、指标（S-EL/S-MA）、标注管线（在线/离线）。方法节先给标注再给目标函数，实验围绕“遗忘效果—能力损伤”权衡曲线展开。局限在标注可靠性与敏感定义。模仿要点：**提出新粒度时，必须同时提出测新粒度的指标，否则贡献不闭环**。

### 3.13 Memory-R1: Enhancing Large Language Model Agents to Manage and Utilize Memories via Reinforcement Learning — ACL 2026（Long Papers）
- **书目**：Sikuan Yan, Xiufeng Yang, Zuchao Huang, Ercong Nie, Zifeng Ding, Zonggen Li, Xiaowen Ma, Jinhe Bi, Kristian Kersting, Jeff Z. Pan, Hinrich Schütze, Volker Tresp, Yunpu Ma. *Proc. ACL 2026 (Vol. 1: Long Papers)*: 12805–12825，DOI 10.18653/v1/2026.acl-long.583（**ACL Anthology 已打开核实**：https://aclanthology.org/2026.acl-long.583/ ）。CCF：**A（已核实；ACL 主会 Long Paper）**。Scholar 被引：未获取（2026 年新作）。
- **问题/方法/数据/结果**：问题是记忆管线多为静态启发式、无学习到的“存/改/删”决策。方法双 agent：**Memory Manager 学结构化操作 {ADD, UPDATE, DELETE, NOOP}，Answer Agent 学在检索结果上做 memory distillation 后再答**，两者以结果奖励用 PPO/GRPO 微调。数据极省：**仅 152 个训练 QA**。结果：三基准（LoCoMo、MSC、LongMemEval）与 3B–14B 多规模上胜强基线；LoCoMo 上以 LLaMA-3.1-8B 为骨干，GRPO 版相对最强基线 Mem0 的 **F1 +28%、BLEU-1 +34%、LLM-as-a-Judge +30%**（论文文本，索引核实）。
- **代码**：仓库 yansikuan/memory-r1 存在（DeepWiki 索引可见其 README 与结果表），但索引快照曾标注 “Code coming soon/筹备中”，**完整代码是否已释放记未确认**，勿写“已开源”。
- **分类位置/空缺**：P 系列五机制族中 policy-learned management 的代表作。它的 DELETE 由**下游答对与否的奖励**驱动——删错了只要还能答对就不受罚，**没有“该删的来源条目必须删”的硬约束与审计**。ProvPurge 与它划界一句话：奖励驱动的遗忘 ≠ 溯源驱动的可验证清除。
- **写作套路**：引言把启发式管线批为“static and heuristic-driven”，随即抛出“RL is the missing ingredient”式论点句。贡献强调数据效率（152 例）这一反直觉卖点，并把它放进摘要与主结果。方法按双 agent 分工写、训练细节后置，实验先主基准定锤、再跨基准跨规模证泛化、消融拆 PPO/GRPO 与 distillation。局限谈奖励信号的粗糙。模仿要点：**把“少数据”做成卖点需要同时给跨规模泛化证据，否则会被当作过拟合单一基准**。

### 3.14 From Recall to Forgetting: Benchmarking Long-Term Memory for Personalized Agents (Memora) — ACL 2026 Findings ★单独标注
- **★标注**：Md Nayem Uddin, Kumar Shubham, Eduardo Blanco, Chitta Baral, Gengyu Wang. *Findings of ACL 2026*: 26814–26841，DOI 10.18653/v1/2026.findings-acl.1337（**ACL Anthology 已打开核实**：https://aclanthology.org/2026.findings-acl.1337/ ）。**Findings 与 ACL 主会 Long Papers 是不同卷；CCF 目录的计数口径与不少学校/机构的评价体系中 Findings 常不按主会长文计，本报告按用户要求单独标注，引用与考核时请按所在单位口径处理。** Scholar 被引：未获取（2026 年新作）。
- **问题/方法/数据/结果**：问题直指本报告主题：既有基准把长期记忆框成“事实检索”，看不见记忆巩固与频繁更新。基准 **Memora**：跨周到月的用户对话，三任务 **remembering / reasoning / recommending**，配自动 memory-grounding 检查与人工评测；新指标 **FAMA (Forgetting-Aware Memory Accuracy)**——**复用已过时/已失效的记忆要扣分**。评测 4 个 LLM + 6 个记忆 agent，结论：系统普遍**频繁复用 invalid memories、无法调和演化中的记忆**，记忆 agent 仅边际改善。仓库 Table 3 复现数据示例（FAMA，weekly→quarterly）：A-MEM 的 remembering 由 71.82 跌至 40.78，裸 LLM 的 remembering 普遍仅约 17–27，显示“记得住”与“忘得对”随时间双双崩坏。
- **代码（已打开核实）**：https://github.com/geniesinc/Memora （Apache-2.0；含 weekly/monthly/quarterly 数据、双轨评测与 FAMA 实现；10 个 persona，每 persona-period 15–30 题，每题含 memory_presence 与 forgetting_absence 两类子问）。
- **分类位置/空缺**：评测格中**与 ProvPurge 最近的一篇**：它把“该忘未忘”做成了可计分的失败，但它只评测、不提供清除机制，更不处理派生记忆与溯源——**ProvPurge 的方法贡献恰好补在 Memora 的评测贡献之後，二者是天然的“问题—方法”配对引用**。
- **写作套路**：标题用 From X to Y 的转折句直接宣告范式移动（Recall→Forgetting），引言先证明旧框定（retrieval-only）系统性漏测。贡献是基准 + 指标（FAMA）+ 令人不安的实证发现三件套。实验矩阵是“模型族 × 时间跨度（周/月/季）”，用时间维度制造衰减曲线而非单点分数。局限坦承合成对话、judge 依赖与 persona 覆盖窄。模仿要点：**给“坏行为”起一个可计算的名字（FAMA 式），比在讨论里抱怨它有力十倍**——ProvPurge 应有自己的 Purge-Verification 指标名。

### 3.15 候选排除与降级说明（勿混入已发表清单）
- **MemGPT: Towards LLMs as Operating Systems（Packer et al.）**：本轮在多篇正式论文的参考文献、OpenReview 引用与作者著录中均只见 **arXiv:2310.08560 / CoRR**，**未核实到正式主会或期刊版本 → 排除出正选**。其“OS 式分级记忆/分页”思想影响极大，相关工作可引，但须注 arXiv。后续系统 Letta 与之关系属产品演进，不改变论文发表状态。
- **Zep: A Temporal Knowledge Graph Architecture for Agent Memory（Rasmussen et al.）**：仅见 **arXiv:2501.13956 / CoRR** 著录，**未核实到正式发表 → 排除**。其双时态（valid time / transaction time）图与“旧边失效而非删除”的做法对 ProvPurge 的时间/冲突建模很有启发，引用时注 arXiv。
- **Who's Harry Potter? Approximate Unlearning in LLMs（Eldan & Russinovich）**：原稿 arXiv:2310.02238；本轮另见 ICLR 2024 Workshop on Secure and Trustworthy LLMs 的相关 OpenReview 记录，但 **workshop 不计主会长文、且该记录与原稿的版本对应未完全核实 → 不入正选**。其三步法（强化模型定位目标 token → 生成 generic 替代标签 → 微调擦除，约 1 GPU 小时擦除 Llama2-7B 的 Harry Potter 内容且通用基准近乎无损，arXiv 摘要口径）可在相关工作中作方法源头一笔带过并注 arXiv。
- **Larimar（Das et al., ICML 2024）**：本轮仅在一篇 ICLR 2026 论文的转述中见到其“selective recall and forgetting、可控知识更新”的描述，**未精读原文，故不入正选**；但方向高度贴题，建议研究生后续补读核实（线索：ICML 2024，情景记忆式快速更新）。

---

## 四、横向对比速查表

| 论文 | 记忆载体 | 更新机制 | 遗忘/删除机制 | 删除可验证？ | 主基准 |
|---|---|---|---|---|---|
| Generative Agents | 文本 stream | reflection 写回 | 无 | — | 自建沙盒/人工评测 |
| Reflexion | 文本 episodic buffer | 反思追加 | 无（buffer 截断） | — | ALFWorld/HotpotQA/HumanEval |
| MemoryBank | 文本分层 | 摘要+画像演化 | **时间衰减（遗忘曲线）** | 否 | 自建陪伴对话/probing |
| ExpeL | 文本 insights+轨迹 | 离线抽取洞见 | 无 | — | HotpotQA/ALFWorld/WebShop |
| MemoryLLM | **参数内 latent pool** | 自更新改槽 | 旧知识缓慢相位退出 | 否 | 编辑基准/LongBench/自设 |
| WISE | **参数（主+侧记忆）** | 分片编辑+路由 | 以新编辑覆盖（分片消解冲突） | 否（以 locality 间接测） | ZsRE 等编辑设定 |
| TOFU | 参数（评测对象） | — | 评测各类 unlearning | **是（vs 重训练 oracle）** | TOFU forget/retain |
| A-MEM | 文本 note 图 | **evolution 改写邻居** | 无显式删除 | 否 | LoCoMo |
| Mem0 | 文本（+图） | LLM 判定 UPDATE | 合并时 DELETE（副产物） | 否 | LoCoMo |
| SEUL | 参数 | — | **span 级选择性遗忘** | 以 S-EL/S-MA 测 | 敏感信息自建评测 |
| Memory-R1 | 文本 bank | **RL 学 UPDATE** | **RL 学 DELETE** | 否（只看下游奖励） | LoCoMo/MSC/LongMemEval |
| Memora | 评测对象 | 评测更新调和 | **评测该忘未忘（FAMA 扣分）** | 评测层面的“验证” | Memora（周/月/季） |
| **ProvPurge（拟）** | 文本/图/向量（外置） | 溯源触发的失效传播 | **溯源链连带清除（含派生）** | **是（审计+恢复攻击测试）** | LongMemEval-update + Memora + 自建 purge 集 |

---

## 五、这些已发表论文的共同写法总结

### 5.1 通用结构（章节与图表的“标准件”）
- **章节**：1 引言（痛点→缺口→方法一句话→贡献列表）→ 2 相关工作（按机制族分组，不按年份罗列）→ 3 方法（先总图、后按流水线阶段/操作分小节，常配形式化定义）→ 4 实验（设置→主结果→消融→分析/案例）→ 5 局限（或并入讨论）→ 6 结论。基准论文把第 3 节换成“数据构建管线 + 统计 + 评测协议”。
- **图**：Fig.1 几乎必是**架构/管线总图**（左旧右新或上下对比，A-MEM、Mem0、Generative Agents 皆然）；Fig.2 常给一个**贯穿全文的例子**（一段对话/一条记忆的生老病死）。WISE 的“三角图”、LongMemEval 的“能力雷达/阶段图”说明：**把权衡画成几何图形**是高水平论文的标志动作。
- **表**：Table 1 常是**与旧基准/旧系统的定位对比表**（维度打勾式）；主结果表是“方法 × 题型/能力分列”（LoCoMo 五类、LongMemEval 五能力、Memora 三任务×三跨度），**从不只报一个总分**；其后必有一张**消融表**（逐组件开关）和一张**效率表**（token/延迟/存储，Mem0 把它升为主表）。
- **实验编排顺序**：先证问题难（基线崩坏的数字，如 -30%、落后人类 56%）→ 再证方法有效（主表）→ 再证为什么有效（消融）→ 再证边界（跨规模/跨骨干/案例与失败分析）。局限一节普遍短、具体、指向数据合成性与 judge 依赖，不写套话。

### 5.2 审稿人显然买账的叙事（从贡献表述与实验设计归纳）
1. **给现象/机制起名字**：verbal reinforcement、impossible triangle、FAMA、memory evolution、knowledge updates——可命名的贡献才可被引用。
2. **先立“不可能/系统性失败”，再给绕行**：WISE 的三角、TOFU 的“基线全灭”、LongMemEval 的 -30%、Memora 的“普遍复用失效记忆”，都是先把旧世界判死刑。
3. **新指标与新方法互相成就**：测不出的问题等于不存在——SEUL 配 S-EL/S-MA、Memora 配 FAMA、LongMemEval 配五能力分列。**只有方法没有指标的遗忘论文很难过审**。
4. **极端/压力证据制造记忆点**：MemoryLLM 的近百万次更新、LongMemEval-M 的 1.5M tokens、Memora 的季度跨度。
5. **效率进主表**：token、延迟、存储至少报一项，纯精度叙事在 agent 方向已不够。
6. **合成数据要明说可控性收益**：TOFU 的虚构作者、LoCoMo/LongMemEval 的机器-人工管线，都把“合成”写成“有 ground truth、可复现”的优点，同时在局限里承认分布差距。

### 5.3 与 ProvPurge 最接近的工作与引用划界（逐篇一句）
- **Memora**：它**评测**该忘未忘（FAMA），ProvPurge **执行并验证**清除；引用时写 “Memora establishes that stale-memory reuse is measurable and prevalent; we provide a mechanism…”。
- **Memory-R1**：它的 DELETE 由 RL 奖励习得、无来源约束；ProvPurge 的 purge 由**溯源失效事件**触发且可审计。划界词：reward-driven vs provenance-driven。
- **Mem0**：它的 DELETE 是冲突合并的内部动作、无连带派生处理与对外验证；ProvPurge 处理**派生记忆图（摘要/反思/图边/索引）的连带清除**。
- **MemoryBank**：遗忘=时间衰减；ProvPurge 的遗忘=事件驱动（来源撤回/被新事实推翻/删除请求），与时间无关或仅弱相关。
- **A-MEM**：evolution 会改写邻居却无溯源与回滚；ProvPurge 可定位为“给 evolution 加溯源与可逆性”。
- **SEUL/TOFU（参数侧）**：对象是权重、粒度是 span/样本、验证靠与重训练 oracle 比；ProvPurge 对象是外置条目、验证靠**清除日志 + 探针问答 + 派生一致性检查**，借其指标思想而不借其设定。
- **WISE**：借其“冲突权衡可视化”与分片思想，但 WISE 解决参数编辑冲突，不解决外置记忆的来源失效传播。

---

## 六、对 ProvPurge 的写作启示与推荐大纲

### 6.1 如何直接借综述框架（最重要的一条建议）
- **相关工作章节就按 S1（TOIS）的“来源 × 形式 × 操作”三轴 + Du et al.（预印本，须标注）的六原子操作词表来搭**：先用 S1 的写–管–读定位全领域，再在“管”之下展开 Updating / Forgetting / Consolidation 三个小节，每小节末尾用一句话点出“已有工作缺溯源与验证”，自然滑向 ProvPurge。这样写，审稿人会觉得你的定位是**从综述坐标系里长出来的**，而不是自说自话。
- **问题定义借 S2（CSUR unlearning）的 desiderata 语言**：把 ProvPurge 的目标写成外置记忆版三条件——Purge Completeness（目标及其派生被清除）、Retention Utility（保留集问答不退化）、Purge Verifiability（第三方可用探针与日志验证清除）。再借 S4 的 robustness 补第四条件：清除后经改述/多跳/摘要再生等诱导仍不泄漏。
- **术语纪律**：全文只用一套词——把 decay（衰减）、invalidation（失效）、deletion/purge（清除）、unlearning（参数遗忘）四词分开定义并在 §2 给对照表；已发表论文最常见的被批点就是四词混用。

### 6.2 推荐大纲（到二级标题，标注写什么与图表）

**Title 建议方向**：*ProvPurge: Provenance-Aware Selective Purging of Stale and Revoked Memories in LLM Agents*（标题含对象、机制、动作三要素，仿 Memora/Memory-R1 的命名法。）

#### 1 Introduction
- 1.1 场景开场：一个具体案例——用户搬家/改偏好/撤回授权后，agent 仍用旧地址、旧偏好、已撤回信息作答（仿 Generative Agents 的场景开场、Memora 的失效复用证据）。**放 Fig.1：一条记忆从写入→派生（摘要/反思/图边）→来源失效→旧系统继续误用 vs ProvPurge 连带清除的对照图。**
- 1.2 缺口陈述：更新与遗忘在综述中的位置（引 S1、LongMemEval、Memora），现有 DELETE 的三不足（无溯源、无连带、无验证）。
- 1.3 方法一句话与贡献列表（4 条：溯源记忆图与失效传播规则、连带清除算法、可验证清除协议与指标、在 update/forget 基准上的证据）。贡献写法仿 WISE：第一条给“问题形式化”，方法放第二条。

#### 2 Related Work
- 2.1 Agent long-term memory systems（按写–管–读组织：Generative Agents、MemoryBank、A-MEM、Mem0、Memory-R1；MemGPT/Zep 注 arXiv 一笔带过）
- 2.2 Experience and reflective memory（Reflexion、ExpeL；点明其派生洞见同样需要可清除性）
- 2.3 Knowledge updating and model editing（MemoryLLM、WISE + S3 综述；参数侧对照）
- 2.4 Machine unlearning and selective forgetting（TOFU、SEUL + S2/S4 综述；借 desiderata 与验证思想，声明设定差异）
- 2.5 Benchmarks for update and forgetting（LoCoMo、LongMemEval、Memora；说明各自测什么、不测什么）**放 Table 1：相关工作 × {载体/更新/遗忘/连带派生/可验证} 打勾对比表（仿 §4）。**

#### 3 Problem Formulation
- 3.1 Memory with provenance（定义记忆条目、来源、派生关系与溯源 DAG）
- 3.2 Invalidation events and purge requests（来源撤回、事实被取代、用户删除请求、过期策略四类触发）
- 3.3 Desiderata（Completeness / Retention Utility / Verifiability / Robustness，四条件形式化，借 S2/S4 语言）**放 Fig.2：溯源 DAG 与一次 purge 的传播示例（贯穿例）。**

#### 4 Method: ProvPurge
- 4.1 Provenance-tracked memory construction（写入时记录来源与派生边，与 A-MEM evolution 的对照）
- 4.2 Invalidation propagation（失效如何在 DAG 上判定受影响集：直接条目、摘要、反思、图边、向量索引）
- 4.3 Purge execution（硬删除/墓碑标记/索引重建的策略与一致性保证）
- 4.4 Verification protocol（探针集生成、派生一致性检查、审计日志）**放 Fig.3：系统总架构图；Table 2：与 Mem0 DELETE、Memory-R1 DELETE 的动作语义对照。**

#### 5 Experiments
- 5.1 Setup（基准：LongMemEval knowledge-update 子集、Memora、LoCoMo 基础分；自建 ProvPurge-Bench：仿 TOFU 的合成注入—失效—探针协议；骨干、judge、检索预算全部写明，吸取 Mem0/Zep 争议教训）
- 5.2 Main results（主表按能力分列：更新后正确率、该忘未忘率/自建 FAMA 式指标、保留集退化）**放 Table 3（主表）。**
- 5.3 Verification and robustness（改述/多跳/摘要再生诱导恢复测试，仿 S4 的 robustness 主张）**放 Table 4。**
- 5.4 Ablations（去溯源连带、去验证、硬删 vs 墓碑、传播深度）**放 Table 5。**
- 5.5 Efficiency and scaling（purge 延迟、索引重建成本、万级条目压力测试，仿 MemoryLLM/Mem0）**放 Fig.4：规模—成本曲线。**
- 5.6 Case studies and failure analysis（2–3 个完整案例：成功连带清除、过度清除、隐晦派生泄漏）

#### 6 Limitations and Ethical Considerations
- （合成评测与真实分布差距、LLM judge 依赖、溯源记录本身的隐私成本、过度清除风险；写具体，不写套话。）

#### 7 Conclusion
- （回扣三 desiderata 与一句话展望：把 learned forgetting 与溯源硬约束结合。）

> 注记：上述大纲基于任务给出的方向（知识更新与选择性遗忘）与 “ProvPurge” 名称推断其“溯源清除”内核；**若 ProvPurge 的实际定义与此不同（例如 Prov 指 provable 而非 provenance），§3 与 §4 的术语需相应替换，但章节骨架与图表配置可保留**。这一点请研究生在动笔前与导师确认。

---

## Could not verify / 未能核实清单
1. **全部论文的 Google Scholar 被引量：未获取**（Scholar 直连受限，`site:scholar.google.com` 索引检索无结果；未以其他平台数字替代）。
2. CCF：UIST、ACM Computing Surveys、Artificial Intelligence Review、COLM 的类别**未核实**；ICLR 在实际打开的 CCF 人工智能类 A 名单中未见，记未核实（各校口径可能不同）。
3. Nguyen et al. unlearning 综述的 TIST 版本（DOI 10.1145/3749987）仅见 ACM DL 索引，年份/卷期未打开原页核实。
4. ExpeL、WISE 的代码仓库为论文/索引指向（LeapLabTHU/ExpeL、zjunlp/EasyEdit），本轮未单独打开仓库首页；Memory-R1 仓库的完整代码释放状态未确认；SEUL 代码未找到，记未确认。
5. A-MEM、Mem0 的部分二手流传数字（倍数类）未在官方页核到，正文只用官方摘要口径；Mem0/Zep 的 LoCoMo 争议细节未逐一核实双方原文。
6. LoCoMo 的规模数字在 Anthology 摘要与项目页之间口径不一致，正文已双注。
7. Who's Harry Potter 的 workshop 记录与原稿的版本对应关系未完全核实，故只作排除说明。

## Sources（主要来源，均为 2026-10-29 实际打开或经网页索引读取）
- CCF 人工智能类推荐目录（已打开）：https://www.ccf.org.cn/Academic_Evaluation/AI/
- ACL Anthology — LoCoMo（已打开）：https://aclanthology.org/2024.acl-long.747/
- ACL Anthology — Memory-R1（已打开）：https://aclanthology.org/2026.acl-long.583/
- ACL Anthology — Memora/Findings（已打开）：https://aclanthology.org/2026.findings-acl.1337/
- LoCoMo 项目页（已打开）：https://snap-research.github.io/locomo/
- 代码仓库（已打开）：https://github.com/joonspk-research/generative_agents ；https://github.com/noahshinn/reflexion ；https://github.com/zhongwanjun/MemoryBank-SiliconFriend ；https://github.com/xiaowu0162/LongMemEval ；https://github.com/WujiangXu/A-mem ；https://github.com/mem0ai/mem0 ；https://github.com/wangyu-ustc/MemoryLLM ；https://github.com/locuslab/open-unlearning ；https://github.com/geniesinc/Memora
- 会议/出版社页（索引读取）：NeurIPS A-MEM https://neurips.cc/virtual/2025/loc/san-diego/poster/119020 ；NeurIPS WISE https://nips.cc/virtual/2024/poster/94912 ；PMLR MemoryLLM https://proceedings.mlr.press/v235/wang24s.html ；AAAI ExpeL https://ojs.aaai.org/index.php/AAAI/article/view/29936 ；OpenReview LongMemEval（ICLR 2025 页眉与摘要，经索引读取）
- 综述著录（索引读取）：TOIS 2025, 43(6):1–47, DOI 10.1145/3748302（作者主页 https://zeyu-zhang.cn/publication.html ）；Xu et al. CSUR 56(1) Art.9 PDF 页眉 https://par.nsf.gov/servlets/purl/10520867 ；Wang et al. CSUR 57(3), DOI 10.1145/3698590；Le-Khac & Truong, DOI 10.1007/s10462-025-11376-7（Crossref https://api.crossref.org/works/10.1007%2FS10462-025-11376-7 ）；Mem0 ECAI 2025, FAIA 413:2993–3000, DOI 10.3233/FAIA251160；SEUL AAAI 2025, DOI 10.1609/aaai.v39i1.32068
- 预印本综述（索引读取，状态如 §2.2）：arXiv:2512.13564；arXiv:2505.00675；arXiv:2603.07670；arXiv:2504.15965；arXiv:2402.08787；排除项 arXiv:2310.08560（MemGPT）、arXiv:2501.13956（Zep）、arXiv:2310.02238（Who's Harry Potter）
