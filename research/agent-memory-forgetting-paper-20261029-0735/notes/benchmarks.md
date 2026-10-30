# 基准核查（2026-10-29）

- LoCoMo ACL2024 arXiv:2402.17753: 公开10对话, ~600轮/对话, 多会话, 1986题(单跳/多跳/时序/开放/对抗). 缺陷: Penfield审计 99/1540=6.4%答案键错误, 上限~93.6%; GPT-4o-mini judge接受62.81%故意错答; Zep博客指无知识更新题/类别5无gold/说话人归属错. 来源 https://penfieldlabs.substack.com/p/we-audited-locomo-64-of-the-answer ; https://www.getzep.com/blog/lies-damn-lies-statistics-is-mem0-really-sota-in-agent-memory/
- LongMemEval ICLR2025 arXiv:2410.10813: 500题, 五能力(抽取/多会话/时序/知识更新78题/弃答), S≈115K token, M≈1.5M/500会话, oracle版. GitHub xiaowu0162/LongMemEval. S版可塞进现代长上下文窗口→部分退化为上下文测试.
- MemoryAgentBench ICLR2026 arXiv:2507.05257: 2071题, 103K-1.44M, 四能力 AR/TTL/LRU/SF; SF=FactConsolidation(编号事实,高序号覆盖), FC-SH最好HippoRAG-v2 54%, FC-MH ≤7% (22系统). https://arxiv.org/html/2507.05257v3 ; GitHub HUST-AI-HYZ/MemoryAgentBench
- MemBench ACL2025? arXiv:2506.21605: 事实+反思记忆 × 参与+观察场景; 指标 effectiveness/efficiency/capacity; GitHub import-myself/Membench. 规模各二手源不一致(1039题 vs 53k QA), 报告中标注未核清.
- MemoryArena ICML2026 arXiv:2602.16313: 766任务, 6.9相互依赖子任务/任务, 57步; 网购/旅行规划/渐进搜索/形式推理; LoCoMo近饱和者在该处很差; 全量评测4850子任务API成本高(作者自述). https://arxiv.org/html/2602.16313v1
- BEMULA/STALE/Memora 见 collision-2026.md
- 综述: Memory for Autonomous LLM Agents arXiv:2603.07670 (write-manage-read, 三维分类, 五机制族); Memory in the Age of AI Agents arXiv:2512.13564 (Forms/Functions/Dynamics).
