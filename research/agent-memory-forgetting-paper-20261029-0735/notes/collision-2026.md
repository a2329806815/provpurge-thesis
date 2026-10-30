# 撞车风险核查（2026-10-29 打开/检索）

- BEMULA / Can Agents Forget When Asked? arXiv:2610.18052v1, 2026-10-13, 清华. https://arxiv.org/html/2610.18052v1
  50多会话对话,135遗忘请求(6类),1742题,配对control,五套探针(存储/删除/保留/再学习/抽取攻击). 6系统×3 backbone, 请求通过率 RPR ≤1.5%, 多数0. => "自建删除后追问基准"已撞车; 但"在BEMULA上真正通过的方法"仍空白.
- PACMI arXiv:2610.05732v1, 2026-10-05. https://arxiv.org/abs/2610.05732v1
  来源图+类型依赖边+四态有效性格+级联失效传播+陈旧前提检测; 自建100例/300查询/5域; 消融去掉级联 错误3→11 (p=0.0574不显著). => "墓碑+级联失效"已撞车(更新场景); 未做显式隐私删除/BEMULA/对话摘要句级清除.
- STALE arXiv:2605.06527. https://pdfs.assets.alphaxiv.org/2605.06527v1.pdf
  隐式冲突400场景/1200查询/至150K, 三探针 SR/PR/IPA, 最好55.2%; CUPMem 写时结构化状态整合+传播感知检索原型. => "隐式更新没人测/没人做写时裁决"已撞车.
- Memora/FAMA arXiv:2604.20006, ACL 2026 Findings. http://arxiv.org/abs/2604.20006
  周-月尺度, remembering/reasoning/recommending, FAMA惩罚使用失效记忆; 4 LLM×6 agent 经常复用失效记忆.
- Agentic Unlearning / SBU arXiv:2602.17692. https://arxiv.org/abs/2602.17692v1
  backflow概念; 参数+记忆双路; 记忆路=dependency closure剪孤立实体+逻辑失效共享制品; 医疗QA, 需参数更新(硕士无算力不适合照搬).
- PolyGraph / Keep Your Receipts arXiv:2610.17719. https://arxiv.org/html/2610.17719
  文档KG来源撤回清理; 直接事实删除仅使污染查询32.2%→28.0%; receipts集合运算无模型调用; 摘要receipt整表不可信(~2/3错), 逐句receipt 100%/88%忠实. => 句级来源是关键洞见, 但对象是文档KG非对话记忆.
- ChronoMem arXiv:2607.27773v2. https://arxiv.org/abs/2607.27773v2
  每次写全量快照commit+NL undo→版本解析+全局回滚, ADK集成, post-exposure协议, 约+10pp. => "带版本事实库"泛泛而谈已撞车.
- Execution-State Unlearning (Forgetting Without Restarting, AAAI 2027版式). provenance-guided selective replay, 覆盖prompt/摘要/KV cache, 删除记录leakage不变0.86-1.00, 指令遗忘抽取下Leak=1.00. => "缓存中的级联清除"在会话执行态层面已有人做; 跨会话持久记忆层仍不同.
- Deterministic freshness (2026-05, 与2606.01435同组工作): MAB FactConsolidation上 candidate+max(serial) 确定性管线 FC-SH 67.2→78.0. => "读时last-write-wins确定性规则"已撞车.
- FactConsolidation gold可达性批判: last-write规则80.25%, 67题gold需已被覆盖陈述, 多跳262K约1/3. 引用时须谨慎.
