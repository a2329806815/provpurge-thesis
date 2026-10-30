# LLM Agent 长期记忆的知识更新与选择性遗忘 — 深度调研报告

> 面向：一名人工智能硕士研究生，目标是发表一篇论文；约束为周期约 3–6 个月、无大模型训练算力、仅 API + 开源中小模型。
> 调研时点：2026-10-29。阅读方式说明：下文标【已证实】的结论来自本次实际打开的论文/项目页面（URL 随附）；标【推测/建议】的是基于已证实事实的判断与设计提案，不是文献结论；标【二手】的是经综述/博客转述、未打开原文核对的数字，引用进论文前必须回原文核对。
>
> **一句话总判断（先说最重要的）：这个方向在 2026 年 10 月已经非常拥挤。你原计划里看起来最自然的三个卖点——“带版本/来源的结构化事实库”“墓碑标记 + 级联删除”“冲突检测后保留无关记忆”——在 2026 年 5–10 月已经分别被 STALE/CUPMem、PACMI、PolyGraph、ChronoMem、Agentic Unlearning/SBU、BEMULA 等工作正面做过（详见第 3 节撞车表）。但仍有一个真实且可做的空位：在 BEMULA（2026-10-13 刚发布）上，所有现成系统的遗忘请求端到端通过率 ≤1.5%——即“测遗忘的基准有了，能在对话式个人记忆上真正删干净、且不误删、且能防改写/推断抽取的方法还没有”。把论文定位从“我提出级联删除这个想法”改成“我提出一个写时句级来源 + 确定性级联清除的方法，并在 BEMULA + LongMemEval 知识更新子集 + STALE 三套协议上首次同时过关”，是本报告的首选路线。**

---

## Summary

- **现状**：记忆系统已从“存对话”演进到“抽取事实并做 ADD/UPDATE/DELETE 裁决”（Mem0）、“双时态知识图谱边失效”（Zep/Graphiti）、“RL 学习记忆操作”（Memory-R1）。评测已从只测召回（LoCoMo、LongMemEval）演进到测失效复用（Memora/FAMA，ACL 2026 Findings）、测隐式冲突（STALE）、测显式遗忘请求（BEMULA）。只做“又一个记忆系统 + LoCoMo 分数”在 2026 年底已无新意。
- **最适合硕士周期的基准组合**：主基准 **BEMULA**（规模小、专测删除、自带配对对照与攻击探针、刚发布尚无人刷）+ **LongMemEval-S 的 knowledge-update 子集（78 题）**（测显式更新）+ **STALE 子集**（测隐式更新）做三协议联合评测；LoCoMo 只作健全性检查，不作主结果。MemoryArena 太贵、MemBench 与本题不对口、MemoryAgentBench 的“选择性遗忘”名不副实（见 2.x）。
- **真正的新意空间**（按可做性排序）：①面向对话记忆的**写时句级来源（per-sentence provenance）+ 确定性级联清除**，清除对象包括事实、向量索引、**摘要/反思中的单个句子**，并用 BEMULA 式攻击探针验证——PolyGraph 在文档知识图谱上证明“摘要来源必须逐句、整表不可信”，但没人把它搬到对话个人记忆 + BEMULA 协议上；②**更新与遗忘的统一控制器 + 统一评测**（同一方法同时过显式更新、隐式更新、显式删除三关），现有工作各过一关；③删除的**可验证回执**在对话记忆层面的轻量实现（只做审计日志 + 删除后探针报告，不碰 KV cache/参数层，那两层已被 Execution-State Unlearning 与 SBU 占了）。
- **首选方案**：方案 A「ProvPurge：句级来源 + 确定性级联清除」（第 4.1 节）。**备选方案**：方案 B「三协议统一裁决器」（第 4.2 节），创新弱一档但风险更低、更容易做完。

---

## Findings

### 1. 现状全景：2023–2026 代表性工作

下表每项按“解决什么问题 / 核心方法 / 评测基准 / 留下的不足”组织。【已证实】除特别标注外均来自本次打开的摘要或正文页。

#### 1.1 记忆系统

| 工作 | 问题与核心方法【已证实】 | 评测【已证实】 | 留下的不足 |
|---|---|---|---|
| **Generative Agents**（Park et al., 2023, arXiv:2304.03442） | 可信行为模拟。记忆流（memory stream）按 recency×importance×relevance 检索，reflection 把记忆综合为高层结论，planning 生成计划；25 个 agent 的 Smallville 沙盒 | 人评可信度 + 组件消融，无标准化 QA 基准（二手转述与原文摘要一致） | 只追加、不更新、不删除；reflection 恰恰是后来“删不干净”的源头之一：事实被改写进反思后，原记录删除也无用 |
| **MemoryBank**（Zhong et al., 2023, arXiv:2305.10250, AAAI 2024） | 长期陪伴对话。存储对话/事件摘要/用户画像，稠密检索，更新机制借 Ebbinghaus 遗忘曲线按时间与重要性强化或衰减记忆。原文：https://ar5iv.labs.arxiv.org/html/2305.10250 | 自建 SiliconFriend 陪伴场景，人工评分/排序 | 这里的“遗忘”是**时间衰减**，不是冲突裁决、更不是按用户请求删除；无知识更新一致性评测。引用时务必区分三种“遗忘”：衰减式、覆盖式、请求删除式——审稿人最常抓的概念混淆 |
| **MemGPT / Letta**（Packer et al., 2023, arXiv:2310.08560） | 上下文窗口有限。借 OS 虚拟内存：core（常驻上下文）/ recall（可搜索历史）/ archival（向量归档）三层，LLM 用函数调用自编辑记忆 | Deep Memory Retrieval (DMR) 等（Zep 论文转述其 DMR 为 93.4%） | 自编辑靠 LLM 自觉，无版本、无来源、无冲突/删除语义；删除一个 archival 条目不处理已进 core 的副本 |
| **Mem0 / Mem0ᵍ**（Chhikara et al., 2025, arXiv:2504.19413, ECAI 2025）原文：https://arxiv.org/abs/2504.19413 | 生产可用记忆层。两阶段：抽取候选事实 → 更新阶段把每个候选与 top-10 相似旧记忆交给 LLM 做 **ADD / UPDATE / DELETE / NOOP** 裁决（更新阶段细节为【二手】多篇拆解一致转述，见 https://memo.d.foundation/breakdown/mem0 ）；Mem0ᵍ 用有向标记图存实体关系，矛盾时标记旧边失效而非硬删【二手】 | LoCoMo：自报相对 OpenAI Memory 的 LLM-judge +26%，p95 延迟比全上下文低 91%、token 省 >90%；图变体总分约 +2% | ①裁决全靠 LLM，错误 UPDATE 静默改写、无审计；②其 LoCoMo 数字被 Zep 公开质疑（见 2.1 缺陷），且在其自身实验里**全上下文准确率反而更高**【已证实，Zep 博客引 Mem0 结果】；③BEMULA 实测 Mem0 遗忘请求通过率 0%（见 3.1）——它的 DELETE 是为“新鲜度”服务，不是为“用户删除意图”服务 |
| **Zep / Graphiti**（Rasmussen et al., 2025, arXiv:2501.13956）原文：https://arxiv.org/html/2501.13956v1 | 动态知识整合。episode / semantic entity / community 三子图；**双时态**：每条边存事件时间 $t_{valid},t_{invalid}$ 与事务时间 $t'_{created},t'_{expired}$ 共四时间戳；新边与语义相关旧边经 LLM 判矛盾后，把旧边 $t_{invalid}$ 设为新边 $t_{valid}$，**失效保留历史** | DMR 94.8% vs MemGPT 93.4%；LongMemEval 自报最高 +18.5% 准确率、延迟 −90% | 这是“带版本/来源的结构化事实库”最强的已有实现——**再做一个双时态事实库等于复刻 Zep**。不足：失效≠删除（历史边仍可查，隐私删除场景不合格）；图构建 LLM 调用多、成本高；FactConsolidation 上仅 7%（见 2.3）说明其时序机制在“读时裁决”上并不可靠 |
| **A-MEM**（Xu et al., 2025, arXiv:2502.12110, NeurIPS 2025） | 记忆组织僵化。Zettelkasten 式：新记忆生成带关键词/标签/上下文描述的结构化笔记，动态链接历史笔记，新记忆触发旧笔记属性演化更新（二手转述：https://scirate.com/arxiv/2502.12110 ） | 6 个 backbone 上对比 SOTA（摘要自述） | “演化更新”会改写旧笔记——这正是删除场景的噩梦：被删事实可能已扩散进多篇笔记的属性里，且无来源记录可追 |
| **MemoryOS**（Kang et al., 2025, arXiv:2506.06326） | OS 式分层：短/中/长三层 + Storage/Updating/Retrieval/Generation 四模块，FIFO 与分段页晋升（摘要：https://arxiv.org/pdf/2506.06326 ） | LoCoMo 自报相对基线 F1 +49.11%、BLEU-1 +46.18%（GPT-4o-mini） | 晋升/淘汰是启发式，无冲突语义与删除语义；相对提升数字是与弱基线比，绝对值需回原文看 |
| **Memory-R1**（Yan et al., 2025/2026, arXiv:2508.19828, ACL 2026 主会）原文：https://aclanthology.org/2026.acl-long.583/ | 记忆操作靠启发式。用 RL（PPO/GRPO）训练 Memory Manager 学 ADD/UPDATE/DELETE/NOOP，Answer Agent 学从候选记忆中蒸馏；仅 152 条训练 QA | LoCoMo、MSC、LongMemEval，3B–14B 多尺度 | 证明了“操作可学”，但需要 RL 训练——**超出你的算力约束，不要走训练路线**；且 DELETE 仍为新鲜度服务，未测显式遗忘请求 |
| **HippoRAG / HippoRAG 2**（Gutiérrez et al., 2024–2025） | 多跳检索。海马索引理论：知识图谱 + Personalized PageRank 单步多跳（二手，多篇基线描述一致） | 在 MemoryAgentBench FactConsolidation-SH 上 54% 为已发表系统最好（见 2.3） | 是检索方法而非记忆管理方法，无写入裁决与删除 |

#### 1.2 记忆管理 / 更新 / 遗忘机制（与你的选题直接重叠，务必精读）

| 工作 | 核心方法【已证实】 | 与你的关系 |
|---|---|---|
| **STALE + CUPMem**（Chao et al., 2026, arXiv:2605.06527）原文：https://pdfs.assets.alphaxiv.org/2605.06527v1.pdf | 提出 **Implicit Conflict（隐式冲突）**：后一观察不带显式否定却使旧记忆失效，分 Type I 同指失效与 Type II 传播失效（如“腿摔伤”使“骑行通勤”失效）；配套原型 CUPMem 做写时结构化状态整合 + 传播感知检索 | “隐式更新没人做”已不成立。你的方法若主打更新，必须至少在 STALE 上与 CUPMem 比，或明确只做显式更新并说明边界 |
| **Memora + FAMA**（Uddin et al., 2026, arXiv:2604.20006, ACL 2026 Findings）原文：http://arxiv.org/abs/2604.20006 | 周–月尺度个性化对话基准，任务 remembering/reasoning/recommending；指标 **FAMA** 惩罚依赖已失效记忆的回答；4 LLM × 6 记忆 agent 实测“频繁复用失效记忆、不会调和演化中的记忆” | “更新后仍用旧事实”已被量化且已进 ACL Findings。你再提一个类似 FAMA 的指标会被直接判撞车；应直接**用** FAMA 或在其上加删除维度 |
| **PACMI**（Wang et al., 2026, arXiv:2610.05732）原文：https://arxiv.org/abs/2610.05732v1 | **来源图 + 类型依赖边 + 四态有效性格（active/superseded/needs-verification/historical-only）+ 级联失效传播** + 陈旧前提检测；自建 100 例 / 300 查询 / 5 域诊断集；去掉级联传播后最终答案错误 3→11，但配对检验 p=0.0574 **未达 0.05 显著**（作者自报） | **与“墓碑 + 级联”撞车度最高的方法之一**。区别点：它做的是更新失效、且**非破坏性保留历史**（隐私删除恰恰要破坏性清除）；评测是自建小集、非对话、未上 BEMULA/LongMemEval。你的差异化必须写死在这三点上 |
| **Agentic Unlearning / SBU**（Wang et al., 2026, arXiv:2602.17692）原文：https://arxiv.org/abs/2602.17692v1 | 提出 **backflow（回流）**：只删参数、记忆会重新污染模型；只删记忆、参数仍可被检索提示重新激活。SBU 双路同步：参数路用随机参考对齐把输出推向高熵先验；**记忆路做 dependency closure——剪除孤立实体、逻辑失效共享制品** | “级联删除 + 依赖闭包”已被命名并实现。但它需更新参数（医疗 QA、需训练），不适用于冻结 API 模型 + 对话个人记忆——这是你的生存空间，论文里要主动引它并划界 |
| **PolyGraph / Keep Your Receipts**（Choudhary, 2026, arXiv:2610.17719）原文：https://arxiv.org/html/2610.17719 | 文档知识图谱的来源撤回清理：每个制品在生成时同时产出 receipts（可替代的充分证据束），清理变成集合运算、无模型调用。关键发现：现行管线只删直接抽取事实，污染查询仅 32.2%→28.0%；**摘要的 receipts 若按整段存则约 2/3 是错的，改为逐句后忠实度 100%/88%**，且能保住 35 篇受影响摘要中的 33 篇 | 给你的首选方案提供了最硬的机制洞见（**句级来源**），但它的对象是文档 KG、来源是模型自报。你把“逐句来源”做进对话记忆的事实/摘要/反思三层、并用删除攻击验证，是真实增量 |
| **ChronoMem**（Su et al., 2026, arXiv:2607.27773）原文：https://arxiv.org/abs/2607.27773v2 | 语义版本控制：每次写产生**全量快照 commit**，自然语言 undo 请求经混合检索映射到具体版本后全局回滚；集成 Google ADK；post-exposure 协议测“回滚后像没见过后续更新一样答题”，约 +10pp | “带版本”本身已无新意（还有 Zep 双时态）。版本只能作你系统的一个组件，不能作标题卖点 |
| **Execution-State Unlearning / Provenance-Guided Selective Replay**（Yao et al., 2026，AAAI 2027 版式预印本，本次打开的 PDF） | 把遗忘定义为反事实等价（像从未见过目标）：来源图定位注入点、裁剪 KV cache 恢复检查点、净化重放后续轨迹；实测只删记忆记录时 leakage 与不删无异（0.86–1.00），指令式遗忘在抽取探针下 Leak=1.00 | “缓存中的级联清除”在**会话执行态**层面已被做透。你若声称处理缓存，必须引它；明智的做法是明确范围为**跨会话持久记忆层**，缓存只讨论不实现 |
| **确定性新鲜度管线**（Reddy, 2026-05，与 arXiv:2606.01435 同组） | 在 FactConsolidation 上用 candidate 抽取 + Python `max(serial)` 确定性管线替代 LLM 裁决，FC-SH 67.2→78.0（同设置对照） | “别让 LLM 管新鲜度、用确定性规则”这个叙事已被占。你的确定性组件必须落在自然对话的属性槽/来源图上，而非编号事实上 |

#### 1.3 综述（开题报告直接引这两篇搭框架）

- **Memory for Autonomous LLM Agents: Mechanisms, Evaluation, and Emerging Frontiers**（2026, arXiv:2603.07670）原文：https://arxiv.org/html/2603.07670v1 ——把记忆形式化为 write–manage–read 循环，三维分类（时间范围 × 表征基质 × 控制策略），五机制族（上下文压缩、检索存储、反思自改进、分层虚拟上下文、策略学习管理）；明确指出经典 IR 忽略的四件事：**staleness、contradiction、forgetting quality、governance compliance**【已证实】。你的论文问题定义可直接挂在这四词上。
- **Memory in the Age of AI Agents: A Survey**（Hu et al., 2025/2026, arXiv:2512.13564）——Forms（token/参数/latent）× Functions（事实/经验/工作）× Dynamics（formation/evolution/retrieval，其中 evolution 含 consolidation、updating、forgetting）【二手，转述一致】。
- 另有 A Survey on the Memory Mechanism of LLM-based Agents（Zhang et al., 2024, ACM TOIS, arXiv:2404.13501）为早期综述【二手】。

---

### 2. 基准与数据

#### 2.0 总表

| 基准 | 测什么 | 规模【已证实】 | 获取 | 指标 | 已知缺陷 |
|---|---|---|---|---|---|
| **LoCoMo**（Maharana et al., ACL 2024, arXiv:2402.17753） | 很长多会话对话的召回与推理：单跳/多跳/时序/开放域/对抗 | 公开版 10 段对话、每段 ~600 轮、1,986 题（不同论文口径 1,540 非对抗题 / 7,512 题含全部任务，**引用时注明口径**） | 公开（Snap Research 发布） | F1 / BLEU / LLM-judge | 见 2.1：答案键 6.4% 错误、judge 过松、无知识更新题 |
| **LongMemEval**（Wu et al., ICLR 2025, arXiv:2410.10813） | 五能力：信息抽取、多会话推理、时序推理、**知识更新**、弃答 | 500 题；类型分布：single-session-user 70 / assistant 56 / preference 30 / multi-session 133 / **knowledge-update 78** / temporal 133【已证实，SGMem 附录转录】；S 版 ~115K token/题，M 版 ~1.5M token/约 500 会话，另有 oracle 版 | GitHub：https://github.com/xiaowu0162/LongMemEval （论文摘要自述公开） | QA accuracy（LLM judge）、Recall@k / NDCG@k | S 版整段历史能塞进现代长上下文窗口，部分退化为上下文测试【已证实，Penfield 审计指出】；合成对话、话题多样性有限（MemoryAgentBench 作者批评） |
| **MemoryAgentBench (MAB)**（Hu et al., ICLR 2026, arXiv:2507.05257）原文：https://arxiv.org/html/2507.05257v3 | 四能力：准确检索 (AR)、测试时学习 (TTL)、长程理解 (LRU)、**选择性遗忘 (SF)**；把长上下文数据集改造成增量多轮喂入 | 2,071 题，上下文 103K–1.44M | GitHub：https://github.com/HUST-AI-HYZ/MemoryAgentBench ；HuggingFace 数据集（论文页链接） | Accuracy / Recall@5 / F1（按子任务映射 substring EM / EM / LLM-judge 等，仓库说明） | 见 2.3：SF 名不副实 + gold 可达性被质疑；上下文太长，全跑成本高 |
| **MemBench**（Tan et al., 2025, arXiv:2506.21605）原文：https://arxiv.org/html/2506.21605v1 | **事实记忆 vs 反思记忆** × **参与 vs 观察**两种场景 | 上下文 1K–100K；总题量二手口径不一致（1,039 与 53,000 QA 两种说法，本次未在原文核清，**列为未核清项**） | GitHub：https://github.com/import-myself/Membench （摘要自述） | effectiveness（accuracy）/ efficiency（记忆操作数）/ capacity（库变大时退化） | 与“更新/删除”不对口：测的是记忆层次与场景覆盖，没有显式更新后/删除后追问协议 |
| **MemoryArena**（He et al., 2026, arXiv:2602.16313, ICML 2026）原文：https://arxiv.org/html/2602.16313v1 | 记忆与行动耦合：相互依赖的多会话 agentic 任务（捆绑网购、偏好约束旅行规划、渐进式信息搜索、序贯形式推理） | 766 个任务、平均 6.9 个相互依赖子任务/任务、平均 57 步 | https://memoryarena.github.io/ | 任务成功率 / 进展分 | **贵**：作者自述全量 4,850 子任务 × 多系统评测 API 成本高昂到无法对多 backbone 穷举；且它测“会不会用记忆做事”，不测更新/删除的正确性。硕士周期内不建议作主基准 |
| **Memora**（Uddin et al., 2026，见 1.2） | 周–月尺度更新与失效复用 | 三种时间尺度设置（weekly/monthly/quarterly，二手） | 随论文（ACL 2026 Findings） | **FAMA**（惩罚用失效记忆） | 失效是对话自然演化带来的**隐式**失效，无显式删除请求、无抽取攻击探针（BEMULA 作者明确指出此区别【已证实】） |
| **STALE**（Chao et al., 2026，见 1.2） | **隐式冲突**下状态分辨 / 前提抵抗 / 隐式策略适配 | 400 专家校验场景、1,200 查询、100+ 话题、上下文至 150K | 随论文 | 三维探针 accuracy；最好模型仅 **55.2%** | 场景为构造式长上下文，测的多是“读时判断”，记忆系统需自行适配其喂入方式（原文以长上下文 + 记忆框架评测） |
| **BEMULA**（Xie et al., 2026-10-13, arXiv:2610.18052）原文：https://arxiv.org/html/2610.18052v1 | **显式遗忘请求**下的记忆卸载：删得掉、删得准、删后不被抽取、删后能再学相
...[truncated 20002 chars]