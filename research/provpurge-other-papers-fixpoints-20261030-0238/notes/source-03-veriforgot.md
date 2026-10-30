# VeriForgot
- URL: https://www.preprints.org/manuscript/202603.2325
- 读于: 2026-10-30, index 全文 (569 行, 引言/§3 框架/§4 实验/§5 安全分析/结论/表 1–4) — index full text
- 身份: "VeriForgot: Blockchain-Attested Verifiable Machine Unlearning Using Membership Inference Oracles for GDPR Compliance", Preprints.org v1, ©2026 by the authors (MDPI Preprints)。作者名在抓取正文中未出现; 二级来源 LinkedIn 文章引为 Tulla & Chowdhury (2026), DOI 10.20944/preprints202603.2081.v1 — 与本次打开的 manuscript 编号 202603.2325 不一致, 未能消解 (index only for authorship/DOI)
- 验证对象: 参数/模型行为 — CIFAR-10 + ResNet-18 的参数化 unlearning; D_forget=500 样本 (class 0,1); 非记忆库、非 LLM (结论称 future work 才扩展到 LLM)
- 机制: MIA oracle (置信度 AUC, 阈值 τ=0.57, PASS/FAIL) + 区块链 Unlearning Certificate (cert ID, hashed subject ID, commitment hashes C_orig/C_new, oracle verdict, timestamp) + ZK (Groth16) 证明 ||θ_new−θ_orig||₂>δ 不泄露参数
- 数字: baseline MIA AUC 0.5918; GA 0.5272 / Selective Retrain 0.5604 / Strong GA 0.4669; oracle 20 例 (10 真/10 假): TPR 100%, TNR 90% (总 95.0%); 唯一 FP 为 noise scale 0.0029, AUC 0.5660 刚低于 0.57; 正文称调 τ 至 0.58 可消除该 FP 并达 100%; GA test accuracy drop 0.03%
- 重要保留: 结论 future work 明言 "implement the full Groth16 ZK circuit" — ZK 部分在本文中为协议规格而非已实现验证 (据已读文本)
