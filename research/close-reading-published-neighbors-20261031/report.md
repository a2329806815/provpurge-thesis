# 已发表近邻精读：6 篇撞车最高论文的全文精读卡与 ProvPurge 定位句修订建议

- 日期：2026-10-31
- 对象：上一轮顶会补查中撞车评级最高、必须正面区分的 6 篇已发表论文。6 篇全部读到全文（证据等级逐卡标注）。
- 用法：每卡的「区分句」可直接进论文相关工作；汇总部分的定位句与规格书修订建议只作建议，不改动规格书本身。
- ProvPurge 内核（对照基准）：冻结底座；句级溯源闭包 + 逐实体扩展生成候选；事实/向量/摘要三层硬删；删除前哈希预承诺的五类攻击探针（原生问答、残余直喂、多跳推导、过时前提、攻击式）全部失败且兄弟保全探针全部通过才提交；回执签署逐项探针记分卡，验证强制含被删行缺席检查 + 探针复检；A 不完整来源闭包；B F4 源—结论双语义；第二贡献推荐 C1+C2 合并「可证伪的删除证明」（待确认）。

---

## 卡 1 — Wang et al., When Machine Unlearning Meets RAG, IEEE TDSC

正式出处：IEEE TDSC, vol. 23, pp. 2030–2045（arXiv:2410.15267 注释 To appear in TDSC 2025）。作者 Shang Wang、Tianqing Zhu（通讯）、Dayong Ye、Wanlei Zhou。正式题名含副标题 “Keep Secret or Forget Knowledge?”。

### ① 一句话主张
不触碰 LLM 参数、不训练，只向 RAG 外部知识库**新增**带保密要求的条目，靠检索命中后让模型拒答，实现可用于闭源模型的**行为性遗忘**——目标知识并未被删除，模型内部仍保留它。

### ② 方法细节
- 条目结构：K_i = P + Q。P 为 retrieval component（检索分量），Q 为 constraint component（约束分量，内含 confidentiality requirement）。注意论文 §IV-C 首段把两分量功能写反，与 Table II 及小节标题矛盾，引用以 Table II 为准。
- 构造 Q：用额外的 LLM（如 GPT-4o）针对目标概念生成含保密要求的文本（如以版权为由拒答），迭代生成直至待遗忘模型据此拒答，作者称一般一次即可。
- 构造 P：样本遗忘直接令 P = 待忘样本原文 x_i；概念遗忘由待遗忘模型自身生成概念的全面描述（默认 5 个 aspects）。
- 入库与触发：UK 与良性库 BK 合并，**只新增，不删改 BK 原有条目**。用户提问经检索，命中含保密要求的条目则模型服从拒答，未命中则直接作答。主实验用 GPT-4o 本身充当 retriever，报告命中率 100%。
- 失效条件（论文自认）：检索未命中即失败；保密要求泄露后攻击者可构造覆盖提示，自适应泄漏下 USR 随曝光比例单调崩塌；§V-F 明言 “the LLM still retains related knowledge of the target data”，绕过 RAG 管线直连底座即可恢复，其威胁模型靠假设用户不能绕过管线排除此情形。
- **对存储内容的最终结论：不删除、不修改。** 目标知识完整保留在 LLM 中，且以 P 的形式被复制进 UK。§VII-D 讨论的“直接从知识库移除”是作者明确区分的另一场景，非本文方法。

### ③ 评测设置与主要数字
- 模型：GPT-4o、Gemini（闭源）；Llama-2-7b-chat（开源，样本+概念）。数据：概念遗忘为 100 个 Wikipedia 主题 × 5 问；样本遗忘为 NQ 子集 Tiny-nq（2,000 条）微调 Llama-2-7b-chat。
- 指标：ROUGE-L recall（越低越好）、USR（GPT-4o 按模板判定，越高越好）、Min-K MIA 的 TPR@1%FPR（K=20，原模型 4.1%）。
- 主结果：概念遗忘 USR — GPT-4o 99.2%、Gemini 99.5%、Llama-2-7b-chat 99.8%；样本遗忘 Llama USR 100%。闭源唯一可用基线 In-context Unlearning 平均 USR 仅 5.2%。
- 分量消融（Table XIII，均值）：仅检索分量命中率 98.6% 但 USR 仅 5.2%；仅约束分量命中率 19.4%、USR 22.5%；两者结合 USR 99.1%。
- 自适应泄漏（Table XII）：曝光 100% 时样本 USR 21.5%、概念 20.2%（正文称最坏平均 20.9%）。
- 简洁性：遗忘 5 个概念约 63 秒，约为 Gradient Ascent（4,752 s）的 1/75。

### ④ 与 ProvPurge 的关系
重叠：同为冻结底座、不改参数、同在外部存储层做文章、同把删除后行为作判据、同关注改写与越狱式绕过。
边界（四条硬区别）：(1) 操作方向相反——Wang 是加法（新增保密条目），ProvPurge 是减法硬删除；(2) 目标残留——Wang 的目标知识保留在模型内甚至被复制进 UK，ProvPurge 删后不得残留；(3) 提交判据——Wang 无探针门、无预承诺，USR 由 GPT-4o 事后单次判定，ProvPurge 须预承诺探针全败 + 兄弟全过才提交；(4) 可验证性——Wang 无回执、无缺席检查，删除方与验证方同体。A 在 Wang 中的对应物是“P 覆盖不足导致漏检”，但其解法是加更全描述，ProvPurge 的解法是来源不完整时不许提交，方向相反；B 在 Wang 中完全缺席。
区分句（中）：「与 Wang 等通过向外部知识库新增保密指令条目来行为性抑制输出不同，ProvPurge 不保留被删内容、不注入保密指令，而是对溯源闭包内的事实、向量与摘要执行硬删除，并以删除前哈希预承诺的探针集全部失败作为提交条件。」
Distinction (EN): “Unlike Wang et al., who add confidential-instruction entries to an external knowledge base to behaviorally suppress outputs while the underlying knowledge remains in the LLM, ProvPurge retains no target content and injects no secrecy instruction; it hard-deletes the provenance closure across facts, vectors, and summaries and commits only when a hash-pre-committed probe set entirely fails.”

### ⑤ 对 ProvPurge 的启示
- 方法：覆盖与约束缺一不可可作消融蓝本（其仅检索分量 USR 5.2%）；ProvPurge 须明确声明删除候选清单本身的处置（删后销毁或仅存哈希），避免像 P 一样把秘密再复制一份。
- 实验：可借其自适应泄漏协议做探针/回执泄漏压力测试（按曝光比例报衰减曲线）；其 USR 完全依赖 GPT-4o 单裁判、无双通道校验，是可引用的评测弱点，正反衬 ProvPurge 的双通道评分（机读解析 + 原文检索，冲突以原文为准）应报告一致率。
- 写作：相关工作须设 external-store / RAG-based behavioral unlearning 坐标并主动让步，不得声称首个外部存储遗忘或首个冻结底座遗忘；其 §VII-D 自承“直接移除知识库信息定位困难”恰是溯源闭包要解决的问题，可引为动机句。

### ⑥ 证据等级
全文（arXiv HTML v2 作者稿，§I–§VIII、Table I–XIII、Algorithm 1；非 IEEE 排版 PDF，页码经 OpenAIRE 记录交叉核对）。

---

## 卡 2 — Athena: Probabilistic Verification of Machine Unlearning, PoPETs 2022

正式出处：Sommer, Song, Wagh & Mittal, Proceedings on Privacy Enhancing Technologies 2022(3): 268–290, DOI 10.56553/popets-2022-0072。

### ① 一句话主张
在 MLaaS 场景下，数据所有者在**提交训练数据之前**预先植入只有自己知道的用户专属后门，删除请求后仅凭黑盒预测标签上后门成功率的落差，用假设检验以可量化的置信度概率推断服务器是否把其数据排除出训练——它验证的是训练集排除的统计迹象，不是物理删除，也不执行任何删除。

### ② 方法细节
- 三阶段：提交数据 → 提交删除请求 → 验证。参与验证的用户称 privacy enthusiasts；普通用户与服务器流程无改动。
- 后门预置（删除前、提交前）：每位用户独立随机选触发器与目标标签，仅本人知晓。图像为随机 4 个像素/点（EMNIST/CIFAR10 置 1、FEMNIST 置 0；ImageNet 用 32×32×3 掩膜 4 点放大），文本为末 15 词中 4 个词位替换为随机专属词；将触发器加到本人比例 f_data 的样本并改标签为目标标签（即投毒）后再提交训练。
- 假设检验：H_0 为服务器已删、H_1 为未删。未删时后门成功率 p 高、已删时 q 低（理想接近随机猜测）。验证者用触发器查询 n 个样本得成功率 r̂，在 t 两侧判读；先固定 Type I 错误 α（如 10⁻³），删除置信度 ρ = 1 − β（β 为漏检率），Theorem 1 给出 ρ 关于 (p, q, n, α) 的闭式。p 在删除前查询估计，q 用未见过该后门的模型或新后门估计。
- 多人协作：c 个用户共享 p̂、q̂、累积 c·n 样本联合检验可把假阴性压到极低；每用户可用多触发器取最低成功率为最坏判据。
- 验证的精确主张：只能主张“以置信度 ρ 推断该用户数据已被排除出该 MLaaS 所暴露的那个具体模型的训练过程”。论文明言范围不含存储的物理删除、不含服务器其他资源与模型副本；这是概率推断而非证明。
- 威胁模型：服务器分诚实、non-adaptive 恶意、adaptive 恶意三类。假设验证者提交前能控制足量自有数据、只需最终预测标签（黑盒）、服务器无法识别查询者身份（需匿名通信）、验证须贴近删除请求时点（持续学习会覆盖后门）。
- 开销：服务器与普通用户零改动；验证者侧每次验证仅 n = 30 次查询即达双错误率 < 10⁻³ 量级，无需影子模型或辅助数据。论文未报告以秒计的运行时间（未核实）。

### ③ 评测设置与主要数字
- 设置：6 数据集 × 4 架构（EMNIST/MLP、FEMNIST/CNN、CIFAR10/ResNet20、ImageNet/ResNet50、AG News/LSTM、20News/LSTM）。主表 f_user = 0.05、f_data = 50%、n = 30、α = 10⁻³。
- 非自适应服务器（Table 2）：p 在 75.43%–99.98%，q 在 0.08%–26.49%（AG News 26.49% 约为 4 类随机水平），β 低至 10⁻¹⁰–10⁻⁷⁷ 量级。
- 自适应服务器（Table 3，CIFAR10）：Neural Cleanse、Neural Attention Distillation、SPECTRE 三种后门防御只能压低 p，β 分别为 1.4×10⁻⁷、2.9×10⁻⁷、2.8×10⁻⁴，仍 < 10⁻³。但部分移除实验（Table 4）显示防御真删掉 70% 投毒样本时 β = 0.798、删 90% 时 β = 0.999，验证随之失效（机制的预期行为）。
- 成员推断基线：n = 30、α = 10⁻³ 时 β 高达 0.715–0.998，基本失效，后门方案优数个数量级。
- 局限（作者自认）：用户须能改数据且有几十个样本；不同用户触发器可能冲突碰撞；完美后门防御可使方法失效；持续学习覆盖后门；物理删除与模型外副本不在验证范围内。

### ④ 与 ProvPurge 的关系
重叠：同走“以删除后可观测行为验证删除”路线；同把验证做成探针；同强调验证须量化而非口头宣称；同考虑自适应对手规避检测。
边界（四条硬区别）：(1) 探针植入时机与对象——Athena 的探针须在训练之前由数据所有者植入训练数据，验证参数模型；ProvPurge 的探针在删除请求发生时针对已存记忆内容构造，对象是外部记忆三层存储。(2) 预承诺方式——Athena 靠触发器仅用户自知防规避，无哈希、无第三方可核验承诺，结果是当场自测的统计量，事后不可复检；ProvPurge 以删除前哈希预承诺冻结探针集，验证强制复检同一集合。(3) 证据形态——Athena 输出单一聚合量，无逐项记录、无签署；ProvPurge 回执是逐项记分卡（含部分泄漏分账）且叠加 Athena 没有的被删行缺席检查。(4) 结论形态——Athena 是双错误率下的概率推断且明确不管物理删除；ProvPurge 的提交条件是全探针失败加兄弟全过的门，验证对象正是物理缺席本身。C1 在 Athena 中无对应概念（其探针即验证者自选的一组，无“已见 vs 留出”泛化缺口）；C2 正是 Athena 的空白。
区分句（中）：「与 Athena 以用户预置后门在训练数据中、删除后以假设检验概率推断训练集成员资格不同，ProvPurge 的探针在删除请求后针对具体记忆内容构造、删除前以哈希预承诺，并与被删行缺席检查共同写入逐项签署回执，验证的是外部对话记忆的硬删除而非模型训练数据的排除。」
Distinction (EN): “Unlike Athena, which pre-plants backdoors in training data and probabilistically infers training-set exclusion via hypothesis testing, ProvPurge builds content-specific probes after a deletion request, pre-commits them by hash before deletion, and binds them to a row-absence check in a signed per-probe scorecard receipt, verifying hard deletion of external conversational memory rather than exclusion from model training.”

### ⑤ 对 ProvPurge 的启示
- 方法：借其功效分析框架给探针门定样本量预算——每类探针应报告在未删对照上的检出率（类 p）与删后残留率（类 q）及间隔，把“全失败才提交”变成有统计功效支撑的门；吸取其教训：探针可被删除方识别时对方会只对探针路径做表面处理，ProvPurge 的哈希预承诺 + 留出探针（C1）正是对位防御，写作应如此论证。
- 实验：设未删对照组测每类探针基线检出率；报告探针间冲突/碰撞率（尤其逐实体扩展后同名实体互相干扰）；做部分删除剂量—反应实验（删 50%/70%/90% 闭包时残留如何变化），直接支撑 A 的必要性；测删除后经若干轮新对话再复检的回执稳定性。
- 写作：可验证删除一节须主动让步，不得声称首个探针验证删除或首个量化验证置信度；可声称的收窄点是三元组：删除前哈希预承诺 + 逐项记分卡回执 + 存储层缺席检查，且对象是外部记忆硬删除。

### ⑥ 证据等级
全文（正式版）。PoPETs 官方 PDF 全文 §1–§8、Table 1–4，并以其 arXiv 前身（2003.04247v2）交叉核对公式。注意 arXiv 版为 5 数据集、防御仅 Neural Cleanse，本卡数字一律取 PoPETs 正式版（6 数据集、三种防御）。

---

## 卡 3 — Eisenhofer et al., Verifiable and Provably Secure Machine Unlearning, IEEE SaTML 2025

正式出处：SaTML 2025, pp. 479–496, DOI 10.1109/SATML64287.2025.00033；作者稿 arXiv:2210.09126v3（2025-03-05）。作者 Thorsten Eisenhofer、Doreen Riepel、Varun Chandrasekaran、Esha Ghosh、Olga Ohrimenko、Nicolas Papernot。

### ① 一句话主张
在恶意服务器模型下，用户无法从参数或预测判断数据是否被删，因此应由服务器给出密码学证明：预先约定的训练/遗忘函数被正确执行，且被删记录不在更新后的概念训练集中、之后不会被重新加入。

### ② 方法细节
- 期望性质：D1 可验证遗忘算法被正确执行、目标记录已移除；D2 可验证模型演化（含训练本身）；D3 已遗忘记录不得被悄悄加回。
- 框架：双方事前约定 admissible 函数三元组 f = (f_I, f_T, f_U)（初始化、训练、遗忘）及公开超参数；函数被假设为确定性（随机性由种子经 PRG 导出）。服务器维护演化的概念数据集 D_0, D_1, …，记录是带用户标识的 (u, d) 二元组。
- **证明绑定的对象**：每轮承诺为四元哈希元组 com_i =（遗忘函数状态、模型、当前训练集哈希集、已遗忘集哈希集），底层为有序集上的迭代哈希链（v3 正文为 hash chain，非早期摘要所称 Merkle 树）。
  - Proof of Training（SNARK）证明：训练函数正确执行；新增记录与已遗忘集不交；已遗忘集未变。
  - Proof of Unlearning（SNARK）证明：遗忘函数正确执行；当前训练集哈希集等于上一轮减去本轮删除集的哈希；已遗忘集只增不减（append-only）。
  - Proof of Non-Membership 不是 SNARK，而是哈希链路径：证明某记录的哈希**在已遗忘链中**，再结合 SNARK 已证的“已遗忘集与当前训练集不交”，推出该记录不在当前训练集中。
- 假设：SNARK 须满足完备性、计算可靠性与知识可靠性（可提取性）；哈希须抗碰撞（绑定性全由此来）。实例化用 Spartan 的 NIZK 变体（透明、无需可信设置）、SNARK 友好的 Poseidon 哈希、ZoKrates/CirC 电路。另有隐含假设：约定的 f 本身须被认定“语义上有意义”——证明只保证执行了约定的 f，不保证 f 语义上真能遗忘；且假设诚实用户持续验证后续更新或委托可信审计者。
- 被实例化的三种遗忘技术：重训式（exact）、Amnesiac（减去目标的直接更新，作者明言只是 approximate）、优化式（负学习率梯度上升，approximate）。**对后两种，证明成立也只证明计算被如实执行，不证明目标对参数的影响为零。**

### ③ 评测设置与主要数字
- 无准确率/遗忘效果评测，只有可行性与开销。机器：Ubuntu 22.04、256 GB RAM、2× Xeon Gold 5320。主实验：线性回归、SGD 3 epochs、batch size 1、合成数据，先证加入 100 条×10 特征，再证删除 10 条。
- 遗忘证明（Prove/Vrfy）：重训式 4m58s / 1m48s（约 810 万 R1CS 约束）；Amnesiac 2m18s / 0m49s（约 61.6 万约束）；优化式 0m53s / 0m20s（约 91.9 万约束）。非成员证明生成与验证均 < 1s。R1CS 编译时间 17s（优化式）至 48m45s（重训式）。
- 模型规模（训练证明）：线性回归约 806 万约束、Prove 4m33s；单隐层网络 N=4 约 4,203 万约束、Prove 24m16s / Vrfy 6m42s。
- 它不证明什么：不证明近似遗忘在语义/影响意义上已消除（只证执行轨迹）；不证明服务器对外推理用的就是被证的模型（须另配可验证推理，作者明言独立问题）；不证明隐私（§VII-C 明言形式化 out of scope）；不证明物理安全擦除；不涉及 LLM 与行为探针，规模止于数百条记录、4 个隐元的单隐层网络。

### ④ 与 ProvPurge 的关系
重叠：同把删除做成向数据主体可验证；同有删除前承诺思想（其承诺链 ≈ ProvPurge 的哈希预承诺）；同有“已删集合只增、禁止复加”与缺席检查（其 D_i ∩ U_i = ∅ + 非成员证明 ≈ ProvPurge 的被删行缺席检查）。
边界：(i) 证明对象不同——其证明的是参数训练数据集层面的**算法执行正确性**，ProvPurge 证明的是冻结模型外存记忆在行为探针下的**不可恢复性**；(ii) 信任模型不同——其为恶意服务器加密码学对手，安全是可忽略量级；ProvPurge 的回执是经验证据而非密码学不可伪造，除非另加签名/哈希链；(iii) 其无闭包、无逐实体扩展、无兄弟保全、无留出探针，删除单位是带 ID 的单条训练记录，与句级溯源闭包和 B 完全不交；(iv) 其框架反衬 A 的风险：D_i 是服务器自报的概念集，若源记录登记本身不完整，证明依然成立——ProvPurge 的 A 正是堵这一层。
区分句（中）：「Eisenhofer 等人证明的是预先约定的训练/遗忘函数在概念训练集层面被正确执行且被删记录不属于该集合，ProvPurge 证明的则是冻结模型外部记忆经硬删除后，在删除前哈希预承诺的攻击探针集下被删内容全数不可恢复且被删行物理缺席，前者的证明对象是算法执行轨迹，后者是行为层面的删除结果。」
Distinction (EN): “Eisenhofer et al. prove that a pre-agreed training/unlearning function was correctly executed and that the deleted record is absent from the conceptual training set, whereas ProvPurge certifies that, after hard deletion from a frozen model's external memory, the deleted content is behaviourally unrecoverable under a probe set hash-committed before deletion and that the deleted rows are physically absent—the former certifies an execution trace, the latter a behavioural deletion outcome.”

### ⑤ 对 ProvPurge 的启示
- 方法：借其承诺链升级回执——回执应链式包含上一轮回执哈希 + 已删集哈希（append-only），使“删后复加”在回执层面可检出（对应其 D3）；缺席检查可形式化为双段式：被删行 ∈ 已删清单 ∧ 已删清单 ∩ 现存三层（事实/向量/摘要）= ∅；把闭包完备性列为回执的显式假设而非隐含前提。
- 实验：回执验证开销宜仿其 Table I/II 按生成/验证/预处理三列、随实体数与探针数报告，哪怕是秒级，也以同构表格坐实“可验证性开销远低于密码学方案”；同时声明其规模上限（数百条记录、小网络）以反衬 ProvPurge 在真实记忆规模上的可行性。
- 写作：相关工作须单列 verifiable unlearning 一节，主动写明 C2 的回执可验证性弱于 SNARK 的知识可靠性、强在其可证伪的行为内容；预先回答“为何不用 SNARK”：底座冻结、无训练轨迹可证，SNARK 在此没有可绑定的 f_T/f_U。

### ⑥ 证据等级
全文。arXiv v3 全文 §I–§IX、表 I/II、附录 A–E，与 SaTML 2025 正式版同稿；版面信息经论文集目录交叉核对。注：v1/v2 摘要作 “SNARK and Merkle trees”，v3 已改为 “SNARKs and hash chains”，本卡以 v3 为准。

---

## 卡 4 — Patil, Hase & Bansal, Can Sensitive Information Be Deleted From LLMs?, ICLR 2024 Spotlight

正式出处：ICLR 2024 Spotlight；arXiv:2309.17410；代码 github.com/Vaidehi99/InfoDeletionAttacks。作者 Vaidehi Patil、Peter Hase（同等贡献）、Mohit Bansal（UNC Chapel Hill）。

### ① 一句话主张
直接编辑权重本应是最彻底的删除方式，但即便用 ROME/MEMIT 编辑到最终输出概率为零，被删事实仍可从中间隐藏状态和改述提问中被大量恢复；为若干攻击设计的防御能压低对应攻击，却不存在对所有提取攻击都有效的单一防御。

### ② 方法细节
- 威胁模型：攻击者求问答对中的答案 A；若 A 落入攻击者以预算 B 得到的候选集 C，即攻击成功。指标 AttackSuccess@B。访问分白盒（权重 + 中间隐藏状态）与黑盒（输入 + 采样输出）。删除目标为最小化 AttackSuccess 并控制知识损伤（以 Random Δ-Acc、Neighborhood Δ-Acc、Rewrite Score 分别报告）。
- 删除的两个正交维度（转述时必须分清）：**编辑优化器**（怎么改权重）——ROME 更新单个 MLP 层（GPT-J 默认第 6 层）；MEMIT 摊到多层（第 5–7 层）；附录另有 Constrained Finetuning。**删除目标**（改成什么）——基线三种：Fact Erasure（最小化 p(y|x)）、Empty Response（最大化空目标串概率，主实验用 “dummy”）、Error Injection（最大化假目标概率，作者明言实践中不可取）；本文新防御三种：Head Projection Defense（让答案跌出各层 logit lens 的 top-k）、Max-Entropy Defense（最大化各层分布熵）、Input Rephrasing Defense（把改述样本并入 Fact Erasure 并行优化，主设置 5 条改述）。
- 提取攻击：Head Projection（白盒，logit lens 取各层 top-k 并集）；Probability Delta（白盒，取相邻层分布差的 top/bottom-k，利用被删答案层间先升后降）；Input Rephrasing（黑盒，用现成改述模型生成 R 条改述、每条采样 S 次）。调参用独立 dev 集 100 条、固定 B = 20：GPT-J 上改述最优 R = 4、S = 5。

### ③ 评测设置与主要数字
- 设置：主模型 GPT-J-6B，附录 Llama-2-7B 与 GPT2-XL-1.5B；数据 CounterFact 与 zsRE；只删模型“已知”事实（生成含答案串、单 token 答案、p ≥ 0.02 等过滤），过滤后 GPT-J 样本量 CounterFact 587 条、zsRE 454 条。各编辑的 Rewrite Score 普遍 90%+、Random Δ-Acc < 1%——即按传统编辑指标“删除成功”，这是后文攻击成功的前提。
- **“38%”的精确口径（禁止笼统转述）**：不是所有删除方法的平均恢复率。其确切条件是 **GPT-J + CounterFact + ROME 编辑 + Empty Response（dummy）目标，B = 20 白盒 AttackSuccess@20 的最高值**：Head Projection 36.84%、Probability Delta 37.65%（≈38%，为摘要数字来源）、Input Rephrasing 29.02%。同设置 B = 1 时 Probability Delta 已达 18%，B = 20 后饱和。注：作者官方 slides 写“B = 10”，与正文 B = 20 不一致，引用以正文与表 1 为准。
- 最坏情形远高于 38%：MEMIT + Empty Response 在 zsRE 上 Head Projection 88.55%（正文记 89%）；Error Injection 多格 98%–99.83%。
- 防御：Max-Entropy 相对最强基线，在 CounterFact/ROME 上把 Head Projection 由 22.2% 压到 1.70%、Probability Delta 由 37.65% 压到 2.39%；但对 MEMIT 弱得多。未预见攻击实验（§7.3）：为 Head Projection 设计的防御挡不住 Probability Delta（zsRE/ROME 高达 56.83%）——这是“对已见攻击调防、对未见攻击失效”的直接实证。Input Rephrasing Defense 在任何组合下不优于基线，仅当攻击改述与防御所用改述完全相同时才有效；强行激进编辑压黑盒会使 Neighborhood Δ-Acc 升至 27.8。
- 目标串对比（附录表 6）：dummy 优于 “I don't know”（后者三攻击 54.89%/55.92%/32.80%）。

### ④ 与 ProvPurge 的关系
重叠：同以“删除后能否被恢复”而非“表面输出是否消失”为判据；其改述攻击 ≈ ProvPurge 的改述/攻击式探针；其“为一种攻击设计的防御挡不住未预见攻击”正是 C1 留出探针泛化缺口要量化的现象；其 Random/Neighborhood Δ-Acc ≈ 兄弟保全探针的先行形态。
边界：(i) 删除对象不同——其删的是权重中的预训练事实，底座被实质改动；ProvPurge 底座冻结，删的是外存对话记忆。(ii) 恢复通道不同——其最强攻击依赖白盒隐藏状态；在冻结底座 + 外存架构下，隐藏状态里至多有底座预训练知识、不是已删对话记录，该通道对记忆内容不适用；ProvPurge 的对应通道是残余直喂与多跳推导。引用 38% 必须带全条件，不得写成“删除后普遍有 38% 可恢复”。(iii) 其无删除事务、无预承诺、无回执、无闭包：每次编辑一个孤立事实，不处理来源句、派生结论、摘要重述之间的溯源关系——这正是闭包 + B 的位置。(iv) 其防御均为训练时目标，ProvPurge 的探针门是提交时门控，时点不同。
区分句（中）：「Patil 等人表明权重编辑后的 LLM 事实仍可经中间隐藏状态与改述提问被恢复，ProvPurge 处理的则不是参数化知识，而是冻结底座之外对话记忆的溯源闭包，其删除以硬删除后全套预承诺探针失败、且留出探针复检通过为提交条件。」
Distinction (EN): “Patil et al. show that facts edited out of LLM weights remain recoverable from intermediate hidden states and rephrased prompts, whereas ProvPurge deletes not parametric knowledge but the provenance closure of conversational memory external to a frozen model, committing a deletion only when the full pre-committed probe set fails after hard deletion and held-out probes pass re-verification.”

### ⑤ 对 ProvPurge 的启示
- 方法：“痕迹不在最终输出而在中间表示”在外存架构下对应“痕迹不在事实层而在向量/摘要层”，支撑三层硬删须逐层设探针；“编辑对原问法有效、对改述失效”对应逐实体扩展须覆盖同一事实的改述问法，且提交门要求改述探针与原问法探针同时失败。威胁模型中应显式声明白盒通道的排除理由，避免审稿人以 Patil 的白盒结果质疑完备性。
- 实验：C1 可仿其 §7.3“未预见攻击”协议——用探针子集 A 做提交门、留出子集 B 事后评测，报告 A 通过而 B 泄漏的比例；报告纪律学其口径：每个恢复率数字绑定（模型、数据、删除方法、探针类型、预算）五元组，并报告探针预算（采样次数/改述条数）对恢复率的影响曲线；兄弟保全可吸收 Random vs Neighborhood 两级分层（随机无关事实 vs 同实体邻居事实），后者更接近 B 的源—结论场景。
- 写作：引用时写全“GPT-J、CounterFact、ROME+Empty Response、B = 20 白盒最高值（Probability Delta 37.65%）”，并可同时引 88.55% 上界说明“传统编辑指标 Rewrite Score 90%+ 与可恢复性可以完全脱钩”，以此论证 ProvPurge 为何弃编辑成功率、改用探针记分卡作回执内容；不得把该文写成“微调删除无效”的证据——其主结果是 ROME/MEMIT 编辑，FT 仅在附录且其防御崩坏源于编辑本身失效。

### ⑥ 证据等级
全文。ICLR 2024 官方论文集 PDF 正文 §1–§8、表 1 及附录 A–E；Spotlight 属性与作者信息经 ICLR slides 与 UNC 官方新闻交叉核对。slides 的 B = 10 与正文冲突一处已标注，以正文为准。

---

## 卡 5 — MEXTRA / Unveiling Privacy Risks in LLM Agent Memory, ACL 2025 Long

正式出处：Bo Wang、Weiyi He、Shenglai Zeng、Zhen Xiang、Yue Xing、Jiliang Tang、Pengfei He, ACL 2025 Long Papers, Anthology ID 2025.acl-long.1227, pp. 25241–25260, DOI 10.18653/v1/2025.acl-long.1227。MEXTRA 是文中方法名，不是题名的一部分。

### ① 一句话主张
在仅能以输入查询交互的黑盒设定下，用“定位被检索记录 + 对齐 Agent 工作流输出格式”的攻击提示及自动多样化生成，可以成批量地把存储在 LLM Agent 长期记忆中的历史用户查询抽取出来，且泄漏量随记忆配置与攻击者先验知识显著变化。

### ② 方法细节
- 目标记忆与工作流：记忆存 m 条（历史用户查询 q_i，Agent 生成的解 s_i）；新查询以相似度函数取 top-k 记录作 in-context demonstrations 生成解，成功执行的新对可被选择性写回。评测为 static memory（评测期间记忆不变）。
- 威胁模型：黑盒攻击者只能发输入查询，目标是最大化 n 条攻击提示抽出的不重复历史查询集合的大小。知识分两级：basic 只知 Agent 的应用域与任务；advanced 还通过探索性交互推断出相似度函数的类型（余弦语义 vs 编辑距离格式）。
- 攻击提示设计（核心）：每条提示 = locator ‖ aligner。locator 明确点名要“被检索到的历史示例查询”并要求其输出优先于完成原任务；aligner 指定与工作流兼容的输出通道（如 Web Agent 的搜索框、EHRAgent 的 answer 字段）。作者论证朴素提示（“repeat all the context” 等）失败：定位不到记忆记录，或与非文本输出工作流不兼容。
- 自动生成：以 GPT-4 为生成器，每次生成 10 条、重复至 n 条，多样性只靠改写措辞。advanced 版按推断出的相似度函数定制：编辑距离型系统地改变提示长度（如 EHRAgent 30→230、间隔 20）；余弦型生成领域词/短语前置拼接以改变语义检索命中面。附录：每条攻击提示最多跑 3 次以控制随机性。

### ③ 评测设置与主要数字
- 目标系统：EHRAgent（代码型 EHR Agent，默认编辑距离检索 top-4，记忆查询采自 MIMIC-III 验证集）与 RAP/Web Agent（WebShop，SBERT-MiniLM 余弦检索 top-3）。默认底座 GPT-4o，默认 m = 200。附录另有自建 QA-Agent 泛化验证。
- 指标：EN（抽出的不重复查询数）、RN（被检索到的不重复数）、EE = EN/(n×k)、CER（完整抽出全部 k 条的攻击占比）、AER（至少抽出 1 条的攻击占比）。
- 主结果（n = 30、m = 200、仅 basic 知识）：EHRAgent — EN 50、RN 55、EE 0.42、CER 0.83、AER 0.83；RAP — EN 26、RN 27、EE 0.29、CER 0.87、AER 0.90。CER ≈ AER 说明一旦命中多为整组抽出。
- 消融（EN）：去掉 aligner 后 EHRAgent 50→36、RAP 26→6（Web 工作流对输出格式最敏感）；去掉生成要求 39/25；去掉示例 29/8。
- 配置因素：打分函数影响最大，编辑距离在所有 m 下比余弦更易泄漏（m 从 50 增至 500，EHRAgent 编辑距离 EN 31→59，余弦仅 13–36）；嵌入模型影响小且无一致趋势；m 与 k 增大均使泄漏上升。RAP 换底座：GPT-4 EN 23、GPT-4o 26、Llama3-70b 17（但 Llama3-70b 本身在 RAP 原任务成功率仅约 8%，结果混杂工作流可用性，不可读作更安全）。
- 攻击者因素：n 从 10 增至 50，EN/RN 持续上升无明显放缓；n = 50 时编辑距离 Agent 泄漏超过记忆的 30%、余弦 Agent 超过 10%。advanced 版几乎处处优于 basic 版（n = 50 时 RAP 的 RN：编辑距离 58→79、余弦 35→84）。
- 防御：仅附录讨论、未做实验——输入/输出控制与 memory sanitation（入库前去标识化），作者自述前者可被伪装成正常语境的攻击提示绕过、后者削弱 demonstration 效用。局限：仅单 Agent、无会话/用户级隔离。

### ④ 与 ProvPurge 的关系
重叠：(a) 都把“防护之后仍能被自然语言查询问出来”视为记忆仍可达的实证标准；(b) MEXTRA 的攻击式提示正是 ProvPurge 五类探针中“攻击式”一类的现成构造范式（locator 点名目标 + aligner 借合法输出通道外带）；(c) 其 advanced 条件（攻击者已知检索打分方式）对应 ProvPurge 应设的最强探针档。
边界：(1) 目标函数相反——MEXTRA 是进攻方、目标未知、批量最大化抽出量，抽到任意一条即计分；ProvPurge 探针是防御方、目标特定（某个被删实体及其闭包）、删除前哈希预承诺，须全部失败且兄弟保全全部通过才提交。(2) 可达通道不同——MEXTRA 只能触及被 top-k 检索进 demonstrations 的记录，“没抽到”可能只是检索未命中而非内容不存在，故 MEXTRA 式失败不能充当缺席证据；ProvPurge 还覆盖残余直喂、多跳推导、过时前提等不经检索层的通道，并以被删行缺席检查兜底。(3) 对象不同——MEXTRA 的记忆是单一 demonstration 存储，无分层、无删除事务、无闭包、无 F4 语义、无回执。(4) 其评测中记忆全程静态、无删除动作发生。
区分句（中）：「MEXTRA 是面向未知目标的批量记忆提取攻击，以抽出多少条他人历史查询计成功；ProvPurge 的攻击式探针则是删除前以哈希预承诺、面向特定被删实体的防御性验证，须全部失败且兄弟保全探针全部通过才允许提交，二者的目标函数与成功判据正好相反。」
Distinction (EN): “MEXTRA is an offensive, target-agnostic bulk-extraction attack scored by how many stored user queries it leaks, whereas ProvPurge's adversarial probes are defensive, target-specific, hash-precommitted before deletion, and must all fail—while sibling-preservation probes all pass—for the deletion to commit.”

### ⑤ 对 ProvPurge 的启示
- 方法：攻击式探针按 locator + aligner 两段式构造，并为每种输出通道（正常回答、工具入参、摘要引用）各备 aligner 变体；设 advanced 档探针（假定验证者已知本系统检索打分与嵌入方式），作攻击侧最坏情形设定；牢记单通道探针失败 ≠ 缺席，缺席结论只能由多通道探针 + 缺席检查共同支撑。
- 实验：借 EN/RN/CER/AER 的分解给记分卡加“失败归因”维度——区分探针失败是因目标未被检索到、被检索到但未被输出、还是被输出为改写残余（对应部分泄漏记账）；报告泄漏量随探针预算 n 的曲线（MEXTRA 显示 n 10→50 无饱和），C1 泛化缺口按探针数量分层报告；每条探针多次运行（如其最多 3 次）控制底座随机性，运行次数写进回执。
- 写作：引 MEXTRA 作风险动机与攻击探针的构造来源，但措辞须限定为“demonstration 型 (q, s) 记忆在黑盒下可被批量抽取”，不得引申为它验证过任何删除方法；其防御部分只是未实验的讨论，引用须注明。

### ⑥ 证据等级
全文。ACL Anthology 条目核正式题名/作者/页码/DOI；arXiv HTML v2（2502.13172v2）通读正文、Table 1–4 与附录 A–C，数字均出自全文表格，无未核实项。

---

## 卡 6 — MemoryAgentBench, ICLR 2026

正式题名：Evaluating Memory in LLM Agents via Incremental Multi-Turn Interactions（MemoryAgentBench 是基准名，不是题名）。作者 Yuanzhe Hu、Yu Wang、Julian McAuley（UC San Diego）。ICLR 2026（OpenReview id: DT7JyQC3MR，2026-01-26 发表）；arXiv:2507.05257。**版本注意**：数字以 arXiv v4 camera-ready（2026-06-28）为准，v4 较投稿版新增 GPT-5-mini，且多跳上限表述由投稿版 “at most 7%” 改为 “at most 28%”，引用勿混用两版。

### ① 一句话主张
把长上下文数据集改造成增量多轮注入格式并新造 EventQA 与 FactConsolidation，MemoryAgentBench 首次在同一协议下同时测记忆 Agent 的准确检索、测试时学习、长程理解与选择性遗忘四种能力，并显示现有系统在四者上均未过关、尤以其选择性遗忘（冲突消解）为甚。

### ② 方法细节（聚焦 Selective Forgetting）
- 能力框架：Accurate Retrieval、Test-Time Learning、Long-Range Understanding、Selective Forgetting (SF)。SF 在 §1 定义为 “revise, overwrite, or remove previously stored information when faced with contradictory evidence”；附录 B.4.1 正式定义为 “the agent's ability to detect and resolve contradictions between out of date knowledge and newly acquired information”。
- 统一格式：所有数据标准化为分块序列 + 问题 + 答案，每块包装成带“请记住”指令的对话逐块注入、增量更新记忆，全部注入后统一提问。全基准 2,071 题，上下文深度 103k–1.44M tokens。
- FactConsolidation 构造（SF 唯一数据集）：取 MQUAKE 的反事实编辑对（一条真实事实 + 一条改写的矛盾版本），给每句编号，**过时事实编号较小、新事实（含正确答案）编号较大**，按编号拼成 6K/32K/64K/262K 长上下文（主表 262K）。分 FactConsolidation-SH（单跳直接回忆）与 FactConsolidation-MH（多跳推理）两组，各 1 个序列 : 100 题。
- 作答规则（关键）：提示明确告知以最终记忆状态为准、冲突时优先后出现信息；v4 §3.3 写明 SF 的 explicit guardrails——明示“facts are indexed by serial numbers, and newer facts have larger serial numbers”，要求以找到最新事实消解冲突。旧事实**不被移除**，仍完整保留在输入序列与记忆池中。
- 被测系统：长上下文 Agent（GPT-4o、GPT-5-mini、Claude-3.7-Sonnet 等）、RAG Agent（BM25、Contriever、Text-Embed-3、HippoRAG-v2、Mem0、Cognee、Zep 等）、Agentic Memory Agent（Self-RAG、MemGPT、MIRIX）。SF 任务 chunk size 512（部分系统 4096），主表 retrieval top-k = 10。
- 指标：FC 用 Accuracy，具体为 SubEM（子串精确匹配）——只测最终答案是否取自新事实，不检查旧事实是否仍存储、能否被其他问法问出。

### ③ 评测设置与主要数字（v4 Table 3，FC 上下文 262K）
- SF 结果（FC-SH / FC-MH）：GPT-5-mini 78.0 / 28.0（SF 均分 53.0，全表总分最高）；GPT-4o 60.0 / 5.0；Claude-3.7-Sonnet 43.0 / 2.0；HippoRAG-v2 54.0 / 5.0（非长上下文系统中 SH 最高）；BM25 48.0 / 3.0；MemGPT 28.0 / 3.0；Mem0 18.0 / 2.0；Zep 7.0 / 3.0；MemoRAG 与 Contriever 的 MH 7.0 为非长上下文系统并列最高。
- 作者结论：所有方法在多跳上失败（v4 表述 at most 28%，且该 28% 仅由 GPT-5-mini 取得，其余系统 ≤ 7%）；只有长上下文 Agent 在单跳上达到“fairly reasonable”。
- 可解性验证（Table 5，缩短上下文）：GPT-4o SH 6K 92.0 / 32K 88.0，MH 6K 28.0 / 32K 10.0；o4-mini SH 6K 100.0 / 32K 61.0，MH 6K 80.0 / 32K 14.0。瓶颈在长程冲突处理而非题目本身。
- **SF 语义的最终确认**：全流程没有删除请求、没有请求方指定的删除目标、没有删除操作、没有删除后验证；“成功”的唯一含义是冲突时答出新事实。一个把全部新旧事实都保留、仅按编号/时间取最新者作答的系统即可得高分——本质是 last-write-wins 冲突消解，与“按请求删除并证明缺席”是不同任务。此点由其定义、构造（旧事实保留在序列内）与提示规则三处原文共同确立。

### ④ 与 ProvPurge 的关系
重叠：(a) 都关注“过时信息不应再支配回答”这一行为层目标；(b) 其 FC-MH 与 ProvPurge 的多跳推导探针同形；(c) 都强调长序列下问题急剧变难；(d) 它是已发表工作中唯一以 selective forgetting 命名一个维度的基准，ProvPurge 必须主动区分，否则易被误认为同题。
边界：(1) 触发不同——其“遗忘”由数据内部新事实自动触发，无用户的删除请求，系统不需识别任何请求；ProvPurge 由系统自身从自然语言识别删除请求并执行。(2) 目标状态相反——其新事实必须保留并用作答案，旧事实只需在作答时被压制、可继续留存；ProvPurge 的被删目标必须三层硬删除且不可恢复，无辜兄弟内容须保全（其无兄弟保全概念）。(3) F4/B 语义缺位——其新旧事实是平行替代关系，无源—结论派生结构，A 与 B 在此完全无对应物。(4) 验证缺位——只有最终答案的 SubEM，无预承诺、无探针集、无缺席检查、无回执，答对不能证明任何东西被删掉。(5) 无攻击面——问题均为善意 QA，不含攻击式/残余直喂探针。
区分句（中）：「MemoryAgentBench 的选择性遗忘测的是新旧事实冲突时以新事实为准作答——FactConsolidation 中旧事实仍保留在输入序列内、仅被赋予较小序号，成功意味着按序号取最新事实；而 ProvPurge 处理的是用户明确提出删除请求后，被删内容须在事实、向量与摘要三层被硬删除、且经删除前哈希预承诺的探针集验证不可恢复，前者是冲突消解，后者是按请求删除与可验证缺席。」
Distinction (EN): “MemoryAgentBench's selective forgetting evaluates last-write-wins conflict resolution—superseded facts remain in the input and success means answering from the newest fact—whereas ProvPurge addresses deletion on explicit request, requiring hard removal across the fact, vector and summary layers and hash-precommitted probes that certify the target's unrecoverability.”

### ⑤ 对 ProvPurge 的启示
- 方法：评测与回执应显式分离两类失败——“冲突消解失败”（有更新事实却答旧值，MAB 型）与“删除失败”（无更新事实、目标本应缺席却仍可答出，ProvPurge 型），避免审稿人用 MAB 分数质疑删除有效性；过时前提探针可借 FC 的编号化构造生成（旧事实在前、问题预设旧状态），但须改造成删除后场景。
- 实验：长度是强混杂变量——o4-mini 的 MH 从 6K 的 80.0 跌到 32K 的 14.0，提示 C1 泛化缺口与探针结果应按记忆长度分层报告（短/中/长三档）；多跳是最难一档，探针记分卡中多跳应单独成栏，不与单跳合并平均。
- 写作：相关工作设一节“同名异义”澄清，引其 B.4.1 定义原文作据；切勿宣称“首个测选择性遗忘的基准/工作”——该名已被 MAB 占用；新颖性应落在“按请求删除 + 预承诺探针 + 回执”三元组上。（注：其早期版本曾以 conflict resolution 命名第四能力，此点据 arXiv v1 摘要检索结果，正文未逐字核对，正式引用前再核 v1。）

### ⑥ 证据等级
全文（含附录），且分版本核对。OpenReview 投稿 PDF 通读正文与附录 B.4/D/E；arXiv v4 camera-ready 复核主数字；OpenReview 元数据核作者与发表日期。唯一标注未逐字核对者：早期版本命名一事，已在上文标明。

---

## 汇总一：定位句修订建议（可直接用于引言与相关工作）

以下 5 句按“先划界、再声称”的顺序排列；每句标注其对应的边界来源。使用总原则：不得声称“首个外部存储遗忘”（Wang TDSC 已占）、“首个探针验证删除”（Athena 已占）、“首个可验证删除”（Eisenhofer 已占）、“首个测选择性遗忘的基准”（MemoryAgentBench 已占）；可声称的始终只有三元组组合本身。

**定位句 1（主定位，对应 Wang + Athena + Eisenhofer 三条边界）**
「ProvPurge 的位置是既有工作之间一个未被占据的格子：与 Wang 等人的“不删而行为抑制”不同，它对外部对话记忆执行溯源闭包硬删除；与 Athena 的后门统计推断和 Eisenhofer 等人的执行轨迹证明不同，它的证据是删除前哈希预承诺的逐项探针记分卡加被删行缺席检查，验证对象是冻结底座之外记忆内容的行为不可恢复性。」
EN: “ProvPurge occupies a cell left open by prior work: unlike Wang et al.'s retention-plus-suppression, it hard-deletes a provenance closure from external conversational memory; unlike Athena's backdoor-based statistical inference and Eisenhofer et al.'s execution-trace proofs, its evidence is a per-probe scorecard—hash-committed before deletion—plus a row-absence check, certifying the behavioural unrecoverability of memory content outside a frozen model.”

**定位句 2（删除语义，对应 MemoryAgentBench 边界）**
「本文所称选择性遗忘是按请求删除（deletion on request），不是新旧事实冲突时的 last-write-wins 消解：目标须在事实、向量、摘要三层硬删除并经预承诺探针验证缺席，无辜的兄弟内容则须由保全探针证明未受损伤。」
EN: “Selective forgetting in this paper means deletion on explicit request, not last-write-wins resolution of conflicting facts: the target must be hard-deleted across the fact, vector and summary layers and certified absent by pre-committed probes, while innocent sibling content must be shown intact by preservation probes.”

**定位句 3（证据强度，对应 Eisenhofer 与 Athena 的强度边界）**
「ProvPurge 的回执不提供密码学意义上不可伪造的删除证明，也不以错误率参数化的统计推断代替核查；它提供的是可证伪的经验证据——任何验证者重跑同一预承诺探针集并执行缺席检查，即可复现或推翻回执结论。」
EN: “A ProvPurge receipt offers neither a cryptographically unforgeable proof of deletion nor an error-parameterised statistical inference in place of checking; it offers falsifiable empirical evidence—any verifier re-running the same pre-committed probe set and the absence check can reproduce or refute the receipt's conclusion.”

**定位句 4（动机，对应 Patil 与 MemCut 的实证）**
「本文弃用“删除动作已执行”与“单次输出已消失”作为遗忘完成的判据，因为已发表证据显示传统编辑指标可与可恢复性完全脱钩（Patil 等人：Rewrite Score 90% 以上时，白盒恢复率最高仍达 88.55%）；遗忘完成在本工作中只由预承诺探针集的逐项失败来定义。」
EN: “We reject 'the deletion routine ran' and 'a single output vanished' as criteria for completed forgetting, because published evidence shows editing metrics can fully decouple from recoverability (Patil et al.: white-box recovery up to 88.55% while Rewrite Scores exceed 90%); here, forgetting is complete only as defined by the itemised failure of a pre-committed probe set.”

**定位句 5（威胁模型，对应 MEXTRA 与 Patil 的通道划分）**
「本文的攻击式探针沿用已发表记忆提取攻击的构造（定位加输出对齐，MEXTRA），但把目标函数反转：从批量抽取未知内容改为验证特定被删目标在原生、直喂、多跳、过时前提与攻击式五种通道下均不可恢复；权重内部状态通道因底座冻结且从未写入对话记忆而被显式排除在威胁模型之外。」
EN: “Our adversarial probes reuse the construction of published memory-extraction attacks (locating plus output alignment, MEXTRA) with the objective inverted—from bulk-extracting unknown content to certifying that a specific deleted target is unrecoverable across native, direct-feed, multi-hop, stale-premise and adversarial channels; the weight-internal channel is explicitly excluded because the frozen backbone is never written with conversational memory.”

---

## 汇总二：规格书是否需要改（建议清单，不改动规格书本身）

以下均为建议，供 Joseph 定夺；标注者与上文定位句存在对应关系，若采纳定位句则建议同步采纳。

1. **回执链式化**：回执增加 prev_receipt_hash 与 deleted_set_hash 两个字段，形成 append-only 链，使“删后复加”在回执层面可检出。
2. **缺席检查双段式形式化**：规格书 §9 把缺席检查写成“被删行 ∈ 已删清单 ∧ 已删清单 ∩ 现存三层 = ∅”的显式公式，并把闭包完备性列为回执的显式假设字段（assumptions），呼应 A。
3. **探针功效报告**：回执记分卡或实验章节增加每类探针在未删对照上的检出率与删后残留率（类 p/q）及间隔；提交门的“全失败”以此为功效背书。建议同时增加探针间冲突/碰撞率的报告口径。
4. **C1 协议细化**：留出探针协议写明“提交门探针子集 A / 留出子集 B”的划分与 B 的保密方式；实验计划增加部分删除剂量—反应扫描（删闭包的 50%/70%/90% 时的残留曲线），作为 A 的必要性证据。
5. **记分卡增加两个维度**：一是失败归因（未检索到 / 检索到未输出 / 输出为改写残余），二是按记忆长度分层（短/中/长）与多跳单独成栏；每条探针的运行次数（如 3 次）写入回执字段。
6. **删除候选清单的处置规则**：规格书中声明候选清单在删除完成后销毁或仅保留哈希，防止“为了删除而再复制一份秘密”。
7. **回执验证开销报告格式**：实验计划预留一张生成/验证/预处理开销表，随实体数与探针数分档，为“远低于密码学方案”的对比留出位置。
8. **不需要改的（明确）**：冻结底座定位、三层硬删、五类探针构成、双通道评分、逐实体无截断扩展与摘要重建定序，本轮 6 篇精读无一篇构成推翻理由；C1+C2 合并方案在本轮证据下进一步得到支撑（Patil §7.3 是 C1 现象的已发表实证、Athena 与 LETHE 之间的空位是 C2 的位置），仍待 Joseph 确认后再生效。

---

## 附：6 篇一句话速览

- Wang TDSC：不删而加保密条目做行为抑制，目标知识全保留——ProvPurge 的“加法 vs 减法”边界。
- Athena：训练前植后门、删除后统计推断训练集排除——探针验证的先行者，但不管物理删除、无预承诺哈希、无逐项证据。
- Eisenhofer：SNARK 证明约定遗忘函数被正确执行且记录不在概念训练集——证明的是执行轨迹，不是语义消除，开销为分钟级、规模止于小模型。
- Patil：编辑指标满分与可恢复性脱钩，白盒恢复最高 88.55%（“38%”只是 GPT-J/CounterFact/ROME+Empty/B=20 的格值）——ProvPurge 弃编辑成功率、用探针记分卡的实证依据。
- MEXTRA：黑盒批量抽取 Agent 记忆（n=30 时 EN 50/26）——攻击式探针的构造来源，但其“抽不到”不能充当缺席证据。
- MemoryAgentBench：其 selective forgetting 是旧事实保留前提下的 last-write-wins 冲突消解（FC-MH 非长上下文系统 ≤7%）——同名异题，须主动区分。
