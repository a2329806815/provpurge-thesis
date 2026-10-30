# ProvPurge 顶会顶刊已发表论文补查报告

- 日期：2026-10-30
- 范围：为 ProvPurge（LLM Agent 长期记忆的知识更新与选择性遗忘；冻结模型、对话记忆、显式遗忘/更新触发；句级溯源图闭包 + 实体扩展生成候选、探针门控硬删除事务、三层硬删除 + 读时墓碑、回执签署探针记分卡）补查**已正式发表**的顶会顶刊论文。
- 方法：逐篇以 ACL Anthology、OpenReview、NeurIPS/ICLR/ICML 官方 proceedings、PMLR、AAAI、IEEE/ACM/USENIX 官方页面核实发表场所与年份；arXiv 页面只用作预印本证据，未用作发表证据。引用量一律未写（无可靠核实来源）。
- 筛选规则（用户硬性规则）：主清单只收已正式发表论文；arXiv-only 预印本入第 ② 节；往期已覆盖的 BEMULA、MemCut、LETHE、AfterDeleteBench、PolyGraph、PACMI、FEB-30、TRACE、TSD-AUDIT、VeriForgot、MemForm、Memora、StateAuditor、GateMem、Stashbird 与四篇综述不重复展开，仅在第 ② 节报告其发表状态变化。
- 撞车评级口径：**高**＝与 ProvPurge 核心主张（记忆层硬删除 + 探针门控提交 + 记分卡回执）实质重叠、足以动摇新颖性；**中**＝占据相邻主张空间（外部存储遗忘、探针验证删除、可验证遗忘、遗忘评测完整性、残留可恢复性等），相关工作必须正面区分；**低**＝同领域背景或机制远亲，常规引用即可。**本轮结论先行：已发表论文中无一篇评级为高——核心内核尚无已发表占位者；评级为中的共 10 篇，是相关工作必须处理的对象。**

---

## ① 主清单（已正式发表，共 52 篇）

### A. LLM/Agent 长期记忆系统与评测（15 篇）

**A1. MemoryBank: Enhancing Large Language Models with Long-Term Memory**
- 作者：Wanjun Zhong, Lianghong Guo, Qiqi Gao, He Ye, Yanlin Wang
- Venue：AAAI 2024 正会（Proc. AAAI 38(17): 19724–19731）
- 与 ProvPurge 的关系：可检索、可更新的长期记忆库，并按 Ebbinghaus 遗忘曲线做记忆强化/衰减——是已发表系统里少数自带「遗忘」机制者，但其遗忘是**被动衰减**，无显式触发、无删除验证。与 ProvPurge 的触发式硬删除正好构成对照轴。
- 撞车风险：**低**。衰减≠删除，不占探针验证与回执空间。

**A2. A-MEM: Agentic Memory for LLM Agents**
- 作者：Wujiang Xu, Zujie Liang, Kai Mei, Hang Gao, Juntao Tan, Yongfeng Zhang
- Venue：NeurIPS 2025 正会（poster）
- 与 ProvPurge 的关系：仿 Zettelkasten 的结构化笔记记忆，新记忆建链并触发旧记忆「演化」（更新侧机制）。其记忆演化是写入时自动改写，与 ProvPurge 的显式更新触发 + 删除事务不同层。
- 撞车风险：**低**。只有更新、无删除与验证。

**A3. LongMemEval: Benchmarking Chat Assistants on Long-Term Interactive Memory**
- 作者：Di Wu, Hongwei Wang, Wenhao Yu, Yuwei Zhang, Kai-Wei Chang, Dong Yu
- Venue：ICLR 2025 正会
- 与 ProvPurge 的关系：500 题长期记忆基准，五种能力含**知识更新**。ProvPurge 既定实验设计已用其知识更新子集（78 题）作辅助评测，本条为其正式发表出处，引用时应引 ICLR 版而非 arXiv。
- 撞车风险：**低**。基准本身不做删除方法主张。

**A4. Evaluating Very Long-Term Conversational Memory of LLM Agents（LoCoMo）**
- 作者：Adyasha Maharana, Dong-Ho Lee, Sergey Tulyakov, Mohit Bansal, Francesco Barbieri, Yuwei Fang
- Venue：ACL 2024 正会 Long Papers（pp. 13851–13870）
- 与 ProvPurge 的关系：超长多会话对话记忆基准。ProvPurge 既定设计中仅作 sanity check，本条为正式发表出处。
- 撞车风险：**低**。

**A5. Mem0: Building Production-Ready AI Agents with Scalable Long-Term Memory**
- 作者：Prateek Chhikara, Dev Khant, Saket Aryan, Taranjeet Singh, Deshraj Yadav
- Venue：ECAI 2025（IOS Press FAIA Vol. 413, pp. 2993–3000）——注意：Mem0 **已正式发表**，不是 arXiv-only。
- 与 ProvPurge 的关系：生产级记忆层，动态抽取/合并/检索对话信息，含图记忆变体；在 LoCoMo 上评测。是「工业界记忆层」的已发表代表，可作系统背景与潜在基线语境；其合并（merge）是更新语义，无删除验证。
- 撞车风险：**低**。

**A6. HippoRAG: Neurobiologically Inspired Long-Term Memory for Large Language Models**
- 作者：Bernal Jiménez Gutiérrez, Yiheng Shu, Yu Gu, Michihiro Yasunaga, Yu Su
- Venue：NeurIPS 2024 正会（poster）
- 与 ProvPurge 的关系：LLM 抽三元组建图 + Personalized PageRank 多跳检索。与 ProvPurge 的句级溯源图同为「图结构记忆」，但 HippoRAG 只做检索整合、无删除语义；可在方法章用作图记忆的结构先例。
- 撞车风险：**低**。

**A7. From RAG to Memory: Non-Parametric Continual Learning for Large Language Models（HippoRAG 2）**
- 作者：Bernal Jiménez Gutiérrez, Yiheng Shu, Weijian Qi, Sizhe Zhou, Yu Su
- Venue：ICML 2025 正会（PMLR 267: 21497–21515）
- 与 ProvPurge 的关系：HippoRAG 的持续学习扩展，非参数地整合新知识。属更新/整合侧背景。
- 撞车风险：**低**。

**A8. MemoryLLM: Towards Self-Updatable Large Language Models**
- 作者：Yu Wang, Yifan Gao, Xiusi Chen, Haoming Jiang, Shiyang Li 等
- Venue：ICML 2024 正会（PMLR 235: 50453–50466）
- 与 ProvPurge 的关系：在 Transformer 潜空间内嵌记忆池实现「自更新」——是**参数内记忆**路线，与 ProvPurge 的冻结模型 + 外置记忆层构成路线对照；其更新无遗忘、无验证。
- 撞车风险：**低**。

**A9. Generative Agents: Interactive Simulacra of Human Behavior**
- 作者：Joon Sung Park, Joseph C. O'Brien, Carrie J. Cai, Meredith Ringel Morris, Percy Liang, Michael S. Bernstein
- Venue：ACM UIST 2023 正会
- 与 ProvPurge 的关系：memory stream + reflection + planning 的奠基架构，Agent 长期记忆的系统叙事源头。背景引用。
- 撞车风险：**低**。

**A10. Reflexion: Language Agents with Verbal Reinforcement Learning**
- 作者：Noah Shinn, Federico Cassano, Ashwin Gopinath, Karthik Narasimhan, Shunyu Yao
- Venue：NeurIPS 2023 正会
- 与 ProvPurge 的关系：把语言反思存入 episodic memory buffer 指导重试——外置记忆的经典用法，无删除问题。背景引用。
- 撞车风险：**低**。

**A11. ReadAgent: A Human-Inspired Reading Agent with Gist Memory of Very Long Contexts**
- 作者：Kuang-Huei Lee, Xinyun Chen, Hiroki Furuta, John Canny, Ian Fischer
- Venue：ICML 2024 正会（PMLR 235: 26396–26415）
- 与 ProvPurge 的关系：分页→gist 摘要记忆→按需回查原文。其「摘要层与原文层分离」与 ProvPurge 三层删除中的摘要句层有结构呼应，但无删除语义。
- 撞车风险：**低**。

**A12. ExpeL: LLM Agents Are Experiential Learners**
- 作者：Andrew Zhao, Daniel Huang, Quentin Xu, Matthieu Lin, Yong-Jin Liu, Gao Huang
- Venue：AAAI 2024 正会（Proc. AAAI 38(17): 19632–19642）
- 与 ProvPurge 的关系：跨任务经验/insight 记忆。背景引用。
- 撞车风险：**低**。

**A13. MemBench: Towards More Comprehensive Evaluation on the Memory of LLM-based Agents**
- 作者：Haoran Tan, Zeyu Zhang, Chen Ma, Xu Chen, Quanyu Dai, Zhenhua Dong
- Venue：**Findings of ACL 2025**（pp. 19336–19352）——正式发表但为 Findings，引用与计数时须标注。
- 与 ProvPurge 的关系：分 factual/reflective 两层记忆、从效果/效率/容量三维评测 Agent 记忆。评测背景。
- 撞车风险：**低**。

**A14. MemoryAgentBench（正式题名：Evaluating Memory in LLM Agents via Incremental Multi-Turn Interactions）**
- 作者：Yuanzhe Hu, Yu Wang, Julian McAuley
- Venue：ICLR 2026 正会（OpenReview 标注 Published 2026-01-26）
- 与 ProvPurge 的关系：已发表基准中唯一设 **selective forgetting** 维度的一个——但其「遗忘」测的是新旧事实冲突时采用最新值（冲突消解），与 BEMULA/ProvPurge 的「按请求删除」不是同一语义。相关工作必须明确区分这两种 forgetting，否则评审可能误判 ProvPurge 与其重叠。
- 撞车风险：**中**。占据了已发表基准里的「forgetting」名目，但语义不同，区分清楚即可化解。

**A15. Agent Workflow Memory**
- 作者：Zora Zhiruo Wang, Jiayuan Mao, Daniel Fried, Graham Neubig
- Venue：ICML 2025 正会（PMLR 267: 63897–63911）
- 与 ProvPurge 的关系：从轨迹归纳 workflow 存入记忆并选择性召回。记忆「写入侧」背景。
- 撞车风险：**低**。

### B. LLM 机器遗忘 / 知识遗忘（9 篇）

**B1. TOFU: A Task of Fictitious Unlearning for LLMs**
- 作者：Pratyush Maini, Zhili Feng, Avi Schwarzschild, Zachary C. Lipton, J. Zico Kolter
- Venue：COLM 2024 正会（注：COLM 为 2024 年新创的语言建模会议，已正式发表、同行评审，但不在 CCF 列表内，引用归类时请留意）
- 与 ProvPurge 的关系：虚构作者问答遗忘基准，以与「从未学过」重训模型的统计接近度定义 forget quality——「删除后应与从未见过不可区分」这一判据的思想源头之一，但对象是模型参数。
- 撞车风险：**低**。参数遗忘，与记忆层删除不同层。

**B2. MUSE: Machine Unlearning Six-Way Evaluation for Language Models**
- 作者：Weijia Shi, Jaechan Lee, Yangsibo Huang, Sadhika Malladi, Jieyu Zhao, Ari Holtzman, Daogao Liu, Luke Zettlemoyer, Noah A. Smith, Chiyuan Zhang
- Venue：ICLR 2025 正会（poster）
- 与 ProvPurge 的关系：六维遗忘评测（逐字记忆、知识记忆、隐私泄漏、效用保持、可扩展性、连续请求可持续性）。它在**参数遗忘**领域先行占据了「遗忘评测要多维、要含泄漏探针」的主张；ProvPurge 的探针记分卡在记忆层做同类事情，相关工作须承认这条评测谱系并指出层级差异（参数 vs 外置记忆、离线评测 vs 提交门控）。
- 撞车风险：**中**。评测完整性主张的已发表先例，但不做门控、不做 Agent 记忆。

**B3. Negative Preference Optimization (NPO): From Catastrophic Collapse to Effective Unlearning**
- 作者：Ruiqi Zhang, Licong Lin, Yu Bai, Song Mei
- Venue：COLM 2024 正会（同 B1 的 COLM 说明）
- 与 ProvPurge 的关系：梯度上升遗忘的稳定化方法（参数侧）。方法背景。
- 撞车风险：**低**。

**B4. The WMDP Benchmark: Measuring and Reducing Malicious Use with Unlearning**
- 作者：Nathaniel Li, Alexander Pan, Anjali Gopal, Summer Yue, Daniel Berrios, Alice Gatti, Justin D. Li, Ann-Kathrin Dombrowski 等（WMDP Team）
- Venue：ICML 2024 正会（PMLR 235: 28525–28550）
- 与 ProvPurge 的关系：危险知识遗忘基准 + RMU 方法。参数遗忘的代表性评测背景。
- 撞车风险：**低**。

**B5. RWKU: Benchmarking Real-World Knowledge Unlearning for Large Language Models**
- 作者：Zhuoran Jin, Pengfei Cao, Chenhao Wang, Zhitao He, Hongbang Yuan, Jiachun Li, Yubo Chen, Kang Liu, Jun Zhao
- Venue：NeurIPS 2024 **Datasets and Benchmarks Track**（非 main track，引用须注明）
- 与 ProvPurge 的关系：在 forget/retain 语料都不可得的设定下，用成员推理（MIA）与**对抗探针**审计真实世界知识遗忘——「用攻击探针审计遗忘是否彻底」的已发表先例之一，但对象是参数知识、无门控提交概念。
- 撞车风险：**中**。探针审计遗忘的谱系必须引用并区分（审计 vs 门控）。

**B6. Knowledge Unlearning for Mitigating Privacy Risks in Language Models**
- 作者：Joel Jang, Dongkeun Yoon, Sohee Yang, Sungmin Cha, Moontae Lee, Lajanugen Logeswaran, Minjoon Seo
- Venue：ACL 2023 正会 Long Papers（pp. 14389–14408）
- 与 ProvPurge 的关系：梯度上升知识遗忘的源头论文之一，以降低隐私提取风险为目标。参数侧背景与动机引用。
- 撞车风险：**低**。

**B7. In-Context Unlearning: Language Models as Few-Shot Unlearners**
- 作者：Martin Pawelczyk, Seth Neel, Himabindu Lakkaraju
- Venue：ICML 2024 正会（PMLR 235: 40034–40050）
- 与 ProvPurge 的关系：**与 ProvPurge 设定最近的对照路线**：不改参数、冻结模型，仅靠推理时翻转标签的上下文示例实现「遗忘」行为。但它不做任何存储层删除，本质是行为掩蔽——信息仍在上下文/可被绕过。ProvPurge 的核心论证（掩蔽≠删除、必须硬删 + 探针验证）正是针对这类路线，相关工作必须正面引用并划清界限。
- 撞车风险：**中**。同为「冻结模型遗忘」，若不加区分，评审可能误以为 ProvPurge 只是其工程变体。

**B8. Can Sensitive Information Be Deleted From LLMs? Objectives for Defending Against Extraction Attacks**
- 作者：Vaidehi Patil, Peter Hase, Mohit Bansal
- Venue：ICLR 2024 正会 **Spotlight**
- 与 ProvPurge 的关系：以提取攻击为判据检验「删除」是否真实——发现 ROME 编辑后仍有 38% 的事实可被白盒攻击恢复。这是「表面删除、残留可恢复」在参数编辑侧的已发表实证，与 MemCut 在记忆侧的残留发现（预印本）互相印证，直接支撑 ProvPurge 探针门（尤其攻击式探针与残余直喂探针）的必要性论证。**必引**。
- 撞车风险：**中**。它提出问题与判据，ProvPurge 在记忆层给出事务化解答；主张不重叠但论证链相邻。

**B9. Rethinking Machine Unlearning for Large Language Models**
- 作者：Sijia Liu, Yuanshun Yao, Jinghan Jia, Stephen Casper, Nathalie Baracaldo, Peter Hase, Yuguang Yao, Chris Yuhao Liu, Xiaojun Xu, Hang Li 等
- Venue：Nature Machine Intelligence 2025（Review Article, Vol. 7, pp. 181–194）
- 与 ProvPurge 的关系：LLM 遗忘领域的权威重审，明确区分「真删除」与「行为抑制」。符合用户「先综述」的文献习惯，适合作相关工作开篇的已发表综述锚点，也为 ProvPurge 的「硬删除 vs 掩蔽」定位提供权威背书。
- 撞车风险：**低**。

### C. 知识编辑与更新（6 篇）

**C1. Locating and Editing Factual Associations in GPT（ROME）**
- 作者：Kevin Meng, David Bau, Alex Andonian, Yonatan Belinkov
- Venue：NeurIPS 2022 正会
- 与 ProvPurge 的关系：因果追踪 + 秩一权重更新改写单条事实。参数编辑路线的源头；ProvPurge 的更新/删除在记忆层完成，与之构成层级对照。
- 撞车风险：**低**。

**C2. Mass-Editing Memory in a Transformer（MEMIT）**
- 作者：Kevin Meng, Arnab Sen Sharma, Alex Andonian, Yonatan Belinkov, David Bau
- Venue：ICLR 2023 正会
- 与 ProvPurge 的关系：ROME 的批量化扩展（一次数千条事实）。参数编辑背景。
- 撞车风险：**低**。

**C3. MQuAKE: Assessing Knowledge Editing in Language Models via Multi-Hop Questions**
- 作者：Zexuan Zhong, Zhengxuan Wu, Christopher D. Manning, Christopher Potts, Danqi Chen
- Venue：EMNLP 2023 正会（pp. 15686–15702）
- 与 ProvPurge 的关系：用多跳问题检验编辑后连带推论是否同步更新，发现权重编辑在多跳上灾难性失败，并提出外置记忆编辑器 MeLLo。为 ProvPurge 的**多跳推导探针**与闭包删除提供已发表动机：单点删除/编辑不处理推导链就是不彻底的。
- 撞车风险：**低**。评测与动机先例，非删除事务。

**C4. Evaluating the Ripple Effects of Knowledge Editing in Language Models（RippleEdits）**
- 作者：Roi Cohen, Eden Biran, Ori Yoran, Amir Globerson, Mor Geva
- Venue：**TACL 2024**（期刊，Vol. 12, pp. 283–298）——正式发表于期刊而非会议，引用须注明
- 与 ProvPurge 的关系：检验一次事实编辑对逻辑关联事实的「涟漪效应」，主流编辑方法均不能维持关联一致性。这与 ProvPurge 的溯源闭包、F4 源—结论双语义是**同一类问题在编辑侧的已发表表述**：动一个事实必须处理其推导后代。相关工作应明确：ProvPurge 把涟漪问题从「编辑一致性评测」推进为「删除闭包 + 探针验证」。
- 撞车风险：**中**。问题表述相邻，但 RippleEdits 只评测、不给删除机制与验证。

**C5. WISE: Rethinking the Knowledge Memory for Lifelong Model Editing of Large Language Models**
- 作者：Peng Wang, Zexi Li, Ningyu Zhang, Ziwen Xu, Yunzhi Yao, Yong Jiang, Pengjun Xie, Fei Huang, Huajun Chen
- Venue：NeurIPS 2024 正会（poster）
- 与 ProvPurge 的关系：终身编辑的可靠性/泛化性/局部性「不可能三角」+ 主/侧双参数记忆。更新侧背景，侧记忆概念与外置记忆有弱呼应。
- 撞车风险：**低**。

**C6. Editing Large Language Models: Problems, Methods, and Opportunities**
- 作者：Yunzhi Yao, Peng Wang, Bozhong Tian, Siyuan Cheng, Zhoubo Li, Shumin Deng, Huajun Chen, Ningyu Zhang
- Venue：EMNLP 2023 正会（pp. 10222–10240；综述/系统评测性质）
- 与 ProvPurge 的关系：知识编辑领域的问题与方法分类锚点。编辑线综述引用。
- 撞车风险：**低**。

### D. 删除机制与可验证删除（17 篇）

**D1. Towards Making Systems Forget with Machine Unlearning**
- 作者：Yinzhi Cao, Junfeng Yang
- Venue：IEEE Symposium on Security and Privacy 2015（pp. 463–480）
- 与 ProvPurge 的关系：machine unlearning 概念奠基（求和形式使删除只需局部重算）。「删除应是局部化操作而非全量重做」的思想源头。
- 撞车风险：**低**。

**D2. Machine Unlearning（SISA）**
- 作者：Lucas Bourtoule, Varun Chandrasekaran, Christopher A. Choquette-Choo, Hengrui Jia 等
- Venue：IEEE Symposium on Security and Privacy 2021（pp. 141–159）
- 与 ProvPurge 的关系：分片—隔离—切片—聚合训练，删除时只重训受影响分片。是「影响范围定位 → 局部处理」的最经典机制先例；ProvPurge 的溯源闭包可表述为把 SISA 的静态分片换成数据驱动的动态影响集。机制谱系必引。
- 撞车风险：**低**。训练侧分片重训，与记忆层删除不同层。

**D3. Certified Data Removal from Machine Learning Models**
- 作者：Chuan Guo, Tom Goldstein, Awni Hannun, Laurens van der Maaten
- Venue：ICML 2020 正会（PMLR 119: 3832–3842）——注意正式题名无 “from a Single Machine”
- 与 ProvPurge 的关系：定义 certified removal（删除后与从未见过不可区分）。ProvPurge 既定立场是**不声称** (ε,δ) 认证删除；引用本文是为划清界限：ProvPurge 给的是经验性探针证据 + 签名记分卡，不是分布不可区分性证明。
- 撞车风险：**低**。定位划界用，非竞争。

**D4. Making AI Forget You: Data Deletion in Machine Learning**
- 作者：Antonio Ginart, Melody Y. Guan, Gregory Valiant, James Zou
- Venue：NeurIPS 2019 正会
- 与 ProvPurge 的关系：形式化 deletion-efficient learning（Q-k-means 等可证明高效删除）。删除效率的形式化背景。
- 撞车风险：**低**。

**D5. Eternal Sunshine of the Spotless Net: Selective Forgetting in Deep Networks**
- 作者：Aditya Golatkar, Alessandro Achille, Stefano Soatto（注意：与 D4 是**两篇不同的论文**，勿混）
- Venue：IEEE/CVF CVPR 2020 正会（pp. 9304–9312）
- 与 ProvPurge 的关系：用 Fisher 信息「擦洗」权重使权重探查无法区分是否见过待忘数据——「以探查不可区分定义遗忘」的早期形式，与探针思想有谱系关系，对象是网络权重。
- 撞车风险：**低**。

**D6. Remember What You Want to Forget: Algorithms for Machine Unlearning**
- 作者：Ayush Sekhari, Jayadev Acharya, Gautam Kamath, Ananda Theertha Suresh
- Venue：NeurIPS 2021 正会
- 与 ProvPurge 的关系：population 设定下 unlearning 的算法与 deletion capacity 理论界。理论背景。
- 撞车风险：**低**。

**D7. GNNDelete: A General Strategy for Unlearning in Graph Neural Networks**
- 作者：Jiali Cheng, George Dasoulas, Huan He, Chirag Agarwal, Marinka Zitnik
- Venue：ICLR 2023 正会
- 与 ProvPurge 的关系：**已发表文献中与 ProvPurge 删除语义最近的一篇**：模型无关的图上删除算子，以 Deleted Edge Consistency（删边后预测应如边从不存在）+ Neighborhood Influence（邻居影响应被修正）两条性质定义「删对了」。ProvPurge 的溯源图闭包删除与之同构（图上删除 + 邻域连带处理），但对象是对话记忆存储而非 GNN 权重，且 ProvPurge 多了探针门控与回执。相关工作必须正面对比。
- 撞车风险：**中**。图删除语义的已发表占位者，层级与验证方式不同。

**D8. Graph Unlearning（框架名 GraphEraser；正式题名即 Graph Unlearning）**
- 作者：Min Chen, Zhikun Zhang, Tianhao Wang, Michael Backes, Mathias Humbert, Yang Zhang
- Venue：ACM CCS 2022 正会
- 与 ProvPurge 的关系：SISA 式分片重训向图数据的迁移（保结构划分 + 学习式聚合）。图遗忘机制背景。
- 撞车风险：**低**。

**D9. Efficient Model Updates for Approximate Unlearning of Graph-Structured Data**
- 作者：Eli Chien, Chao Pan, Olgica Milenkovic
- Venue：ICLR 2023 正会（注：其 arXiv 预印本题为 “Certified Graph Unlearning”，正式发表时改题，引用须用 ICLR 题名；另有 NeurIPS 2022 GLFrontiers Workshop 前身）
- 与 ProvPurge 的关系：首个带可证明保证的近似图 unlearning（节点特征/边/节点删除）。图删除 + 保证的谱系引用。
- 撞车风险：**低**。

**D10. Amnesiac Machine Learning**
- 作者：Laura Graves, Vineel Nagisetty, Vijay Ganesh
- Venue：AAAI 2021 正会（Proc. AAAI 35(13): 11516–11524）
- 与 ProvPurge 的关系：按批保存参数更新、删除时回滚，并用模型反演/成员推理攻击**检验删除效果**——「用攻击检验删除」在训练侧的早期实践，可作探针验证的思想谱系。
- 撞车风险：**低**。

**D11. Proof-of-Learning: Definitions and Practice**
- 作者：Hengrui Jia, Mohammad Yaghini, Christopher A. Choquette-Choo, Natalie Dullerud, Anvith Thudi, Varun Chandrasekaran, Nicolas Papernot
- Venue：IEEE Symposium on Security and Privacy 2021 正会
- 与 ProvPurge 的关系：为「训练过程」构造可验证证明（证明方确实付出了训练算力）。是回执/证明思路的源头，但证明对象是「训练过」而非「删除过」；与 D14 合读构成回执设计的正反两极。
- 撞车风险：**低**。

**D12. Verifiable and Provably Secure Machine Unlearning**
- 作者：Thorsten Eisenhofer, Doreen Riepel, Varun Chandrasekaran, Esha Ghosh, Olga Ohrimenko, Nicolas Papernot
- Venue：IEEE SaTML 2025（3rd IEEE Conference on Secure and Trustworthy Machine Learning, pp. 479–496）——正式发表；注：SaTML 是安全 ML 方向的 IEEE 会议，不在 CCF 列表内
- 与 ProvPurge 的关系：**已发表文献中与 ProvPurge 回执最近的一篇**：首个密码学框架定义 verifiable unlearning，用 SNARK + 哈希链给出 proof of unlearning。但其证明绑定的是训练数据成员关系（d ∉ D′），不涉及记忆存储层的内容缺失检查与行为探针；ProvPurge 的记分卡回执签的是探针结果与逐层哈希，验证对象是「内容不可恢复」。相关工作必须明确这一区分（密码学成员证明 vs 内容级探针证据）。
- 撞车风险：**中**。占据「可验证遗忘」的旗舰名目，但层级与证据类型不同。

**D13. Athena: Probabilistic Verification of Machine Unlearning**
- 作者：David M. Sommer, Liwei Song, Sameer Wagh, Prateek Mittal
- Venue：PoPETs 2022（Proceedings on Privacy Enhancing Technologies 2022(3): 268–290；PETS 2022）
- 与 ProvPurge 的关系：**已发表文献中与 ProvPurge 探针门最近的一篇**：用预先植入的后门样本 + 假设检验，概率化验证服务方是否真执行了删除——「提交/宣布删除之前（之后）用探针检验」的形式化框架。ProvPurge 的探针门与之同构，但场景是记忆层内容探针（问答/直喂/多跳/过时前提）而非训练数据后门，且探针结果进入提交条件与签名回执。必引并区分。
- 撞车风险：**中**。探针验证删除的已发表占位者，对象与用途（审计第三方 vs 自系统门控）不同。

**D14. On the Necessity of Auditable Algorithmic Definitions for Machine Unlearning**
- 作者：Anvith Thudi, Hengrui Jia, Ilia Shumailov, Nicolas Papernot
- Venue：USENIX Security 2022 正会
- 与 ProvPurge 的关系：证明仅凭训练轨迹/参数**无法**形式化证明某数据未被训练过，PoL/PoUL 类证明可被数据排序攻击伪造，主张 unlearning 只能在算法层审计。这是回执设计必须正面回应的已发表批评：ProvPurge 回执因此不能只签「删除事件」，必须绑定内容缺失检查与探针复检——既定规格 §9 的「purged-row absence check + probe re-check 强制项」正是对这条批评的回应，论文里应明说。
- 撞车风险：**中**。不是竞品，而是约束回执主张边界的必引反方文献。

**D15. When Machine Unlearning Meets Retrieval-Augmented Generation (RAG): Keep Secret or Forget Knowledge?**
- 作者：Shang Wang, Tianqing Zhu, Dayong Ye, Wanlei Zhou
- Venue：IEEE Transactions on Dependable and Secure Computing（TDSC，CCF A 期刊；DOI 10.1109/TDSC.2025.3620832，卷期 Vol. 23(2): 2030–2045，年份标注以 DOI 为准）
- 与 ProvPurge 的关系：**已发表文献中与 ProvPurge 场景最近的一篇**：不改 LLM 参数、只改造 RAG 外部知识库实现行为层遗忘，适用于闭源 LLM。它占据了「外部存储侧遗忘」这一主张空间；但其路线是构造「unlearned knowledge」达成行为遗忘，无溯源闭包、无三层硬删除、无探针门控与回执。ProvPurge 的新颖性表述必须据此收窄：不能再笼统声称「首个在外部记忆/知识库做遗忘」，应精确到「首个对 Agent 对话记忆做溯源闭包硬删除 + 探针门控提交 + 记分卡回执的事务」。
- 撞车风险：**中**（本轮最高档之一）。场景重叠、机制不同；靠精确限定主张化解。

**D16. Customized Retrieval-Augmented Generation with LLM for Debiasing Recommendation Unlearning（CRAGRU）**
- 作者：Haichao Zhang, Chong Zhang, Peiyu Hu, Shi Qiu, Jia Wang
- Venue：IEEE ICDM 2025 正会
- 与 ProvPurge 的关系：把 unlearning 解耦为检索阶段过滤 + 生成阶段重构，在推荐场景隔离目标用户数据影响。「检索层过滤式遗忘」的旁例，领域较远。
- 撞车风险：**低**。

**D17. CPR: Retrieval Augmented Generation for Copyright Protection**
- 作者：Aditya Golatkar, Alessandro Achille, Luca Zancato, Yu-Xiang Wang, Ashwin Swaminathan, Stefano Soatto
- Venue：IEEE/CVF CVPR 2024 正会（pp. 12374–12384）
- 与 ProvPurge 的关系：在 RAG 场景下论证「从检索库移除数据即实现确定性、常数时间的 unlearning」并给出 Near Access Freeness 保证。是「删存储即遗忘」命题在已发表文献中的最强表述之一，可作 ProvPurge 硬删除路线的旁证；但其对象是图像扩散模型的检索集，非 LLM 文本 Agent 记忆，且无推导闭包与行为验证问题。引用时须注明对象差异。
- 撞车风险：**低**。

### E. Agent 记忆隐私与攻击（5 篇）

**E1. AgentPoison: Red-teaming LLM Agents via Poisoning Memory or Knowledge Bases**
- 作者：Zhaorun Chen, Zhen Xiang, Chaowei Xiao, Dawn Song, Bo Li
- Venue：NeurIPS 2024 正会（poster）
- 与 ProvPurge 的关系：首个针对 LLM Agent 长期记忆/知识库的后门投毒攻击——「记忆写入侧不可信」的已发表代表。与 MemForm（预印本，形成侧投毒）并列引用，支撑 ProvPurge 的威胁模型写入侧一半。
- 撞车风险：**低**。攻击侧文献，不占删除主张。

**E2. PoisonedRAG: Knowledge Corruption Attacks to Retrieval-Augmented Generation of Large Language Models**
- 作者：Wei Zou, Runpeng Geng, Binghui Wang, Jinyuan Jia
- Venue：USENIX Security 2025 正会（pp. 3827–3844）——注意正式题名是 Knowledge **Corruption** 而非 Poisoning
- 与 ProvPurge 的关系：向 RAG 知识库注入少量恶意文本即可操纵答案。知识库完整性威胁的已发表锚点。
- 撞车风险：**低**。

**E3. Unveiling Privacy Risks in LLM Agent Memory（攻击名 MEXTRA）**
- 作者：Bo Wang, Weiyi He, Shenglai Zeng, Zhen Xiang, Yue Xing, Jiliang Tang, Pengfei He
- Venue：ACL 2025 正会 Long Papers（pp. 25241–25260）
- 与 ProvPurge 的关系：黑盒设定下用 locator + aligner 两段式攻击提示，从 Agent 记忆中成批提取历史私密 query，在 EHRAgent、RAP 上验证。**这是 ProvPurge 威胁模型的已发表核心锚点**：它证明记忆内容可被系统性提取，从而「删没删干净」必须用攻击式探针检验；ProvPurge 探针集中的攻击式提问与残余直喂设计应明确引用其攻击分类。必引。
- 撞车风险：**中**。它定义攻击面，ProvPurge 给防御与验证；不区分会被误读为「攻击已解决一切」。

**E4. Memory Injection Attacks on LLM Agents via Query-Only Interaction（MINJA）**
- 作者：Shen Dong, Shaocheng Xu, Pengfei He, Yige Li, Jiliang Tang, Tianming Liu, Hui Liu, Zhen Xiang
- Venue：NeurIPS 2025 正会（poster）——注意：正式发表时已改题（原预印本题为 “A Practical Memory Injection Attack against LLM Agents”），引用须用新题名
- 与 ProvPurge 的关系：攻击者无写权限、仅靠普通 query 交互即可把恶意记录注入 Agent 记忆。写入侧威胁的最新已发表代表，与 E1 并列。
- 撞车风险：**低**。

**E5. InjecAgent: Benchmarking Indirect Prompt Injections in Tool-Integrated Large Language Model Agents**
- 作者：Qiusi Zhan, Zhixiang Liang, Zifan Ying, Daniel Kang
- Venue：**Findings of ACL 2024**（pp. 10471–10506）——正式发表但为 Findings，须标注
- 与 ProvPurge 的关系：工具集成 Agent 的间接提示注入基准（含数据窃取类）。与记忆提取相邻的攻击背景。
- 撞车风险：**低**。

---

## ② 撞车预警（预印本，未正式发表，不进主清单）

**P1. MemGPT: Towards LLMs as Operating Systems**（Packer et al., arXiv:2310.08560, 2023）
- 状态：**仅预印本**。已以作者主页与 arXiv 页面核实，无 journal-ref、无会议收录记录。
- 预警说明：它是 Agent 记忆领域被引最多的系统叙事之一，评审与读者都会预期在相关工作中见到它；但按用户硬规则不能进主清单。若论文必须提及，只能以 arXiv 预印本身份出现在本节口径的位置（或脚注），并注明未正式发表。BEMULA 的基线集合不含 MemGPT，实验设计不受影响。

**P2. Who's Harry Potter? Approximate Unlearning in LLMs**（Eldan & Russinovich, arXiv:2310.02238, 2023）
- 状态：**待核实，倾向仅预印本**。个别非官方清单标注 ICLR 2024 且有 OpenReview forum 链接流传，但在 ICLR 官方虚拟会场检索不到该文条目，多数后续正式论文仍按 arXiv 2023 引用；OpenReview 页面受验证墙阻挡，未能确认该 forum 的录用决定。
- 预警说明：这是 LLM 遗忘的标志性工作，TOFU/MUSE 等已发表工作都建立在它之上。核实清楚前不进主清单；若 Joseph 后续人工登录 OpenReview 确认 forum decision 为录用，可升入主清单 B 节。

**P3. 已覆盖预印本的发表状态更新（2026-10-30 复核，均无变化）**
- **BEMULA**（arXiv:2610.18052）：仍仅预印本，无中稿/发表记录。主基准地位不变；论文中继续按预印本引用，并注意其作为主基准的发表状态风险（评审可能质疑基准未发表——既定应对是同时报告 LongMemEval-S 等已发表基准上的辅助结果，A3 已确认 ICLR 2025 出处）。
- **MemCut**（arXiv:2610.21405）：仍仅预印本。
- **LETHE**（本论文所指为记忆存储层 purge + 签名回执一文）：仍仅预印本，无发表记录。注意检索中有**同名异文**干扰（一篇联邦 unlearning 方向的 “Lethe”，arXiv:2601.22601，亦为预印本），引用与检索时须以题名全称/作者区分，勿混。
- **AfterDeleteBench**（arXiv:2610.31366）：仍仅预印本。
- **PolyGraph / Keep Your Receipts**（arXiv:2610.17719）：仍仅预印本。其 arXiv Comments 自述与 NeurIPS 2026 Workshop “Who Verifies the Agents?” 相关，正文脚注自称 workshop camera-ready 可能不同，但本轮未在 OpenReview/NeurIPS 官网找到独立收录页佐证；且即便属实，Workshop 论文按用户规则亦不进主清单。原仓库 404 的状态与本条无关，不改变既有处理。

**P4. 向量数据库专属删除方向的检索缺口**
- 本轮以 “vector database deletion unlearning” 等多组检索词核查，检到的向量库删除后残留/删除验证类工作**均为 arXiv 预印本，无一篇可确认正式发表**，故不列名、不编造。结论：在「向量库删除」这一具体点上，已发表文献是空白，ProvPurge 的向量层硬删除 + 探针复检没有已发表直接对手；但这也意味着该点的新颖性只能靠预印本圈层（MemCut 等）界定，写作时不要声称「首个研究向量库删除」这类无法核实的宽泛主张。

---

## ③ 结论（一页）

### 必须进论文相关工作的（按优先级）
1. **E3 MEXTRA / Unveiling Privacy Risks in LLM Agent Memory（ACL 2025）**——记忆提取攻击的已发表锚点，ProvPurge 攻击式探针的威胁模型出处。
2. **B8 Patil et al.（ICLR 2024 Spotlight）**——「删除后 38% 仍可提取恢复」的已发表实证，探针门必要性的最强已发表论据。
3. **D13 Athena（PoPETs 2022）+ D12 Eisenhofer et al.（SaTML 2025）+ D14 Thudi et al.（USENIX Security 2022）**——探针验证删除、密码学可验证遗忘、证明可伪造性批评，三篇合起来界定回执主张的边界，缺一不可。
4. **D15 Wang et al.（IEEE TDSC）**——外部知识库遗忘的已发表最近邻，决定新颖性表述的收窄方式（见下）。
5. **B7 In-Context Unlearning（ICML 2024）+ B9 NMI 2025 重审**——「掩蔽≠删除」的对照路线与权威区分，ProvPurge 定位句的出处支撑。
6. **A14 MemoryAgentBench（ICLR 2026）**——已发表基准中唯一的 forgetting 维度，必须主动区分其冲突消解语义与按请求删除语义。
7. **D7 GNNDelete（ICLR 2023）+ C4 RippleEdits（TACL 2024）**——图删除语义与涟漪/闭包问题的已发表表述，溯源闭包的谱系与区分对象。
8. **B2 MUSE（ICLR 2025）+ B5 RWKU（NeurIPS 2024 D&B）**——遗忘评测完整性与探针审计遗忘的已发表谱系。
9. 系统与基准锚点：**A1 MemoryBank（AAAI 2024）、A3 LongMemEval（ICLR 2025）、A4 LoCoMo（ACL 2024）、A5 Mem0（ECAI 2025）、A2 A-MEM（NeurIPS 2025）、E1 AgentPoison（NeurIPS 2024）、E4 MINJA（NeurIPS 2025）**——分别支撑记忆系统背景、辅助评测出处、写入侧威胁模型。

### 改变什么
- **相关工作结构改变（新增两个已发表小节）**：其一「可验证遗忘」（D11–D14 + D3）——此前该空间看似只有 LETHE（预印本）一家，实际已有密码学证明（Eisenhofer）、探针概率验证（Athena）与证明局限批评（Thudi）三篇已发表工作，回执一节必须以这三篇为坐标系重写定位句：ProvPurge 回执的差异点是「内容级探针记分卡 + 逐层哈希 + 缺失复检，绑定 Agent 记忆层」而非「首个可验证删除」。其二「外部存储遗忘」（D15、D17、B7）——「在存储侧遗忘」已有已发表先例，新颖性主张按 D15 条目所述精确收窄到事务三元组（闭包硬删 + 探针门控 + 记分卡回执）。
- **探针设计多一条已发表依据（不改设计、改论证）**：Athena 的「预置探针 + 假设检验」框架可直接引用为探针门的理论谱系；实验报告部分可借其口径把探针结果表述为带统计检验的验证证据。探针集构成（原生问答/残余直喂/多跳/过时前提/攻击式）不变。
- **引用规范修正（防参考文献出错）**：D3 正式题名无 “from a Single Machine”；D4 与 D5 是两篇不同的论文（Ginart-NeurIPS 2019 vs Golatkar-CVPR 2020）；D8 正式题名为 Graph Unlearning；D9 须引 ICLR 2023 改名版题名；E2 正式题名是 Knowledge Corruption；E4 须引 NeurIPS 2025 改名版题名；A14 须用正式题名并注明其 forgetting 语义。以上均已在本报告条目中按正式题名著录。

### 不改变什么（明确）
- **方法内核不改变**：本轮 52 篇已发表论文中无一篇实现或声称「溯源闭包候选生成 + 探针门控提交 + 探针记分卡回执」的组合，无高撞车项；A+B 两项已采纳修正（不完整来源闭包、F4 双语义）与本轮发现不冲突，规格书 §5/§7/§9 的既定修订（双通道探针评分、逐实体扩展、分层缺失检查、删除后摘要重建定序、回执规范 JSON 序列化、缺失检查 + 探针复检强制项）全部保留——其中缺失复检一条恰好被 D14 的批评正面支持。
- **实验设计不改变**：主基准仍为 BEMULA（其预印本状态风险已由 A3 LongMemEval ICLR 出处对冲），不新增 MemoryAgentBench 为评测基准（仅相关工作区分），不引入 MUSE/RWKU 等参数遗忘基准作对比（层级不同）；基线集合、双 3090×5 的本地推理计划、C1/C2 第二贡献的待决状态均不受本轮影响。
- **投稿线不改变**：本轮发现的是相关工作坐标，不是新的投稿约束。

---

## 附：核实口径备忘
- COLM 2024（B1、B3）为正式同行评审会议但不在 CCF 列表；IEEE SaTML（D12）同理。如 Joseph 要求主清单严格限定 CCF 列表场所，可将这三篇降入「已发表但非 CCF」附注，主清单计数相应减为 49。
- Findings（A13、E5）与 NeurIPS D&B Track（B5）均为正式发表，但按「CCF 只认 Full/Regular」的口径在学位材料中应单独标注，本报告已逐条标注。
- 本报告未运行任何实验、未写任何代码；全部发表状态以 2026-10-30 检索到的官方页面为准。
