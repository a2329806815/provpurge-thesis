# MemCut — arXiv:2610.21405v1（全文已读，1521 行 HTML 全部翻完）
- URL: https://arxiv.org/html/2610.21405v1 ；abs: https://arxiv.org/abs/2610.21405v1
- 读取：2026-10-29，index 全文（browser_open）逐段读完正文+附录 A–E。状态：verified（index 全文）
- 作者：Cheng Zhang, Xuanning Cao, Zhicheng Zheng, Hui Xiong（HKUST-GZ / USTC）；2026-10-15 提交。注意：其参考文献已引 LETHE(2606.15903)。

## 关键事实
- 基准非方法：456 手工设计秘密，15 episodes，43,402 turns / 490 sessions / 1.74M tokens，均 2,893 turns/episode，≥12h/episode 人工，GPT-6 只渲染对话措辞，事实不用 LLM 发明；双标注初始一致 97.6%。
- 每事实标注：disclosure D、evidence E（均 8.1 turns，跨 8.0 sessions，与 disclosure 同 session 仅 1.8%）、minimal sufficient sets S（15.7/事实，共 7,148，每 set 2–5 turns、均 2.7、无 size-1）、distractors 26.6/事实、deletion request、四选一问题（可拒答、两种选项顺序都答对才算恢复）。
- 遗忘的形式定义：删除集 X 是 cut ⟺ D⊆X 且 X 与每个 S 相交；minimal cut 均 3.04 turns、max 5（预算 10 的一半）。
- 四种 derivation：fact assembly / relation chain / aggregation / temporal state，各 113–115 事实。
- Native recovery：经系统自身检索问四选题（同 reader 模型）；Residual recovery：绕过检索，把所有溯源到 D∪E 的存活对象直接喂同一 reader。Hidden-risk share = (Res−Nat)/Res，是 |R\N|/|R| 的下界（附录 B.1：真实值高 0–12 点）。
- 校验：组合 evidence 恢复 96–99%；最佳单 evidence turn 0%；删 D+E 后 0%；无记忆 0%。
- 删除基线（复用原生组件，非作者提出的新删除法）：S0 只删 disclosure（+closure）；S1 用删除请求查原生检索器建 20-turn 候选池 P_q，删排名 top-10；S2 同一池 20 不变，用 write model（LLM）按存储文本重排删 top-10。Reach 失败=证据在池外；Choice 失败=在池内未被选。两者均不见标注。
- 统一协议 §4.2：写入时记录每个对象的 source turns，按 turn 计数池/预算，删对象=closure（删其全部源 turn 衍生对象）；系统只需 list/search/delete/snapshot；作者称 release harness。
- 五系统：Turn RAG、Session RAG（逐字、无衍生）、MemoryBank（原文+daily summaries）、Nemori（segment 重写为 narratives）、LightMem（离线 consolidated summaries、不留原文）。四 write models：DeepSeek-V4.1-Flash、Qwen3.8-Flash、GLM-5.3-Flash、MiMo-V2.5；同一模型兼任写入/候选打分/reader；每事实用 fresh copy of store。
- 主结果：删除前 Res 94–100%（逐字库 100%）。S0 后 Nat 降 12–50 点（Turn RAG 91–95→57–60），Res 仍 85–99%。S1 后 Res 48–86%（Turn RAG 59–62%——零衍生也残留）。S2 再降 7–17 点，Res 仍 39–76%。同系统内四模型 Res(S2) 差 ≤~5 点（Session RAG ≤10）。
- Hidden-risk：Turn RAG Full 5–9% → S0 39–42% → S2 76–82%；20 配置全上升，S2 后 14/20 >50%；Session RAG S2 83–92%。
- Sweep（144 事实 EP01–05，三系统两模型）：预算固定 10、池 20→60 使 S2 Res 降 12.5–23.9 点；池固定 20、预算 10→20（删全池）Res 只再降 ≤6.5 点，全池删除后 Turn RAG/LightMem 仍 43–51%、MemoryBank 66–72%。Res≈13×(8.2−k)，R²=0.95（132 settings），系统/模型/策略合计位移 ≤4 点。
- 候选覆盖数：池20 含 evidence：Turn RAG/LightMem 4.5、MemoryBank 3.0（每事实共 8.1）；池60 → 5.8/5.9/4.5。S2 实际删中 evidence（全 456、DeepSeek+Qwen、三系统）：S1 3.0/10、S2 3.6/10 → off-target ~70%/~64%；LLM 判断只多删 0.7 evidence/事实。分系统 S2：MemoryBank 2.9、Nemori 3.4、LightMem 4.7。分类型 S2：relation chain 2.4 < assembly 3.8 / temporal 3.9 / aggregation 4.6。
- 判断收益随 reach 增长：S2−S1 在池20 为 5–15 点、池60 为 20–32 点。池60 时 S2 已交 92–93% sufficient sets（Turn RAG/LightMem）Res 仍 31–40%（partial cut 不够，要 complete cut）。
- Table 11（DeepSeek–Turn RAG，144 事实）：池20/选10：off-target 5.54、Res 52.08%；池20/选20：off-target 15.49、Res 50.69%（多删 9.95 非证据只换 −1.39 点）；池60/选10：off-target 4.50、Res 39.58%（−12.50 点且少删 1.04 非证据）。
- 类型：relation chain S2 后 Res 最高（均值 71.8%，hidden 87.1%），aggregation 最低（24.3%）；池60 Turn RAG 含 chain evidence 4.4 vs aggregation 6.9。
- 案例（App D）：关系链事实证据相似度排名 3–141，池20 只含 E1,E3（不同 link 各一，删了也不成 cut）；池30 E2 入池未被选；池50 才成 cut；E7 从未入池。每个 evidence turn 都与请求实体 1-hop 相连，作者明说 entity expansion 可在 10-turn 池内凑齐 complete cut，但「effectiveness as a deletion strategy remains untested」。
- 局限（作者自述）：受控构造（部署中 trail 频率未知）；reference baselines 只复用现有组件、结论限于该复用策略，MemCut 本身支持评测 dedicated erasure methods；只研究文本记忆，多模态/参数记忆不在范围；source-level collateral 度量不捕获 closure 删掉的无关内容与下游效用损失。
- 代码：全文（§4.2「we release its harness」、Ethics「benchmark will be released for research use」）未给任何仓库 URL；2026-10-29 两轮搜索未找到 GitHub 仓库。结论：未获取（承诺释放，未验证可打开）。
