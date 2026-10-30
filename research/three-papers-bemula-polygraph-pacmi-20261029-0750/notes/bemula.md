# BEMULA 全文笔记
- 来源: https://arxiv.org/html/2610.18052v1 （verified live via index fetch, 2026-10-29 07:51 UTC crawl）；abs: https://arxiv.org/abs/2610.18052v1 提交 2026-10-13
- 作者: Bofan Xie, Chaokun Wang, Guanchen Ge, Songyao Wang, Fang Niu, Hang Zhang (Tsinghua), Wei Wang (Sina Weibo)
- GitHub 实测公开: https://github.com/Nboxff/bemula-benchmark (Public, main, data/configs/scripts/results, README 含 run_eval 与三阶段生成脚本, 2026-10-29 打开确认)
- 关键: 50 instances/746 sessions/36,900 turns per version/135 requests (F1 32,F2 23,F3 26,F4 13,F5 20,F6 21)/269 target facts/1,742 Q (ret 224, eff 224, relearn 114, attack 1030=5×206, pres 150=3×50); profiles 2,993 atomic+77 derived facts; avg 84.8K tokens, max 379K; 生成模型 GPT-5.6-Luna seed42 temp0.9; 手检 10 instances/388 Q; judge human agreement 98%, kappa 0.96 (100 items)
- 指标: 有效正确性 z~ = z∧c; RA/FE/PR/RLA/AR(5攻击均值); RPR 全对才过; Score=FA×UG, FA=√(FE·AR), UG=√(PR·RLA)
- 结果: RPR ≤1.5% (LangMem/DeepSeek), 多数 0; Score max 23.1 (GPT FullCtx), 31.0 (DeepSeek LangMem), 9.5 (Qwen SimpleMem); AR ≤35.7; F6 FE≥61.9 但 AR≤30.5; F3/F5 最难; LangMem GPT PR 20.0 过删
- Backbones: GPT-5.6-Luna, DeepSeek-V4.1-Flash (兼 judge), Qwen3-32B-AWQ; embedding text-embedding-3-small; 系统: FullContext, NaiveRAG, Mem0, A-MEM, LangMem, LightMem, SimpleMem
- 局限(作者自述, §5): 合成对话、单一生成管线、仅对话记忆不含工具痕迹(cache/log/外部服务)
- Bemula-Lite: 31 short+medium, 848 Q
