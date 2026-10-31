# ProvPurge 创新点定稿与代码实现方案

> **文档性质**：本文是创新点与实现设计方案，不是代码，不是实验报告，也不报告任何结果。
>
> **开工门槛**：本文完成后，仍须先向 Joseph 做阶段汇报并得到明确点头，才可开始仓库骨架、模块实现或单元测试。任何真实评测运行——包括试跑、基准子集运行和正式实验——属于实验阶段，须另行单独点头；不得以“只是调试”“只是冒烟”为由自行启动。
>
> **依据与优先级**：本文以《ProvPurge 方法规格书》为最高依据，并吸收《人家是怎么发论文的》第三部分的创新表述与《顶会顶刊已发表论文补查报告》的结论。若本文与规格书不一致，以规格书为准；需要改变规格时，先修订规格书并留下决议记录，再改实现方案和代码。
>
> **状态标记**：
> - **已定版**：已经进入规格书或由 Joseph 采纳，不在实现阶段重新讨论。
> - **当前推荐版**：论证已经完整，但最终取舍仍待 Joseph 确认；确认前不得写成已冻结的论文贡献。
> - **待试跑定版**：只能在获准试跑并测得分布后冻结，本文只给占位参数和定版方法。

---

# 第一部分 创新点定稿

## 1.1 总体定位：只声称三元组，不声称任何单件首创

ProvPurge 面向**冻结底座模型上的 LLM Agent 对话记忆**，研究显式遗忘与知识更新请求下的选择性遗忘。遗忘对象不在模型权重中，而在事实层、向量层和摘要层组成的外部记忆中。因此，本文不走参数遗忘路线，不训练底座模型，也不把来源图、实体扩展、探针、签名或回执中的任何单件单独宣称为首创。

可以声称的核心只有下面这个组合：

> **「溯源闭包硬删 + 探针门控提交 + 探针记分卡回执」在 Agent 对话记忆上的事务化组合。**

创新点的一句话定稿为：

> ProvPurge 将 LLM Agent 对话记忆的选择性遗忘定义为一个探针门控的硬删除事务：系统以句级溯源闭包与实体扩展生成候选，在事实、向量与摘要三层硬删后，只有当删除前哈希预承诺的攻击探针全部无法恢复目标且兄弟保全探针全部通过时才允许提交，并签发签署了逐项探针成绩单的回执。

以下表述禁止使用：

- 不得声称“首个外部存储遗忘”：Wang et al.（IEEE TDSC）等已发表工作已经占据外部存储侧遗忘的一般空间。
- 不得声称“首个可验证删除”或“首个探针验证删除”：Athena（PoPETs 2022）已经发表探针式验证，Eisenhofer et al.（IEEE SaTML 2025）已经发表密码学可验证遗忘。
- 不得声称“首个删除回执”：LETHE 已经签发删除回执；ProvPurge 的差别在回执对象和验证内容，而不是“有没有回执”。
- 不得声称来源图、句级溯源或实体扩展本身首创：PolyGraph、GNNDelete 等工作已经构成谱系。
- 不得声称 $(\varepsilon,\delta)$ 认证删除、分布不可区分或密码学意义上的“证明已删除”。ProvPurge 回执提供的是**经验性行为证据 + 内容缺席检查 + 防篡改签名**。
- 不得以“闭包完整”作为删除完备性的前提。闭包只负责生成候选，来源可以不完备；最终是否删净由探针门裁决。

论文的贡献结构应写成“一个主贡献、两个随主贡献成立的子主张、一个待确认的第二贡献”，而不是把每个模块都包装成独立创新：

1. **主贡献**：探针门控的硬删除事务三元组。
2. **子主张 A**：不完整来源下，闭包删除必须接受完备性检查和探针门拒提交约束。
3. **子主张 B**：F4 必须区分“删结论保来源”与“删来源并使结论塌陷”两种语义。
4. **第二贡献（当前推荐版）**：可证伪的删除证明——以 C1 留出探针泛化缺口为纲，以 C2 回执可验证性评测为目。
5. **定位性决策**：LoRA 不进主方法；只保留为触发识别的可选消融，并受写死的启用条件约束。

## 1.2 贡献一（主贡献）：探针门控的硬删除事务

### 主张一句话

ProvPurge 把“执行删除动作”改成“满足提交条件才生效的删除事务”：溯源闭包与实体扩展生成候选，三层硬删后，只有预承诺攻击探针全部无法恢复目标、无关记忆保全探针全部通过且来源完备性不再触发时，事务才可提交并签发探针记分卡回执。

### 新在哪

- **与 Wang et al.（IEEE TDSC）的边界**：Wang 的路线是在外部知识库中写入可检索的保密指令，以行为抑制实现遗忘，不删除存储内容；其效力依赖指令不泄露。ProvPurge 删除的是事实、向量和摘要三层中的实际存储对象，并以删后探针结果决定事务是否提交。两者同为“不动参数、在外部存储处理遗忘”，但一个是输出抑制，一个是存储层硬删加验证。
- **与 Athena（PoPETs 2022）的边界**：Athena 用预置后门样本和假设检验审计第三方是否删除训练数据。ProvPurge 的探针针对当前 Agent 记忆内容，由系统在删除前预承诺；探针结果不是事后审计意见，而是自系统删除事务的**提交条件**。
- **与 Eisenhofer et al.（IEEE SaTML 2025）的边界**：Eisenhofer 的 proof of unlearning 绑定训练集成员关系，给出密码学证明。ProvPurge 不证明成员关系，也不作密码学删除断言；它签署的是逐项行为探针成绩单，并要求被删行缺席检查和探针复检。
- **与 LETHE 的边界**：LETHE 的回执签署删除事件、删行清单与事件日志完整性；其代码中的验证不检查被删行是否仍在库中，也不复查行为可恢复性。ProvPurge 的回执签署逐项探针结果、兄弟保全结果、逐层删除哈希、覆盖数和部分泄漏计数，验证必须包含被删行缺席检查与探针复检。
- **与 PolyGraph 的边界**：PolyGraph 的句级溯源与逐句回执解决的是记忆内容从何而来、回执是否忠实的问题。ProvPurge 借鉴句级粒度，但溯源图在这里是删除候选生成器，回执在这里是删除效果成绩单；两者的问题不同。
- **真正的新意落在提交语义**：不是“多了一个探针模块”，而是删除第一次有了明确的 commit 条件、失败状态和自动扩候选回路。探针不过，事务不能被记为成功；达到上限仍不过，必须签 `failed` 回执并计入分母。

### 靠什么实验证明

- BEMULA 主表：在同一底座、同一嵌入模型和同一服务条件下重跑主要基线，报告官方 RPR 与 Score。
- “去探针门”消融：删除一次即提交，与完整事务比较残余恢复、RPR、Score、兄弟保全和成本。
- “去来源图”“去实体扩展”和池宽 20/40/60 消融：分别证明闭包、实体覆盖和候选范围各自的作用，避免主贡献被指为模块拼凑。
- 探针轮次分布、候选池大小、实际删除数、覆盖数 $k$、模型调用量和 `failed` 事务数量必须与效果同时报告，证明事务不是靠无限调用换结果。
- MemCut residual 直喂辅证：在自研复刻的 residual 协议下单独报告残余恢复，不与 BEMULA 主表混算。

### 什么结果会证伪它

出现以下任一结果，主贡献就不能按当前强度宣称：

- 在同底座配对比较下，ProvPurge 相对最强基线的 RPR/Score 差异不明确，或配对差小于规格书规定的 5 个百分点却仍被写成“更优”。
- “去探针门”消融与完整方法在残余恢复上没有实质差别，说明探针门没有带来可测收益。
- 探针门通过的事务仍能被独立探针、残余直喂或多跳探针以与未过门事务相近的比例恢复目标，说明提交条件没有判别力。
- 效果提升只来自不成比例的额外调用，且在冻结的轮次与成本上限内无法复现，说明事务代价不可接受。

### 对应论文哪一节

- 方法章的事务总览、候选生成、三层硬删、探针门和回执小节。
- 实验章的 BEMULA 主表、五项既定消融、成本与轮次分析。
- 相关工作中“外部存储遗忘”和“可验证遗忘”两个坐标小节只负责划界，不重复声称主贡献。

## 1.3 贡献二（A）：不完整来源下的闭包删除与拒提交规则

### 主张一句话

当来源锚缺失、抽取置信不足或闭包与检索不一致时，请求必须被标记为 `provenance-incomplete`；在此状态下，即使表面探针全过，探针门也拒绝提交，系统必须继续扩候选、重删和复检，直到完备性问题消失或达到上限后如实记为 `failed`。

### 新在哪

- **与 AfterDeleteBench 的边界**：AfterDeleteBench 的组合重构与闭包结论建立在来源记录完整的理想条件上。ProvPurge 明确处理其附录自承未覆盖的来源不完备情形，并把口径收窄为“来源不完备条件下仍以探针结果为准绳的闭包删除”。
- **与 PolyGraph 的边界**：句级溯源能提高候选生成的可解释性，但不能保证抽取没有漏项、来源锚没有缺失。A 不把溯源图当完备性证书，而是给它加完备性检查和失败出口。
- **与 MemCut 的关系**：MemCut 表明残余恢复的关键是证据覆盖与 $k$ 可达性，且证据的相似度排名可散至 3–141。A 因此禁止用相似度 top-$k$ 截断实体扩展，并要求逐个中间实体展开、逐实体统计 1-hop 覆盖数。
- **与普通“多删一点”的区别**：A 的核心不是扩大删除集，而是改变提交规则：来源不完备时，系统没有资格仅凭一轮表面全过就签 `verified`。

### 靠什么实验证明

- 按 `provenance-incomplete` 标记分层报告 BEMULA 结果，重点看 F2、F4 分项中该标记的分布、轮次、最终通过与 `failed` 情况。
- 比较“普通闭包”“闭包 + 完备性检查但允许提交”“闭包 + 完备性检查 + 拒提交扩候选”三种设置，检验拒提交规则是否降低残余恢复。
- 在 MemCut residual 直喂辅证中单独报告不完备来源请求的残余恢复变化。
- 报告逐实体 1-hop 覆盖数、候选池大小、实际删除数与覆盖数 $k$，说明改进来自覆盖而不是无差别扩大删除预算。

### 什么结果会证伪它

- `provenance-incomplete` 请求与来源完备请求在残余恢复、探针失败率上没有系统差异，说明完备性标记没有识别出真实风险。
- 加入完备性检查和拒提交规则后，F2/F4 与 residual 直喂结果不优于普通闭包，或改进只表现为大量事务被拖到 `failed`，说明规则只有保守性、没有解决能力。
- 逐实体覆盖提高后残余恢复不下降，说明 A 假设的“覆盖优先”机制不成立。

### 对应论文哪一节

- 方法章的候选生成与来源完备性检查小节。
- 实验章的 F2/F4 分项、`provenance-incomplete` 分层分析和 MemCut residual 辅证小节。

## 1.4 贡献三（B）：F4 源—结论双语义删除

### 主张一句话

F4 遗忘请求必须先被解析为 `conclusion-only` 或 `source`：前者只删除派生结论并登记防再推导墓碑、保留源事实；后者删除源事实，并强制其全部派生结论随闭包一同塌陷；两种语义分别用再推导探针验收，不能混成一个“结论忘了没有”的指标。

### 新在哪

- **与 BEMULA 官方口径的边界**：BEMULA 的 F4 主要检验派生结论是否被遗忘，不同时判定源事实应否保留。双语义拆分是 ProvPurge 的附加分析，必须在论文中如实标注，不能冒充 BEMULA 官方协议。
- **与一般硬删的边界**：一般硬删把“源”和“结论”都当普通记录删除，无法表达用户只要求撤回推论、但保留原始事实的请求。
- **与一般输出拦截的边界**：只在回答时屏蔽结论、但不删除结论节点，也不登记防再推导关系，无法处理换一种多跳问法重新推出结论的情况。B 同时要求结论节点删除、防再推导墓碑和读路径拦截三者配套。
- **与 PACMI 等非破坏性失效路线的边界**：失效或版本取代不等于按请求语义删除；B 讨论的是源与结论各自应保留、应删除还是应防再推导的操作语义。

### 靠什么实验证明

- F4 分项必须把 `conclusion-only` 与 `source` 分开统计，不得只报合并通过率。
- `conclusion-only` 验收包含两组探针：一组经保留源事实做一至两跳推导，检查结论是否被重新推出；另一组直接询问源事实，检查源是否仍可正常回答。
- `source` 验收同时覆盖源事实的直接问法、residual 直喂问法和由源推出结论的多跳问法，任一恢复源或结论均判失败。
- 单独报告语义识别错误、低置信未执行事件和防再推导拦截次数，避免把触发识别错误混进删除效果。

### 什么结果会证伪它

- `conclusion-only` 场景中，源事实虽被保留，但结论仍可被多跳探针稳定重新推出，说明防再推导机制无效。
- `conclusion-only` 场景中源事实问答明显受损，说明“保源”没有实现，只是变相整段删除。
- `source` 场景中源已删但派生结论仍可恢复，说明结论塌陷不成立。
- 两种语义拆分后结果与合并口径没有差异，且语义识别错误成为主要失败来源，说明双语义设计没有带来净收益。

### 对应论文哪一节

- 问题定义或方法章的触发语义小节。
- 方法章的防再推导墓碑与读路径拦截小节。
- 实验章的 F4 双语义附加分析小节。

## 1.5 贡献四（第二贡献，当前推荐版）：可证伪的删除证明

> **确认状态**：本节按当前推荐的合并方案撰写，名称为「可证伪的删除证明（falsifiable deletion evidence）」。C1、C2 或合并方案的最终取舍仍待 Joseph 确认。确认前，本节只能作为实现预留和论文预案，不得回写为规格书中已经冻结的贡献；确认后须先修订规格书第 0 节和第 12 节，再冻结实现范围。

### 主张一句话

一张删除成绩单只有同时经得起两类证伪才可信：用与门控集不同源的密封留出探针检验它有没有被“刷题”，再用完整性、完备性、新鲜性三分解和假接受攻击检验它能不能被伪造、缩范围、重放或在行未真删时蒙混过关。

### 新在哪

- **C1 留出探针泛化缺口（为纲）**：门控探针全过只证明系统通过了它反复优化的那组题。C1 另建密封留出探针集，变换族与门控集不同源；删除前先做阳性对照校准，使两组探针在未删除状态下的泄漏率大致相等，排除难度差异；删除后计算 $\Delta = \mathrm{Leak}_{holdout} - \mathrm{Leak}_{gate}$。若最终采纳 C1，$\Delta$ 应在规格修订后写入回执的扩展字段。
- **C2 回执可验证性评测（为目）**：把回执主张拆为完整性（integrity，签名与哈希能否防篡改）、完备性（completeness，回执未覆盖的残余是否存在）和新鲜性（freshness，回执是否绑定当前存储状态）。假接受攻击至少包括：挑探针签发、缩范围签发、旧回执重放、日志完整但行未真删、删后又被写回。
- **与 Athena 的边界**：Athena 证明的是第三方对训练数据的删除可被概率验证；C1/C2 检验的是自系统签发的记忆删除成绩单是否过拟合、是否可伪造，对象和威胁模型不同。
- **与 Eisenhofer et al. 的边界**：Eisenhofer 给出成员关系的密码学证明；本贡献不与其比证明强度，而是评测行为成绩单在经验层面的可证伪性。
- **与 LETHE 的边界**：LETHE 已占“签发事件回执”的位置，但其验证不查被删行缺席、不做探针复检。C2 正是把这两项变成强制验证步骤，并进一步测假接受。
- **与 Thudi et al. 的关系**：Thudi 对“证明可伪造”的批评是 C2 的问题来源。ProvPurge 的回应不是声称证明不可伪造，而是把可被攻击的面拆开、逐项实测并报告假接受率。
- **与 TSD-AUDIT 的边界**：TSD-AUDIT 证明某次检索遍历没有碰过已删集合，既不蕴含内容已不可恢复，也不等于行为探针通过。ProvPurge 的验证组合是存储缺席、输出探针和可选的遍历检查，不用其中一项冒充另一项。

### 靠什么实验证明

- C1：报告门控集与留出集在删除前的阳性对照泄漏率、删除后的 `Leak_gate`、`Leak_holdout` 与 $\Delta$，并按实例做配对置信区间。
- C2：对每一类假接受攻击报告攻击成功率、验证成本和被检出环节；给出验证预算与假接受率的关系曲线。
- 对照验证器消融：仅验签、验签 + Merkle 根、验签 + 缺席检查、验签 + 缺席检查 + 探针复检四档，证明每加一项实际堵住了哪类攻击。
- 若篇幅不足以完整展开 C2，按推荐预案处理：C1 保留为第二贡献主实验，C2 压缩为一节，保留三分解、四至五类假接受攻击和预算—假接受曲线。

### 什么结果会证伪它

- 等难校准成立后，$\Delta$ 仍系统性显著大于零，说明门控成绩单主要反映对门控探针的过拟合，不能外推为真实遗忘。
- 任一低成本假接受攻击能在完整验证流程下以不可忽略的比例通过，尤其“行未真删”或“删后写回”未被缺席检查发现，说明回执验证主张不成立。
- 留出集与门控集在删除前无法做到大致等难，导致 $\Delta$ 无法与难度差异区分，说明 C1 的证据设计无效，不能据此下结论。

### 对应论文哪一节

- 方法章末尾或独立小节「可证伪的删除证明」。
- 实验章的留出探针泛化实验、验证器消融和假接受攻击实验。
- 若只采纳 C1 或只采纳 C2，论文目录和规格书贡献清单必须同步改写，不得继续使用合并版名称。

## 1.6 定位性决策：LoRA 不进主方法，只保留条件启用的可选消融

### 主张一句话

ProvPurge 的主方法不训练底座模型、不加 LoRA；LoRA 最多作为“触发识别器提示版 vs LoRA 版”的可选消融对照臂，且只有试跑失败归因证明触发识别是首要瓶颈时才启用。

### 新在哪

本条不是创新主张，而是由文献拆解得出的定位决策：

- TOFU 的论文设定是全量微调，LoRA 只是其代码中的可选开关；Who's Harry Potter 也是全量微调。把“用了 LoRA”写成创新会犯事实和定位双重错误。
- LoKU（ICLR 2025）真正出售的是新损失与 Fisher 加权 LoRA 初始化，不是 LoRA 本身。
- MemoryBank 的 rank-16 LoRA 用于示范聊天机器人的心理领域适配，与其长期记忆机制正交；删掉该 LoRA 不改变 MemoryBank 的记忆贡献。
- 与 ProvPurge 同赛道的 BEMULA、LETHE、Wang TDSC 和 MEXTRA 均未靠训练底座成立。审稿人接受的是判定标准、事务语义和实证事实，不是参数是否被动过。

### 靠什么实验证明

本条不需要证明“LoRA 有用”，只需要用失败归因决定是否值得跑消融：

1. 先用提示版识别器完成获准的试跑。
2. 对失败请求做统一归因：触发未识别、语义误判、作用域错误、候选覆盖不足、探针后仍恢复、兄弟误伤、裁判或解析错误等。
3. 只有当触发识别错误占比最高，且明显高于“删不净”类错误时，才允许启动 LoRA 消融。
4. LoRA 消融只能使用与 BEMULA full、lite 和 quick_check 零重叠的自生成开发数据训练；报告识别 F1、F4 语义准确率、低置信未执行率和端到端 RPR/Score 的变化。

### 什么结果会证伪它

- 提示版识别并非主要错误来源，或 LoRA 版只提高分类指标却不改善端到端结果，则不启用、不写入主方法。
- LoRA 版改善来自与基线不同的额外监督信号，无法在公平口径下解释，则不得用它支撑主表结论。
- LoRA 绑定特定骨干版本后破坏同底座重跑和可复现性，则立即退回提示版。

### 对应论文哪一节

- 主方法中只用一句话说明“冻结底座、触发识别由骨干加冻结提示词完成”。
- 若满足启用条件，LoRA 对照只进消融或附录，不进入贡献清单。
- 若未满足启用条件，论文不提 LoRA 实验，只在局限性或实现说明中保留“未采用训练组件”的定位。

## 1.7 主张—证据—实验对照表

| 主张 | 核心证据 | 对应实验/报告 | 失败信号 |
|---|---|---|---|
| 删除应以探针结果为提交条件，而不是以删除动作完成为准 | 预承诺探针逐项结果、兄弟保全结果、事务状态与回执 | BEMULA 主表；去探针门消融；轮次与成本报告 | 去探针门后残余恢复不差，或过门事务仍可被独立探针恢复 |
| 溯源闭包与实体扩展负责覆盖，覆盖数 $k$ 是结果而非预设 top-$k$ | 候选池、实际删除数、逐实体覆盖数、$k$ 分布 | 去来源图、去实体扩展、池宽 20/40/60 消融 | 关闭任一通道后覆盖与效果不变 |
| 来源不完备时不得直接签 `verified` | `provenance-incomplete` 标记、触发原因、强制扩候选轮次 | F2/F4 分层；完备性规则消融；MemCut residual 辅证 | 不完备组风险不更高，或规则只增加 `failed` 不降残余 |
| F4 必须区分源与结论的删除语义 | 语义字段、防再推导墓碑、两组再推导探针 | F4 双语义附加分析 | 保源时结论可再推导，或删源时结论不塌陷 |
| 回执签的是成绩单，不是删除动作 | 逐项探针、逐层哈希、部分泄漏、判定合同版本、重建顺序 | 回执字段审计；验证器消融 | 只验签即可通过“行未真删”等假接受 |
| 成绩单不能只通过门控集自证 | 密封留出集、等难校准、$\Delta$ | C1 留出探针实验 | 等难校准后 $\Delta$ 系统性显著大于零 |
| 回执必须可被独立复核 | 缺席检查、探针复检、攻击检出环节 | C2 假接受攻击与预算曲线 | 低成本攻击能稳定绕过完整验证 |
| 冻结模型定位足够支撑主方法 | 同底座基线重跑、失败归因 | 主表与错误归因报告 | 触发识别成为首要瓶颈且系统层方案无法修复 |
| LoRA 不是默认组件 | 失败归因占比、零重叠开发集说明 | 条件启用的识别器消融 | 未达启用条件时不得运行或宣称 |

---

# 第二部分 代码应该怎么写

## 2.1 总体架构与数据流

### 2.1.1 架构原则

实现采用“领域核心 + 端口适配器”的结构：删除事务、候选生成、探针门和回执验证只依赖抽象接口，不直接依赖某个向量库、某个模型 SDK 或 BEMULA 的内部代码。这样做有三个目的：

1. 同一核心可被 BEMULA 适配器、辅助 harness 和本地单元测试共同调用。
2. vLLM、BGE-M3、向量索引和裁判模型都可替换，但替换必须进入 run manifest。
3. `query` 的只读约束可以通过只读存储句柄在结构上保证，而不是靠开发者自觉。

系统只允许两个跨路径共享状态：**来源图**与**墓碑表**。写入路径只追加，删除事务是唯一能删除来源图节点和写入墓碑的组件，读路径只读。任何模块若需要第三种共享状态，必须先修订规格书。

### 2.1.2 四条路径

```text
                         ┌──────────────────────────────┐
                         │ 配置 / 提示词 / 评分合同      │
                         │ 模型网关（LLM + Embedding）   │
                         └──────────────┬───────────────┘
                                        │
对话轮次 ──> 触发识别 ── 普通轮次 ──> 写入路径
              │                         ├─ 分句
              │                         ├─ 事实抽取
              │                         ├─ 来源登记
              │                         └─ 事实/向量/摘要三层落盘
              │
              └─ 遗忘/更新请求
                     │
                     ├─ 1. 构造探针集并哈希预承诺（删除前）
                     ├─ 2. 溯源图闭包 + 逐实体扩展生成候选
                     ├─ 3. 来源完备性检查
                     ├─ 4. 三层硬删（事务隔离区内执行）
                     ├─ 5. 按固定顺序重建摘要
                     ├─ 6. 探针门：目标探针 + 兄弟保全探针
                     │       ├─ 目标恢复 -> 扩候选 -> 回到 2/4
                     │       ├─ 兄弟误伤 -> 剔除误入节点并回滚其删除
                     │       ├─ 来源不完备 -> 强制扩候选 -> 回到 2/4
                     │       └─ 全过且完备 -> COMMIT
                     └─ 7. 签发 verified / failed 探针记分卡回执

用户提问 ──> 检索 ──> 墓碑过滤 ──> 上下文组装 ──> 骨干生成
                                      │
                                      ├─ 答案出口扫描
                                      ├─ 陈旧前提检查
                                      └─ 防再推导拦截（F4 conclusion-only）
                                             │
                                             └─> 答案（只读，不回写）
```

删除事务内部的顺序不可交换：**探针集必须先于候选生成而构造并锁定**。先看候选、再出探针，会给事后挑题留下空间，直接破坏主贡献的证据链。

### 2.1.3 与 BEMULA 的对接面

BEMULA 只通过适配器接入，核心系统不知道评测标注的存在：

```text
BEMULA runner
   │ reset()
   ├──────────────> 清空三层存储、来源图、墓碑表、防再推导墓碑和实例缓存
   │
   │ observe(turn)
   ├──────────────> 数据加载层只保留 speaker / text / timestamp
   │                -> 触发识别 -> 写入路径或删除事务
   │
   │ query(question) -> str
   └──────────────> 只读读路径 -> 答案字符串
```

适配约束：

- `reset()` 必须回到空系统，不得保留上一个实例的记录、向量、图节点、墓碑或缓存答案。
- `observe(turn)` 是唯一有写操作的入口。
- `query(question)` 绝不写回三层存储、来源图或墓碑表；实现中用只读句柄和写计数断言双重保证，评测期间写计数非零即判本实例接入违规。
- 数据加载层只向系统传 `speaker`、`text`、`timestamp`。`linked_fact_ids`、`target_fact_ids` 等标注字段在加载层丢弃，不得进入日志成为决策依据。
- BEMULA 的统一服务条件（关闭思考、输出长度预算、裁判温度等）由官方接入层处理；ProvPurge 不单独申请特殊设置。
- 一个 BEMULA 实例内部的轮次必须串行处理；并行只能发生在互相独立的实例之间，以使用多张 3090 上的多个 vLLM 副本。不得为了吞吐打乱同一实例的写入、删除和提问顺序。

## 2.2 建议仓库目录结构

```text
provpurge/
├─ README.md
│  └─ 项目定位、规格书链接、运行边界、目录说明；明确“未经批准不跑评测”
├─ pyproject.toml
│  └─ Python 包、依赖、测试、格式化与静态检查配置
├─ .env.example
│  └─ 只列环境变量名，不写任何真实密钥、签名私钥或服务令牌
├─ configs/
│  ├─ base.yaml                 # 路径、日志、状态机等基础配置
│  ├─ models.yaml               # 骨干/裁判/嵌入模型标识与服务参数
│  ├─ deletion.yaml             # 池宽、扩展跳数、轮次/成本上限占位
│  ├─ scoring_contract.yaml     # 探针答案格式、解析规则、判定合同版本
│  ├─ prompts/                  # 抽取、触发、探针、裁判、摘要等版本化提示词
│  └─ experiments/              # 主表、消融、辅助协议的实验配置模板
├─ src/provpurge/
│  ├─ domain/                   # 纯数据结构、枚举、错误类型，不依赖外部服务
│  ├─ config/                   # 配置加载、校验、配置哈希与冻结检查
│  ├─ model_gateway/            # LLM 与嵌入模型的统一调用端口
│  ├─ ingestion/                # 分句、事实抽取、来源登记、三层落盘
│  ├─ trigger/                  # 遗忘/更新识别、作用域解析、F4 双语义
│  ├─ provenance/               # 来源图读写、版本链、墓碑表
│  ├─ candidates/               # 闭包、实体扩展、组件覆盖、完备性检查
│  ├─ stores/                   # 事实层、向量层、摘要层的存储适配器
│  ├─ deletion/                 # 事务状态机、三层硬删、重建、回滚与隔离区
│  ├─ probes/                   # 探针构造、预承诺、运行、双通道判定
│  ├─ receipts/                 # 回执构建、canonical JSON、Merkle、签名与验证
│  ├─ readpath/                 # 检索、墓碑过滤、出口扫描、陈旧前提与防再推导
│  ├─ adapters/
│  │  └─ bemula/                # BEMULA MemoryAdapter 的薄适配层
│  ├─ harnesses/                # LongMemEval-S、STALE、LoCoMo、MemCut residual 辅证
│  ├─ evidence/                 # C1 留出探针与 C2 假接受评测的预留模块
│  ├─ experiments/              # 运行编排、配对统计、bootstrap、报告生成
│  └─ audit/                    # 结构化日志、run manifest、成本与错误归因
├─ tests/
│  ├─ unit/                     # 按模块拆分的单元测试
│  ├─ integration/              # 使用假模型与本地小样本的端到端测试
│  ├─ contract/                 # BEMULA 适配器契约、回执模式与配置契约测试
│  └─ fixtures/                 # 人工构造的小型对话、图、探针与回执样本
├─ devset/
│  ├─ recipes/                  # 自生成开发集的模板、种子与生成说明
│  ├─ manifests/                # 开发集版本、哈希与零重叠审计记录
│  └─ README.md                 # 开发集与 full/lite/quick_check 的隔离规则
├─ scripts/
│  ├─ check_overlap.py          # 开发集与评测集重叠审计的脚本入口（实现阶段再写）
│  ├─ verify_receipt.py         # 独立回执验证入口（实现阶段再写）
│  └─ summarize_run.py          # 运行结果汇总入口（实现阶段再写）
├─ docs/
│  ├─ architecture.md           # 本文第二部分的落地版架构说明
│  ├─ scoring-contract.md       # 判定合同的版本记录与变更说明
│  └─ runbook.md                # 试跑、正式评测与故障处理手册
├─ third_party.lock
│  └─ BEMULA 等外部仓库的固定版本、来源与校验信息，不直接改官方代码
└─ runs/                        # 运行产物目录，默认不进版本库
   └─ <run_id>/                 # manifest、日志、事务、回执、指标与报告
```

目录约束：

- `src/provpurge/domain/` 不得 import 模型 SDK、向量库 SDK 或 BEMULA 代码。
- `adapters/bemula/` 只能做协议转换，不得把评测逻辑、标注字段或基线特殊处理写进核心模块。
- `evidence/` 在 Joseph 确认第二贡献前只保留接口和数据结构预留，不默认启用 C1/C2 实验。
- `runs/` 只存运行产物，不存源代码；原始模型回复与回执必须分开存放和授权。
- BEMULA 官方仓库应作为外部固定版本依赖管理。若接入时必须修改官方文件，先记录差异并评估是否改为在本仓库适配层实现；不得静默改官方评测逻辑。

## 2.3 模块拆分

### 2.3.1 配置与冻结模块 `config`

- **职责**：加载基础、模型、删除、评分合同与实验配置；校验必填项和占位项；在运行开始时生成解析后配置与配置哈希。
- **输入**：配置文件、环境变量名、规格版本号。
- **输出**：不可变的 `ResolvedConfig`、配置哈希、未定版项清单。
- **关键约定**：含 `<TODO_PILOT>` 的关键参数在正式运行模式下必须阻断启动；单元测试模式可使用测试夹具值，但产物必须标记为 test，不得冒充评测运行。

### 2.3.2 领域模型模块 `domain`

- **职责**：定义请求、记忆记录、来源边、探针、成绩单、回执、事务状态与错误类型。
- **输入/输出**：只处理内存中的结构化对象，不读写数据库、不调用模型。
- **关键约定**：所有 ID 由上层生成并全程不变；枚举值不得用自由文本替代；任何新增边类型、回执字段或事务状态都视为规格变更。

### 2.3.3 模型网关 `model_gateway`

- **职责**：统一封装骨干 LLM、裁判 LLM 与嵌入模型调用；记录模型标识、修订版本、提示词版本、采样参数、token 用量、延迟和原始回复引用。
- **输入**：结构化消息、调用用途、采样配置、请求上下文 ID。
- **输出**：`ModelResponse` 或 `EmbeddingResponse`，包含内容、用量、完成原因和可追溯的运行 ID。
- **关键约定**：业务模块不得绕过网关直接调用 vLLM 或嵌入服务。裁判温度按冻结配置执行，主表口径为 0；所有提示词从 `configs/prompts/` 按版本加载。

### 2.3.4 写入路径 `ingestion`

- **职责**：共享同一次分句结果，完成事实抽取、来源登记与三层一致落盘。
- **输入**：含说话人、文本、时间戳的对话轮次。
- **输出**：本轮生成的 `MemoryRecord`、来源边、版本链变更和写入诊断；失败时输出结构化失败原因且三层无残留。
- **关键约定**：无来源不入库；抽取失败不得降级为整轮原文直接进向量层；任一层落盘失败必须撤销本轮其他层写入。

### 2.3.5 触发识别 `trigger`

- **职责**：判断普通轮次、遗忘请求或知识更新请求；输出 F1–F6/update 类型、作用域、置信度和 F4 语义类型。
- **输入**：当前轮次文本、有限的会话上下文、冻结提示词与阈值。
- **输出**：`DeletionRequest` 或 `UpdateRequest`；低置信时输出未执行事件，不产生删除动作。
- **关键约定**：作用域必须结构化到槽位、实体和可选时间范围；不得只输出自然语言说明。无法判定 F4 语义时不得默认按 `conclusion-only` 静默执行。

### 2.3.6 来源图与墓碑 `provenance`

- **职责**：维护句级来源图、三类边、槽位版本链、普通墓碑和防再推导墓碑。
- **输入**：写入登记、知识更新和已提交或已记录的删除事务事件。
- **输出**：图快照、邻居查询、版本链、墓碑命中结果和逐实体关系数据。
- **关键约定**：边类型只允许 `derived-from`、`summarizes`、`same-slot`；边的方向不得在不同模块中各自解释，必须由本模块统一提供“找来源”和“找派生物”两种遍历。

### 2.3.7 存储适配器 `stores`

- **职责**：分别实现事实层、向量层、摘要层的写入、检索、按 ID/哈希查存和硬删除。
- **输入**：记录、向量、查询向量、删除 ID 集和事务上下文。
- **输出**：存储结果、检索结果、删除回执、缺席检查结果。
- **关键约定**：向量层只打元数据删除标记不算硬删；删除后向量不得被索引召回。缺席检查必须同时按记录 ID 与内容哈希查询，不能只查主键。

### 2.3.8 候选生成 `candidates`

- **职责**：执行来源图闭包、逐实体扩展、组件节点覆盖、删除集选择、覆盖统计与来源完备性检查。
- **输入**：结构化请求、删除前图快照、删除前检索快照、池宽与扩展参数。
- **输出**：`CandidateBatch`，含候选节点、入选通道、选择理由、候选池大小、逐实体覆盖数和完备性标记。
- **关键约定**：实体扩展不得按相似度 top-$k$ 截断；关系链的每个中间实体必须逐个展开；闭包成员与实体扩展成员必须可独立开关，以支持消融。

### 2.3.9 删除事务 `deletion`

- **职责**：管理事务状态机、事务隔离区、三层硬删、摘要外科改写与确定性重建、选择性回滚、成本计数和终局处理。
- **输入**：请求、预承诺探针集、候选批次、存储端口和探针门结果。
- **输出**：状态转移日志、每轮删除增量、提交/失败结果和签发回执所需材料。
- **关键约定**：状态只能按 `candidate -> deleted_pending -> committed / rolled_back / failed` 转移。事务隔离区不是第四层记忆：其中对象不得被检索或读路径观察，只用于执行失败时恢复和兄弟误伤时选择性回滚；事务终局后必须按日志中的哈希清单安全清除隔离副本并记录清除结果。

### 2.3.10 探针工厂、运行器与判定器 `probes`

- **职责**：删除前构造并预承诺探针；按类别运行原生、residual 直喂、推导多跳、陈旧前提/攻击型、兄弟保全和 F4 再推导探针；执行双通道判定与部分泄漏记账。
- **输入**：删除前可见的目标槽位、实体、旧值引用、无关记忆抽样、冻结的生成与判定合同。
- **输出**：`CommittedProbeSet` 与逐轮 `ProbeScorecard`。
- **关键约定**：删除后不得增删或改写门控探针；确需改题必须回滚并以新事务重新预承诺。探针题目可以保存在受控的运行档案中供复检，但回执只写探针集哈希和逐项结果，不写被删旧值明文。

### 2.3.11 回执与验证 `receipts`

- **职责**：按 canonical JSON 构建载荷，计算字段哈希与 Merkle 根，Ed25519 签名；为独立验证方提供模式校验、签名校验、缺席检查和探针复检编排。
- **输入**：事务终局材料、逐轮成绩单、逐层删除哈希、覆盖统计、重建顺序记录和缺席层级声明。
- **输出**：`Receipt`、验证报告和失败分类。
- **关键约定**：同一事务不得同时存在有效 `verified` 与 `failed` 回执；回滚后重开必须用新事务 ID 签发，旧回执标记作废并保留作废记录。只验签或只重算 Merkle 根不算完成验证。

### 2.3.12 读路径防御 `readpath`

- **职责**：执行检索、墓碑过滤、上下文组装、生成、答案出口扫描、陈旧前提拒绝和防再推导拦截。
- **输入**：问题、只读存储句柄、来源图只读视图和墓碑只读视图。
- **输出**：答案与逐项防御事件计数。
- **关键约定**：读路径没有任何存储写权限。墓碑过滤必须槽位与旧值双命中；出口扫描只查与提问槽位相关的墓碑，避免全表字符串误杀；防再推导拦截不得连带拒答源事实。

### 2.3.13 BEMULA 适配器 `adapters/bemula`

- **职责**：实现 `reset()`、`observe(turn)`、`query(question) -> str`，并在加载层执行字段白名单。
- **输入**：BEMULA 轮次与问题。
- **输出**：答案字符串和接入诊断。
- **关键约定**：适配器不得读取标注字段，不得针对某个基线或实例写特殊分支，不得修改 BEMULA 官方计分逻辑。

### 2.3.14 辅助 harness `harnesses`

- **职责**：分别承载 LongMemEval-S 知识更新子集、STALE 子集、LoCoMo sanity check 与 MemCut residual 直喂辅证。
- **输入**：各自冻结的协议配置与同一核心系统接口。
- **输出**：独立结果文件与协议说明，不与 BEMULA 主表混合。
- **关键约定**：MemCut harness 必须在代码和报告中标明是“按论文协议自研复刻”，不得称为官方实现。

### 2.3.15 第二贡献评测 `evidence`（预留）

- **职责**：在 Joseph 确认后，实现 C1 密封留出探针、等难校准与 $\Delta$ 计算，以及 C2 三分解、假接受攻击与验证预算曲线。
- **输入**：门控探针运行器、回执验证器、密封留出集清单和攻击配置。
- **输出**：泛化缺口报告、假接受率报告和验证器消融结果。
- **关键约定**：留出集在删除前完成等难校准并密封；在确认与规格修订完成前，本模块不得进入主运行路径。

### 2.3.16 实验编排与审计 `experiments` / `audit`

- **职责**：按配置编排实例，记录 run manifest、模型调用、事务日志、成本、错误归因，并生成配对 bootstrap 与规定分项报告。
- **输入**：冻结配置、适配器、基线注册表和结果目录。
- **输出**：原始结果、结构化指标、置信区间、报告表和未完成项清单。
- **关键约定**：`failed` 事务必须计入分母；任何分项缺失时报告生成器应显式报错，不得静默生成“完整报告”。

## 2.4 关键数据结构

### 2.4.1 `MemoryRecord`

| 字段 | 含义 |
|---|---|
| `record_id` | 全局唯一记录 ID，终生不变 |
| `layer` | `fact` / `vector` / `summary` |
| `node_type` | `fact_item` / `vector_chunk` / `summary_sentence` |
| `slot_id` | 槽位 ID；无槽位时为空，并须有 `slot_absent_reason` |
| `entity_ids` | 关联实体 ID 列表 |
| `value_text` | 事实取值原文；非事实节点可为空 |
| `value_normalized` | 用于等价判断的规范化取值 |
| `value_components` | 组件列表，每项含组件文本、组件哈希和角色 |
| `component_group_id` | 能共同组合出一个目标的组件组 ID；仅作候选计算分组，不新增来源图边类型 |
| `content_text` | 记录正文，仅保存在系统存储和受控运行档案中 |
| `content_hash` | 正文规范化哈希 |
| `source_turn_id` | 来源对话轮次 ID |
| `source_sentence_index` | 来源句序，从 0 开始 |
| `extraction_confidence` | 抽取置信度；非抽取节点记录生成方式与置信来源 |
| `embedding_ref` | 向量索引中的引用；非向量层可为空 |
| `embedding_model` | 嵌入模型标识与版本 |
| `embedding_dim` | 向量维度 |
| `vector_hash` | 向量内容的稳定哈希，用于缺席检查 |
| `version` | 同槽位版本号 |
| `lifecycle_state` | `active` / `superseded`；硬删后记录本体不存在，由事务日志保存其 ID 与哈希 |
| `created_at`、`updated_at` | 创建与更新时间 |

### 2.4.2 `ProvenanceEdge`

| 字段 | 含义 |
|---|---|
| `edge_id` | 边 ID |
| `from_record_id` | 派生物或摘要句的记录 ID |
| `to_record_id` | 来源记录 ID |
| `edge_type` | `derived-from` / `summarizes` / `same-slot` |
| `created_by_operation` | `ingest` / `summarize` / `update` |
| `created_at` | 创建时间 |

方向必须统一：`derived-from` 与 `summarizes` 都由派生物指向来源；`same-slot` 由旧版本指向新版本。闭包遍历需要同时使用正向“找来源”和反向“找派生物”，不得误以为只沿出边走就能找到全部派生结论。

### 2.4.3 `DeletionRequest`

| 字段 | 含义 |
|---|---|
| `request_id` | 请求 ID |
| `source_turn_ref` | 触发轮次引用，不保存评测标注 |
| `request_type` | `F1`–`F6` 或 `update` |
| `semantic_type` | `conclusion-only` / `source` / `not_applicable` |
| `target_slot_ids` | 目标槽位集合 |
| `target_entity_ids` | 目标实体集合 |
| `time_range` | 可选的起止时间 |
| `confidence` | 触发识别置信度 |
| `recognizer_version` | 识别提示词与规则版本 |
| `disposition` | `execute` / `low_confidence_not_executed` / `update_conservative` |

### 2.4.4 `ProbeCase`

| 字段 | 含义 |
|---|---|
| `probe_id` | 探针 ID |
| `request_id` | 所属请求 |
| `probe_family` | `native` / `residual_direct` / `derivation` / `stale_or_attack` / `sibling` / `f4_rederive_source` / `f4_rederive_conclusion` |
| `question` | 探针问题，保存在受控运行档案中 |
| `context_mode` | `system_readpath` / `direct_feed` / `premise_embedded` |
| `context_record_ids` | residual 直喂时使用的存活记录 ID |
| `target_slot_id` | 目标槽位；兄弟探针填兄弟槽位 |
| `target_value_ref` | 目标值在受控档案中的引用；回执中不得出现明文 |
| `target_value_hash` | 目标值哈希 |
| `target_components` | 组件文本引用与哈希，用于部分泄漏记账 |
| `expected_sibling_value_ref` | 兄弟探针的期望值引用 |
| `generation_seed` | 探针构造种子 |
| `generator_version` | 探针生成提示词与规则版本 |
| `is_holdout` | 是否属于 C1 密封留出集；门控集恒为 `false` |

### 2.4.5 `ProbeOutcome` 与 `ProbeScorecard`

`ProbeOutcome` 逐题记录：

| 字段 | 含义 |
|---|---|
| `probe_id`、`round_index` | 探针与轮次 |
| `raw_response_ref`、`raw_response_hash` | 原始回复在受控档案中的引用与哈希 |
| `parsed_answer` | 按冻结解析合同得到的答案 |
| `parse_status` | `parsed` / `parse_failed` / `format_violation` |
| `parsed_channel_recovered` | 解析通道是否判为恢复 |
| `raw_exact_target_found` | 原始回复全文是否出现目标串 |
| `raw_channel_recovered` | 原始回复通道是否确认目标或等价表述已恢复 |
| `final_recovered` | 双通道合并后的最终恢复判定 |
| `channel_conflict` | 两通道是否冲突；冲突时以原始回复通道为准并标记复核 |
| `component_hit_hashes` | 命中的组件哈希列表 |
| `partial_leak` | 是否发生组件级部分泄漏 |
| `sibling_passed` | 兄弟探针是否通过；目标探针为空 |
| `judge_model`、`judge_prompt_version` | 裁判标识与提示词版本 |
| `scoring_contract_version` | 判定合同版本 |
| `latency_ms`、`prompt_tokens`、`completion_tokens` | 成本记录 |

`ProbeScorecard` 汇总一轮结果，至少包含：`transaction_id`、`round_index`、全部 `ProbeOutcome`、目标探针总数与恢复数、兄弟探针总数与通过数、`partial_leak_count`、目标恢复的探针 ID 列表、兄弟失败的探针 ID 列表、总调用次数与 token 用量。

### 2.4.6 `Receipt`

| 字段 | 含义 |
|---|---|
| `receipt_version` | 回执模式版本 |
| `receipt_id`、`request_id`、`transaction_id` | 回执与事务标识 |
| `transaction_status` | `committed` / `failed` / `rolled_back` |
| `conclusion` | `verified` / `failed`；回滚事务不得签 `verified` |
| `closure_hash` | 删除集节点 ID 清单哈希 |
| `probe_precommit_hash` | 删除前锁定的探针集哈希 |
| `round_scorecards` | 每轮逐项探针与兄弟保全结果 |
| `layer_delete_hashes` | 事实、向量、摘要三层各自的被删对象清单哈希 |
| `candidate_pool_size` | 候选池大小 |
| `deleted_count` | 实际删除数 |
| `coverage_k` | 覆盖数 $k$ |
| `per_entity_hop_coverage` | 逐实体 1-hop 邻居总数与已入池数 |
| `partial_leak_count` | 组件级部分泄漏总数，与全串恢复分开 |
| `scoring_contract_version` | 双通道判定合同版本 |
| `channel_review_flags` | 双通道冲突与复核标记 |
| `summary_rebuild_order` | 删除后摘要重建/合并的确定性顺序记录或其哈希 |
| `absence_check_layers` | 实际覆盖的 `storage` / `output` / `traversal` 层级；未覆盖必须显式缺省 |
| `issued_at` | 签发时间戳 |
| `payload_hash` | canonical JSON 载荷哈希 |
| `merkle_root` | 载荷字段的 Merkle 根 |
| `signature` | Ed25519 签名 |
| `signing_key_id` | 签名密钥标识，不含私钥 |
| `voided_by_receipt_id` | 作废关系；无作废时为空 |

若 Joseph 最终确认 C1 或合并方案，须先修订回执模式，再在新版本中增加 `holdout_leak`、`gate_leak` 与 `generalization_gap_delta` 等字段。不得在当前 `receipt_version` 中静默塞入未定版字段。

## 2.5 模块间接口约定

1. **只用结构化对象通信**：模块间不得传自由文本作用域、自由文本删除清单或未标版本的判定结果。
2. **快照一致性**：同一事务的探针构造、候选生成、完备性检查和首轮删除必须引用同一 `memory_snapshot_id`；每轮扩候选后生成新的快照 ID 并记录父快照。
3. **幂等键**：写入、删除和回执签发都以 `request_id + transaction_id + round_index` 形成幂等键，重复调用不得重复删除或重复签发。
4. **错误分类统一**：至少区分 `MODEL_CALL_FAILED`、`PARSE_FAILED`、`SOURCE_MISSING`、`STORE_WRITE_FAILED`、`INDEX_DELETE_FAILED`、`SUMMARY_REWRITE_MISMATCH`、`PROBE_RECOVERED`、`SIBLING_DAMAGED`、`PROVENANCE_INCOMPLETE`、`COST_CAP_EXCEEDED`。不得把所有失败压成一个布尔值。
5. **日志先于结论**：任何状态转移、候选增量、删除增量和探针结果必须先写结构化日志，再更新事务结论，避免进程中断后只剩结论没有证据。
6. **只读句柄**：读路径只能拿到 `ReadOnlyMemoryView`。该接口不暴露写入、删除和墓碑追加方法；契约测试要断言 `query` 期间写计数为零。
7. **模型调用可追溯**：每次调用必须带 `run_id`、`transaction_id`、用途、提示词版本和采样配置 ID。禁止业务模块散落硬编码提示词。
8. **时间与随机性**：业务代码不得直接调用系统随机数或未记录的当前时间生成关键结果；随机源由运行上下文注入并记录种子，时间由时钟接口注入以便测试。
9. **基线公平性**：基线适配器与 ProvPurge 必须通过同一模型网关、同一嵌入配置和同一服务条件运行。ProvPurge 专属的提示词或额外调用必须单独计量，不得藏进公共层。
10. **第二贡献开关**：`evidence` 功能由独立配置开关控制，默认关闭。只有 Joseph 确认具体方案且规格修订完成后，才能在实验配置中启用。

---

## 2.6 核心算法伪代码

### 2.6.1 闭包扩展、组件覆盖与来源完备性检查

```text
FUNCTION generate_candidates(request, graph_snapshot, retrieval_snapshot, config):
    seeds := records_for_slots(request.target_slot_ids)
             UNION records_for_entities(request.target_entity_ids)
             UNION all_versions_of_target_slots(request.target_slot_ids)

    closure := EMPTY_SET
    queue := seeds

    WHILE queue not empty:
        node := queue.pop()
        IF node in closure: CONTINUE
        closure.add(node)

        # 找来源：派生物 -> 来源
        FOR edge IN graph.outgoing(node, types={derived-from, summarizes}):
            queue.push(edge.to_record)

        # 找派生物：来源 -> 派生物/摘要句/结论
        FOR edge IN graph.incoming(node, types={derived-from, summarizes}):
            queue.push(edge.from_record)

        # 同槽位版本链必须整链处理，避免只删 active 版本
        FOR edge IN graph.same_slot_chain(node):
            queue.push(edge.other_record)

    # 组件覆盖：单个节点不含目标值，但多个节点可组合出目标时，整组进入候选
    REPEAT until fixpoint:
        FOR group IN component_groups_related_to(request, closure):
            IF union_of_components(group) can compose or imply target:
                closure.add_all(group.member_records)

    candidate_pool := copy(closure)

    # 逐实体扩展：每个中间实体单独展开，不做相似度 top-k 截断
    entity_frontier := request.target_entity_ids
    FOR depth IN 1 .. config.entity_expansion_depth:   # 默认从 1 跳开始，扩候选时可到 2 跳
        next_frontier := EMPTY_SET
        FOR entity IN entity_frontier:
            neighbors := graph.entity_neighbors(entity)       # 全部一跳邻居，不按相似度截断
            FOR neighbor_entity IN neighbors:
                records := records_for_entity(neighbor_entity)
                candidate_pool.add(records, channel="entity_expansion", via_entity=entity)
                next_frontier.add(neighbor_entity)
            coverage[entity] := {
                one_hop_neighbor_total: count(neighbors),
                one_hop_neighbor_in_pool: count(neighbors whose records entered candidate_pool)
            }
        entity_frontier := next_frontier

    IF request.semantic_type == "source":
        candidate_pool.add_all(all_derived_conclusions_of(request.target_slot_ids))
    IF request.semantic_type == "conclusion-only":
        candidate_pool := filter_to_conclusion_records(candidate_pool, request)
        register_anti_rederivation_intent(request)

    selected := select_deletion_set(
        candidate_pool,
        priority = [closure_first, marginal_evidence_coverage, entity_channel_last]
    )

    completeness := check_provenance_completeness(
        request, candidate_pool, retrieval_snapshot
    )
    RETURN CandidateBatch(selected, candidate_pool, coverage, completeness)


FUNCTION check_provenance_completeness(request, candidate_pool, retrieval_snapshot):
    reasons := []

    IF any candidate or related source record lacks source anchor:
        reasons.add("SOURCE_ANCHOR_MISSING")

    IF any target-related extraction confidence < config.frozen_extraction_threshold:
        reasons.add("EXTRACTION_CONFIDENCE_LOW")

    retrieved_relevant := retrieval_snapshot.live_records_related_to(request)
    IF exists record IN retrieved_relevant such that record NOT IN candidate_pool
       OR exists candidate_pool record with no retrieval counterpart where one is expected:
        reasons.add("CLOSURE_RETRIEVAL_MISMATCH")

    IF any source sentence mentions target entity/slot
       but has no registered record in write_diagnostics:
        reasons.add("SOURCE_SENTENCE_UNREGISTERED")

    RETURN {
        is_incomplete: reasons not empty,
        reasons: reasons,
        discrepancy_record_ids: IDs used only as expansion evidence, not as a third candidate channel
    }
```

实现注意：

- 检索不一致的记录不能被偷偷变成“第三候选通道”。它先触发 `provenance-incomplete`，再作为下一轮沿来源图和实体图扩候选的证据；若图和实体关系始终够不到该记录，事务应扩轮失败，而不是绕过规格直接删一个无来源关系的记录。
- 池宽是候选装载与成本参数，不是覆盖目标。候选选择可以按边际证据覆盖分批进入删除集，但不得按与目标的相似度排名丢弃实体扩展命中项。池宽消融必须记录未入选候选及其通道。
- `coverage_k` 是最终删除集实际命中的可推导证据数量，必须在删除集确定后计算并写入事务记录与回执。

### 2.6.2 探针门控提交循环

```text
FUNCTION execute_deletion_transaction(request, config):
    snapshot := memory.current_snapshot()
    probe_set := probe_factory.build_from_predelete_information(request, snapshot)
    precommit_hash := canonical_hash(probe_set)
    journal.write_probe_precommit(request.id, precommit_hash)

    transaction := journal.create(request, state="candidate")
    expansion_depth := 1
    extra_candidate_seeds := EMPTY_SET

    FOR round IN 1 .. config.MAX_ROUNDS:        # <TODO_PILOT> 定版后冻结
        batch := candidates.generate(
            request, snapshot,
            expansion_depth=expansion_depth,
            extra_seeds=extra_candidate_seeds
        )
        validate_cost_before_round(batch, config) else FAIL("COST_CAP_EXCEEDED")

        store_txn := stores.begin_transaction(snapshot)
        quarantine_manifest := deletion.move_candidates_to_quarantine(store_txn, batch.selected)
        deletion.hard_delete_fact_vector_summary(store_txn, batch.selected)
        deletion.rebuild_summaries_deterministically(store_txn, batch.affected_summaries)
        IF transaction.state == "candidate":
            transaction.transition("deleted_pending")
        journal.record_round_pending(transaction, round)

        scorecard := probes.run_all(
            probe_set.gate_probes,
            read_view = store_txn.pending_read_view,
            scoring_contract = config.scoring_contract
        )
        journal.append_round(transaction, batch, scorecard)

        completeness := candidates.recheck_completeness(request, store_txn, batch)

        IF scorecard.all_targets_not_recovered
           AND scorecard.all_siblings_passed
           AND NOT completeness.is_incomplete:
            stores.commit(store_txn)
            deletion.secure_purge_quarantine(quarantine_manifest)
            transaction.transition("committed")
            receipt := receipts.issue_verified(transaction, journal)
            RETURN committed(receipt)

        IF scorecard.has_sibling_failures:
            offending := deletion.find_off_target_records(batch.selected, scorecard)
            deletion.restore_from_quarantine(store_txn, offending)
            batch.remove(offending)
            # 误伤修复后必须重跑全部探针；下一轮从已修复的删除集继续
            extra_candidate_seeds := EMPTY_SET
            CONTINUE

        IF execution_error_occurred:
            stores.rollback(store_txn)
            deletion.restore_all_from_quarantine(quarantine_manifest)
            transaction.transition("rolled_back")
            RETURN rolled_back(transaction)

        # 目标仍可恢复，或来源完备性仍触发：强制扩候选
        expansion_depth := min(2, expansion_depth + 1)
        extra_candidate_seeds := seeds_from_failed_probes(scorecard)
                                 UNION seeds_from_discrepancies(completeness)
                                 UNION next_unselected_coverage_tranche(batch)
        snapshot := store_txn.pending_snapshot_for_next_round

    # 达到轮次或成本上限：不得提交、不得签 verified、必须计入分母
    transaction.transition("failed")
    deletion.finalize_failed_without_verified_commit(transaction)
    receipt := receipts.issue_failed(transaction, journal)
    RETURN failed(receipt)
```

实现注意：

- 上述伪代码中的隔离区只服务于单个事务的回滚与误伤修复，不是可检索的记忆层。事务进入终局后，隔离副本必须清除并留下只含 ID 与哈希的清除记录。
- 兄弟保全失败时不得继续扩大候选。先找误入删除集的无关节点、恢复其删除，再重跑全部探针。
- `provenance-incomplete` 即使在探针表面全过时也必须进入扩候选分支。只有完备性复检不再触发，才允许提交。
- 轮次、调用次数、token 和时延都必须有上限。达到任一上限时走 `failed`，不得为了“再试一次”临时改配置。
- F4 的 `conclusion-only` 分支还必须在提交前确认：结论再推导探针不过、源事实探针仍通过、防再推导墓碑已登记。`source` 分支必须确认源与结论均不可恢复。

### 2.6.3 双通道探针评分

```text
FUNCTION score_probe(probe, response, scoring_contract):
    raw := response.raw_text

    # 通道一：按冻结合同解析
    parsed := parse_answer(raw, scoring_contract.answer_format)
    parsed_recovered := FALSE
    IF parse succeeded:
        parsed_recovered := judge_equivalent(parsed.answer, probe.target) == RECOVERED

    # 通道二：原始回复优先
    exact_found := literal_contains(raw, probe.target_value)
    raw_judgement := judge_on_raw_response(raw, probe)
    raw_recovered := exact_found OR raw_judgement == RECOVERED

    conflict := parsed_recovered != raw_recovered
    final_recovered := raw_recovered
    # 解析通道只能提供结构化答案与审计信息，不能覆盖原始回复中的实际出现；
    # 若解析判恢复而原始通道不确认，标冲突并进入复核，不直接判系统已删净。

    component_hits := []
    FOR component IN probe.target_components:
        IF literal_contains(raw, component.text)
           OR judge_component_present(raw, component) == PRESENT:
            component_hits.add(component.hash)

    partial_leak := component_hits not empty AND NOT final_recovered

    IF probe is sibling:
        sibling_passed := parsed.answer matches expected
                          AND raw_judgement confirms expected
                          AND NOT conflict
        RETURN outcome(sibling_passed, conflict, component_hits)

    RETURN outcome(
        final_recovered=final_recovered,
        parsed_channel=parsed_recovered,
        raw_channel=raw_recovered,
        exact_target_found=exact_found,
        conflict=conflict,
        component_hits=component_hits,
        partial_leak=partial_leak
    )
```

判定规则：

- 裁判提示词、答案格式、解析器和等价表述规则共同组成判定合同，必须有版本号；合同变更视为方法变更。
- 裁判温度按冻结配置执行。边界情形宁可判为恢复成功，即对删除方从严。
- 全串恢复与组件级部分泄漏分开计数。完整目标没出现不等于没有泄漏。
- 原始回复必须保存受控引用与哈希。只保存解析后答案会重犯 AfterDeleteBench 揭示的解析合同风险。

### 2.6.4 回执构建与验证

```text
FUNCTION issue_receipt(transaction, journal, conclusion):
    payload := {
        receipt_version,
        receipt_id, request_id, transaction_id,
        transaction_status, conclusion,
        closure_hash: hash(sorted deleted record IDs),
        probe_precommit_hash,
        round_scorecards: journal.all_round_scorecards,
        layer_delete_hashes: {
            fact: hash(sorted fact IDs),
            vector: hash(sorted vector IDs),
            summary: hash(sorted summary IDs)
        },
        candidate_pool_size, deleted_count, coverage_k,
        per_entity_hop_coverage,
        partial_leak_count,
        scoring_contract_version,
        channel_review_flags,
        summary_rebuild_order,
        absence_check_layers,
        issued_at
    }

    canonical_bytes := canonical_json(payload)  # 键排序、紧凑分隔符、稳定编码
    payload_hash := hash(canonical_bytes)
    merkle_root := merkle_root_over_ordered_field_hashes(payload)
    signature := ed25519_sign(signing_key, payload_hash || merkle_root)
    RETURN payload + {payload_hash, merkle_root, signature, signing_key_id}


FUNCTION verify_receipt(receipt, current_store, sealed_probe_archive, config):
    # 1. 结构与防篡改
    REQUIRE receipt matches schema and receipt_version is supported
    REQUIRE canonical_json(receipt.payload) hashes to receipt.payload_hash
    REQUIRE rebuilt merkle_root == receipt.merkle_root
    REQUIRE ed25519_verify(receipt.signature, receipt.payload_hash, receipt.merkle_root)
    REQUIRE no other active verified receipt exists for same transaction

    # 2. 被删行缺席检查（强制）
    FOR layer IN {fact, vector, summary}:
        FOR deleted_id, deleted_hash IN receipt.layer_delete_manifest_refs:
            REQUIRE current_store[layer].find_by_id(deleted_id) is absent
            REQUIRE current_store[layer].find_by_hash(deleted_hash) is absent
            REQUIRE vector_index cannot retrieve deleted vector, if layer == vector

    # 3. 遍历层声明核对（按回执声明，不许含糊）
    IF "traversal" IN receipt.absence_check_layers:
        scored_ids := instrumented_retriever.collect_scored_vector_ids(verification_contexts)
        REQUIRE intersection(scored_ids, deleted_vector_ids) is empty
    ELSE:
        verification_report.mark("traversal_not_covered")

    # 4. 探针复检（强制）
    probe_set := sealed_probe_archive.load_by_hash(receipt.probe_precommit_hash)
    REQUIRE canonical_hash(probe_set) == receipt.probe_precommit_hash
    recheck := probes.run_all(
        probe_set,
        read_view = current_store.rebuilt_current_view,
        scoring_contract_version = receipt.scoring_contract_version
    )
    REQUIRE recheck.all_targets_not_recovered
    REQUIRE recheck.all_siblings_passed

    RETURN verification_passed(
        integrity = TRUE,
        storage_absence = TRUE,
        output_absence = recheck result,
        traversal = covered_or_explicitly_not_covered
    )
```

验证失败必须分类为：签名/哈希不一致、被删行仍存在、向量仍可检索、探针集哈希不一致、复检恢复目标、兄弟保全失败、回执已作废、遍历声明不实。不得只返回一个无原因的 `false`。

### 2.6.5 读时墓碑过滤与出口防御

```text
FUNCTION answer_question(question, read_only_view):
    write_count_before := read_only_view.write_counter

    premise := detect_stale_premise(question, read_only_view.tombstones,
                                    read_only_view.active_slots)
    retrieved := read_only_view.retrieve(question)

    filtered := []
    FOR item IN retrieved:
        IF tombstone exists with slot == item.slot AND old_value == item.value:
            defense_events.tombstone_filter += 1
            CONTINUE
        filtered.add(item)

    context := assemble_context(filtered)
    draft := model_gateway.generate(question, context)

    # 防再推导：只拦截被删结论，不连带拒答源事实
    IF anti_rederivation_tombstone matches (draft.source_records, draft.conclusion_type):
        defense_events.anti_rederivation += 1
        draft := regenerate_with_exclusion(question, context,
                                           excluded_conclusion = matched_conclusion)
        IF draft still derives matched_conclusion:
            draft := answer_without_conclusion(matched_conclusion)

    related_tombstones := tombstones_for_slots(question.slots)
    IF draft contains any old_value IN related_tombstones:
        defense_events.output_scan += 1
        draft := regenerate_with_exclusion(question, context,
                                           excluded_values = related_tombstones)
        IF draft still contains any related old_value:
            draft := refusal_without_old_value()

    IF premise.is_stale:
        draft := prepend_correction(draft,
            "该前提中的信息已删除或已更新，以下按当前版本回答。")
        defense_events.stale_premise += 1

    REQUIRE read_only_view.write_counter == write_count_before
    RETURN draft, defense_events
```

误杀控制：墓碑过滤要求槽位和旧值双命中；当前 active 值即使与墓碑同槽位也不得过滤。出口扫描只查与当前问题槽位相关的墓碑，不做全墓碑表无差别匹配。陈旧前提与当前版本一致时不触发拒绝话术；用户只是问旧值是否存在时，按“已删除或已更新”如实回答。

## 2.7 配置与可复现规范

### 2.7.1 配置组织

配置按“基础配置 + 模型配置 + 方法配置 + 评分合同 + 实验覆盖”分层，后层只能覆盖前层中明确允许覆盖的字段。每次运行保存解析后的完整配置，不只保存命令行参数。

必须占位、待试跑定版的参数：

| 参数 | 配置名 | 当前状态 | 定版方法 |
|---|---|---|---|
| 探针门最大轮次 | `deletion.max_rounds` | `<TODO_PILOT>` | 获准试跑后看单请求轮次分布与边际收益，冻结后写入规格书 |
| 单请求探针调用上限 | `deletion.max_probe_calls_per_request` | `<TODO_PILOT>` | 按调用分布和成本上限共同定版 |
| 单请求 token 上限 | `deletion.max_tokens_per_request` | `<TODO_PILOT>` | 记录输入/输出 token 分布后冻结 |
| 单请求时延上限 | `deletion.max_wall_time_seconds_per_request` | `<TODO_PILOT>` | 与轮次上限一起定版，避免只限轮次不限慢调用 |
| 骨干具体版本 | `models.backbone.exact_revision` | `<TODO_PILOT>` | Qwen3-32B 级候选中定版并记录量化或跨卡部署方式 |
| 裁判归属 | `models.judge.provider` | `<TODO_DECISION>` | 测得裁判调用总量后由 Joseph 决定本地或外部 API |
| MemCut 辅证子集规模 | `harnesses.memcut.subset_size` | `<TODO_PILOT>` | 子集构造方式与规模在试跑阶段冻结 |

已经定版、不得在实现中重新打开的参数包括：候选池默认宽度的口径、实体扩展不做相似度截断、摘要重建顺序必须确定性化并记录、判定必须双通道、回执序列化为 canonical JSON、验证必须含缺席检查与探针复检。

### 2.7.2 Seed 规范

- 设一个运行级主种子 `run.seed`，再按固定规则派生实例种子、探针构造种子、兄弟抽样种子、开发集生成种子和 bootstrap 种子。
- 派生规则与种子值写入 run manifest；不得在代码中散落 `random` 调用。
- 兄弟保全探针的抽样种子必须逐请求记录，保证同一事务可重放。
- bootstrap 固定 10,000 次，聚类单位为实例；种子写入实验配置。
- 模型服务即使温度为 0，也要记录服务版本与采样参数，不把“温度为 0”误写成“结果天然完全确定”。

### 2.7.3 Run manifest 必备字段

每次运行目录必须有 `manifest.json`，至少包含：

- `run_id`、开始/结束时间、运行类型（unit-test / contract-test / dev / pilot / formal；未获准时不得出现 pilot/formal 产物冒充正式结果）；
- 代码版本：git commit、是否存在未提交改动、规格书版本与文件哈希、本方案版本；
- 配置：解析后配置的哈希、实验配置名、所有 `<TODO_*>` 是否已定版；
- 骨干模型：名称、准确 revision、量化方式、vLLM 版本、部署拓扑、温度与其他采样参数、思考开关与输出预算；
- 嵌入模型：名称、revision、维度、部署方式；
- 裁判模型：名称或 API 标识、提示词版本、温度、判定合同版本；
- 探针：生成提示词版本、每请求的预承诺哈希索引、兄弟抽样种子、留出集状态（未启用/已确认启用）；
- 数据：基准名称与版本、官方仓库固定版本、开发集版本与零重叠审计哈希；不得把标注字段写进 manifest 作为系统输入；
- 硬件与环境：GPU 型号与数量、驱动、CUDA、Python 与依赖锁文件哈希、vLLM 副本数；
- 成本：总模型调用次数、输入/输出 token、探针轮次汇总、失败与回滚计数；
- 随机性：主种子与各类派生种子规则；
- 产物索引：事务日志、回执、原始回复档案、指标和报告的相对路径及哈希。

### 2.7.4 日志与结果目录

```text
runs/<run_id>/
├─ manifest.json
├─ config.resolved.yaml
├─ environment.txt
├─ logs/
│  ├─ system.jsonl
│  ├─ model_calls.jsonl
│  ├─ transactions.jsonl
│  └─ errors.jsonl
├─ transactions/<request_id>/
│  ├─ candidates.json
│  ├─ completeness.json
│  ├─ rounds.json
│  └─ scorecards.json
├─ receipts/
│  ├─ verified/
│  ├─ failed/
│  └─ voided/
├─ raw_responses/          # 受控保存，不进公开仓库
├─ metrics/
│  ├─ main.json
│  ├─ breakdowns.json
│  ├─ ablations.json
│  └─ cost.json
└─ reports/
   ├─ summary.md
   └─ error_attribution.md
```

隐私与安全要求：回执不得含被删旧值明文；签名私钥不得进入仓库、配置、日志或 run manifest。原始回复可能包含目标值，只能存放在受控目录，并在公开代码包中默认排除。评测标注字段不得进入系统日志作为决策依据。

### 2.7.5 硬件与公平性口径

- 计算资源固定按 5 张 RTX 3090（每张 24 GB）规划，不训练底座模型。
- vLLM 提供 OpenAI 兼容端点；骨干可按试跑定版选择量化单卡或原精度跨卡。优先多副本并行处理独立实例，而不是把全部卡绑成一个副本。
- 嵌入使用本地 BGE-M3 级模型，不调用外部嵌入 API。
- 所有进入主表的基线必须在同一骨干、同一嵌入、同一服务条件下重跑；外部论文数字只可用于相关工作讨论。
- 基线若因官方适配器结构而有上限，例如删除范围受检索批次限制，必须原样保留官方行为并在报告中说明，不得为基线私下增强后再与 ProvPurge 比较。

## 2.8 测试策略

### 2.8.1 单元测试清单

**写入与来源图**

- 同一轮多次分句得到同一句序编号；抽取引用不存在的句序时整轮拒绝落盘。
- 任一节点缺来源锚时三层均无残留。
- `derived-from`、`summarizes` 方向正确；同一事实可同时追溯来源并找到全部派生物。
- 同槽位版本链 active 版本唯一，旧版本为 `superseded`。

**触发识别**

- 任意输入都输出结构化类型、作用域、置信度；F4 请求必须输出语义类型或进入低置信路径。
- 低置信输入不产生删除，且生成可检索的未执行事件。
- `conclusion-only` 不把源事实放入删除集；`source` 把派生结论放入闭包候选。

**候选生成**

- 图中存在环时闭包遍历终止且结果无重复。
- 关闭实体扩展后，候选严格等于来源图闭包。
- 实体扩展不按相似度排名截断；关系链的每个中间实体都被单独展开。
- 组件节点单看不含目标、组合后可推出目标时，整组进入候选。
- 三类完备性触发条件分别能产生 `provenance-incomplete` 与正确原因码。
- 每个请求的候选池、删除数、覆盖数 $k$ 与逐实体覆盖数都被记录。

**删除事务**

- 任一层注入删除失败时，事务回滚且三层与事务前一致。
- 向量删除后按 ID、哈希和相似检索三种方式都找不到该向量。
- 摘要只改目标句时通过；非目标句变化时判失败并回滚。
- 相同保留集、删除集与重建顺序产生相同重建结果；重建顺序被写入回执材料。
- 状态机拒绝跳过 `deleted_pending` 直接进入 `committed`。

**探针与判定**

- 删除执行后尝试修改门控探针集被拒绝，或触发回滚与新事务重承诺。
- 四类目标探针与兄弟探针均被构造且计数分开；F4 两组再推导探针均执行。
- 解析通道判未恢复、原始回复含目标时，最终判恢复并标记冲突。
- 完整目标未出现但组件出现时，只增加 `partial_leak_count`，不误记为全串恢复。
- 兄弟探针失败时不扩候选，先走误伤修复分支。
- `provenance-incomplete` 未解除时，即使探针全过也不得提交。

**回执与验证**

- canonical JSON 对同一载荷在键顺序扰动后产生相同字节与哈希。
- 篡改任一成绩单字段后，哈希、Merkle 根或签名校验失败。
- 回执全文不含被删旧值明文。
- “签名正确但被删行仍在库中”的样本必须验证失败。
- “日志完整但删后写回”的样本必须被缺席检查或新鲜性检查发现。
- 回执未声明 traversal 时，验证报告显式写“不覆盖遍历层”，不得默认通过。

**读路径**

- 槽位与旧值双命中时过滤；同槽位当前 active 值不被过滤。
- 生成答案含相关旧值时触发一次重生成；第二次仍含旧值时返回不含旧值的回答并记录事件。
- 陈旧前提与当前值一致时不触发；不一致时先纠正前提。
- `conclusion-only` 的源事实问题可正常回答，但重新推出被删结论的答案被拦截。

**BEMULA 适配器**

- 数据加载层输出字段集合严格等于白名单。
- `reset()` 后三层、图、墓碑和缓存全空。
- `query()` 期间所有写计数为零。
- `observe()` 是唯一写入口；同一实例的轮次顺序不被并行打乱。

### 2.8.2 集成测试

使用假 LLM、假嵌入器和人工小样本完成以下端到端链路，不连接真实评测：

1. 普通对话写入后可问答，来源锚可追溯。
2. 知识更新后旧值进入墓碑与版本链，探针确认旧值不可恢复、新值可答。
3. 已知“删不干净”样本触发自动扩候选，并留下多轮候选增量与成绩单。
4. 达到轮次上限的样本最终为 `failed`，签 `failed` 回执且计入分母。
5. F4 两种语义分别完成“保源删结论”和“删源塌结论”。
6. 摘要改写误伤、向量删除失败、事实层删除失败三类故障分别回滚。
7. 回执签发后由独立验证入口完成验签、缺席检查和探针复检。
8. C1/C2 未确认时，默认运行路径不加载留出集、不执行假接受攻击；确认后才按开关启用。

### 2.8.3 开发集生成原则与红线

以下三条是已核实的坑，写成开发红线：

1. **开发集必须自生成且与 BEMULA full 零重叠**。重叠检查至少覆盖实例、对话文本、实体、槽位值、请求文本和近重复改写；审计结果与哈希写入 `devset/manifests/`。
2. **不得在 Lite 上调参再报 full**。BEMULA lite 的实例 ID 是 full 的 100% 子集，用 Lite 调参再报告 full 构成泄漏。任何阈值、提示词、池宽或轮次参数都不得在 Lite 或 full 上调。
3. **quick_check 只能冒烟**。它只有一个 tiny 合成实例、11 个问题，与 full 零重叠；只能验证管道连通性，不能调参，也不得把结果写成评测效果。

其他原则：

- 开发集按“校准集 / 验证集”再分工。校准集用于定触发阈值、提示词和判定合同；验证集只在冻结后检查，不回头调参。
- 开发集的实体、数值、关系链和请求表达要覆盖 F1–F6、update、F4 双语义、来源缺失、抽取低置信与闭包/检索不一致等目标场景。
- 若未来启用 LoRA 消融，其训练数据只能来自自生成开发数据，且训练、校准和最终验证之间继续保持零重叠。
- 开发集生成配方、种子、版本和人工抽查记录必须可复现；不得把 BEMULA 请求原文改写后冒充独立开发集。

## 2.9 已知坑清单

| 坑 | 已核实事实 | 实现处理 |
|---|---|---|
| 解析契约可制造假阴性 | AfterDeleteBench 中两个模型官方解析为 0，但原始回复含目标分别为 77/104 与 78/104；只换解析合同，提取数可由 77/104 变为 0/104，write 效应甚至反号 | 探针判定必须双通道；原始回复优先；判定合同版本入回执；原始回复受控留档 |
| 部分泄漏会被全串指标掩盖 | AfterDeleteBench C3 中完整恢复可为 0，但组件级部分泄漏可达 104/104 | `partial_leak_count` 独立统计，不并入全串恢复，也不因全串为 0 而清零 |
| BEMULA `memoryos.py` 缺失 | 官方基线注册表登记了 memoryos，但仓库中实现文件缺失，选用必报错 | 基线运行时跳过，并在论文中如实注明缺失与跳过处理 |
| LangMem 删除口径特殊 | BEMULA 的 LangMem 适配器打开了原厂默认关闭的 `enable_deletes`，删除范围受 `query_limit`（默认 5）与当前检索批次限制 | 复现时保持官方适配器行为，不私下增强；在相关工作中说明其结构上限 |
| BEMULA Lite 是 full 子集 | Lite 实例 ID 100% 包含于 full | 禁止用 Lite 调参报 full；开发集独立自生成并做零重叠审计 |
| quick_check 不是小评测集 | 只有 1 个 tiny 合成实例、11 题，与 full 零重叠 | 只做管道冒烟，不调参、不报效果 |
| 判分与服务条件会改变结果 | BEMULA 默认 seed 42、`probe_writeback: false`、裁判温度 0.0；接入层会统一处理思考开关与输出预算 | 按官方配置接入并写入 manifest；`query` 写回用断言拦截 |
| LETHE 回执序列化文实不一致 | 其论文写 CBOR，仓库 `receipt.py` 实际签 canonical JSON | ProvPurge 按代码核查结论定版 canonical JSON，并在相关工作中如实注明出入 |
| 只验签会放过“行未真删” | LETHE `verify_receipt` 不查被删行是否仍在库中 | 我方验证强制包含被删行缺席检查与探针复检，缺一不可 |
| 摘要重建顺序会影响行为 | MemForm 的 Path 结果显示保留集不变、只改合并顺序也能改变行为 | 重建输入、合并顺序与生成参数固定，顺序写入回执；兄弟探针必须在重建后状态运行 |
| MemCut 代码未公开 | 论文称将发布，复核时仍未找到官方代码 | residual harness 按论文协议自研复刻并明示效度风险，只作辅证 |
| PolyGraph 仓库不可达 | 2026-10-30 复查仍为 404 | 只按论文内容定位，不声称核查过其代码，不据此设计接口细节 |
| 基线外部数字不可直接比 | 不同基线原论文使用不同骨干、嵌入与服务条件 | 主表基线一律同底座重跑；外部数字只进相关工作讨论 |
| 基线名称与实现可能不一致 | 注册表、论文描述和仓库文件可能彼此不一致 | 每个基线在运行前做“注册名—实现文件—实际配置”三方核对，并写入基线清单 |

## 2.10 开发里程碑 M1–M5

> **总门槛**：M1 开始前先汇报本方案并得到 Joseph 点头。每个里程碑结束都要汇报交付物、测试结果、未决项和下一阶段范围；未经确认不得自动跨阶段。M5 中任何试跑或正式评测都属于实验，必须单独获得明确点头。

### M1 仓库骨架与领域契约

- **开始前汇报节点**：汇报本文的创新点、目录结构、数据结构和 M1 范围；Joseph 点头后才可创建代码仓库与骨架。
- **交付物**：仓库目录、配置加载与哈希、领域数据结构、错误分类、日志与 run manifest 骨架、模型网关假实现、存储接口与内存版来源图。
- **验收标准**：领域模型与配置单元测试通过；任一记录可按来源锚定位；含未定版占位参数的正式运行模式会被阻断；仓库中无真实密钥和评测标注字段。
- **对应论文章节**：方法章的系统总览与数据结构小节可据此起草，但不得写结果。

### M2 写入、触发、读路径与 BEMULA 契约

- **开始前汇报节点**：汇报 M1 验收结果、触发提示词版本、低置信策略和适配器契约测试范围，确认后再进入 M2。
- **交付物**：分句与事实抽取、来源登记、三层落盘、触发识别与 F4 语义解析、版本链与墓碑、只读读路径、BEMULA 薄适配器。
- **验收标准**：字段白名单测试通过；`reset` 后状态全空；`query` 写计数为零；低置信不删除；知识更新可完成版本切换；读路径三项防御与防再推导拦截的单元/集成测试通过。
- **对应论文章节**：方法章的写入路径、触发识别与读路径防御小节。
- **禁止事项**：本阶段只用人工夹具与假模型做契约测试，不运行 BEMULA quick_check、Lite 或 full；即使 quick_check 只是冒烟，也须等 Joseph 对“管道冒烟是否启动”的明确许可。

### M3 删除事务、探针门与回执

- **开始前汇报节点**：汇报 M2 验收结果、事务终局语义、隔离区清除方式、回执字段和判定合同初稿，确认后再进入 M3。
- **交付物**：闭包与逐实体扩展、组件覆盖、完备性检查、三层硬删、确定性摘要重建、探针工厂与预承诺、双通道评分、门控循环、回执签发与独立验证、MemCut residual 协议的接口预留。
- **验收标准**：已知删不干净夹具能触发扩候选；达到上限能签 `failed` 且计入分母；`provenance-incomplete` 未解除不能签 `verified`；故障注入可回滚；篡改回执、行未真删和探针复检失败都能被验证器分类检出。
- **对应论文章节**：方法章的候选生成、删除事务、探针门与回执小节。
- **禁止事项**：不得为了让夹具通过而临时修改判定合同；合同变更必须升版本并重跑受影响测试。

### M4 基线、开发集与评测脚手架

- **开始前汇报节点**：汇报 M3 验收结果、基线清单、开发集生成配方、零重叠审计方法和第二贡献的确认状态，确认后再进入 M4。
- **交付物**：同底座基线适配与三方核对清单、自生成开发集及重叠审计、实验编排、配对 bootstrap 与分项报告生成器、辅助 harness 脚手架；若 Joseph 已确认第二贡献，再交付 `evidence` 模块的相应范围。
- **验收标准**：开发集与 full 零重叠审计可复现；Lite/full 未被用于调参；五项消融都能通过配置开关独立启停；报告生成器在缺失规定分项时会报错；所有基线使用同一模型与嵌入配置。
- **对应论文章节**：实验设置、基线、消融与统计规范小节。
- **禁止事项**：脚手架完成不等于获准运行评测。不得在本阶段自行运行 Lite/full、辅助协议或基线来“看看效果”。

### M5 试跑定版与正式实验（强门控）

- **开始前汇报节点**：先提交试跑方案，逐项列明目的、样本范围、预计模型调用量、轮次与成本上限、要定版的未决项和停止条件。**只有 Joseph 明确点头后，才可开始任何试跑。**
- **第一步：受控试跑**：在获准范围内测单请求调用分布、探针轮次、裁判总量、失败归因和骨干部署方式；据此提出最大轮次、成本上限、裁判归属、骨干版本与 MemCut 子集规模的建议值。
- **试跑后汇报节点**：向 Joseph 汇报试跑结果、失败归因、LoRA 消融是否达到写死的启用条件、第二贡献最终范围和正式实验预算。未经再次确认，不得把试跑配置直接扩成正式实验。
- **第二步：正式实验**：只有在规格书第 12 节未决项全部定版、配置冻结并再次得到 Joseph 点头后，才运行 BEMULA 主表、五项消融、辅助协议和已确认的第二贡献实验。
- **验收标准**：主表与全部分项齐全；`failed` 计入分母；配对差与置信区间按规格报告；裁判双复核和 100 题人工抽查完成；每次运行的 manifest、日志、回执与结果可相互追溯。
- **对应论文章节**：实验结果、消融、成本分析、第二贡献实验与局限性小节。

---

# 第三部分 开工前必须守住的检查表

在 Joseph 点头开工后、写第一行实现代码前，逐项确认：

- [ ] 本方案已汇报，Joseph 已明确同意进入的实现阶段和范围。
- [ ] 规格书仍是最高依据；第二贡献若已确认，先完成规格书修订再改代码范围。
- [ ] 仓库不包含真实签名私钥、服务密钥或被删内容公开样本。
- [ ] 配置中的 `<TODO_PILOT>` 与 `<TODO_DECISION>` 没有被私自填成“经验值”。
- [ ] 数据加载层白名单已写成测试，而不是口头约定。
- [ ] 开发集与 BEMULA full 的零重叠规则已写进 `devset/README.md`。
- [ ] 基线清单已逐项核对注册名、实现文件和实际配置；memoryos 的缺失处理已记录。
- [ ] 判定合同、触发提示词、探针生成提示词和裁判提示词均有版本号。
- [ ] 回执验证清单包含验签、Merkle 根、被删行缺席检查和探针复检四项。
- [ ] M5 试跑与正式实验的批准记录分别保存，不用一次点头覆盖两个阶段。

**最后一句**：ProvPurge 的代码首先要证明的不是“功能很多”，而是每一次删除都有明确的候选来源、提交条件、逐项成绩和可复核回执。任何让这条证据链变短的实现捷径，即使短期跑分更快，也视为偏离方案。

