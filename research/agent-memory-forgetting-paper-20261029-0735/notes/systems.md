# 系统核查（2026-10-29）

- Generative Agents arXiv:2304.03442: memory stream + recency/importance/relevance检索 + reflection + planning; 25 agent Smallville; 评测为可信度/消融, 无更新/删除.
- MemoryBank arXiv:2305.10250 (AAAI2024): 存储/检索 + Ebbinghaus遗忘曲线更新(强化/衰减) + 用户画像; SiliconFriend陪伴场景人工评测. 遗忘=时间衰减, 非按请求删除, 非冲突裁决.
- MemGPT/Letta arXiv:2310.08560: 虚拟上下文/OS分层, core/recall/archival, LLM自编辑函数调用; DMR等. Letta filesystem+grep 在LoCoMo 74.0% (二手, 审稿时需引Letta博客原文核对).
- Mem0 arXiv:2504.19413 (ECAI2025): 抽取→更新两阶段; 更新阶段对top-10相似记忆由LLM选 ADD/UPDATE/DELETE/NOOP; Mem0g图变体矛盾时标记失效而非硬删; LoCoMo自报66.9/68.4 J-score量级, p95延迟-91%, token-90% vs全上下文; 但全上下文在其实验中准确率更高(~73%). 分数被Zep公开质疑(65.99 vs 84 vs 修正58.44争议) → 引用必须注"厂商自测, 跨harness不可比".
- Zep/Graphiti arXiv:2501.13956: episode/semantic/community三子图; 双时态(事件时间+事务时间)四时间戳; 新边与旧边LLM判矛盾→设t_invalid失效保留历史; DMR 94.8 vs MemGPT 93.4; LongMemEval自报+18.5%/延迟-90%.
- A-MEM arXiv:2502.12110 (NeurIPS2025): Zettelkasten结构化笔记+动态链接+记忆演化(新记忆触发旧笔记属性更新); 6 backbone.
- MemoryOS arXiv:2506.06326: 短/中/长三层, Storage/Updating/Retrieval/Generation, FIFO+分段页; LoCoMo自报相对基线F1+49.11%.
- Memory-R1 arXiv:2508.19828 (ACL2026): RL训练Memory Manager学ADD/UPDATE/DELETE/NOOP + Answer Agent蒸馏; 仅152训练QA; 3B-14B; 需RL训练算力, 硕士方案应避开或仅作对照引用.
- HippoRAG/HippoRAG2: 图+Personalized PageRank多跳检索, 非对话更新专用.
