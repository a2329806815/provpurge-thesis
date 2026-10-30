# TSD-AUDIT
- URLs: https://arxiv.org/abs/2609.19456 (摘要, index); 方法 §3 与贡献段经索引摘录 (PDF 文本镜像 http://arxiv.cnoctave.top/pdf/2609.19456, index only)
- 读于: 2026-10-30, 层级: 摘要 + 方法概览 (按任务要求)
- 身份修正: arXiv 列表标题为研讨会名 "Beyond Private Training: The New Landscape of AI Privacy" (Culatana & Li, Atlassian, submitted 16 Sep 2026, NeurIPS 2026 InfPriv workshop); 论文内标题 "Beyond Output Filtering: Externally Auditable Erasure for Inference-Time RAG Memory"
- 机制: output safety vs traversal safety; alive-before-scoring 不变量 (alive bitset 在三处、计距之前检查: 入口点取 live、邻居进 frontier 前丢 deleted、pop 时复查); 只用 live 候选修复连通性
- 证书到底证书化什么 (§3 Certificate 段): 每个 checkpoint 发出 deleted-set hash + sampled expansion traces 的 hashes + visited∩D_t verdict; 独立 verifier 重算 hashes 并断言 no-intersection。即证书化 "某次查询的遍历/计分集合与删除快照中的删除集不相交" 这一过程性质, 按 per-query scored-trace certificate Against deletion snapshot 核验
- 实证: Faiss IndexHNSWFlat 原生过滤距离计算数不变; 70% 删除率下 trace-faithful replay 在 100/100 审计查询中检出 deleted-vector scoring; 外部效度: replay 与原生引擎 top-10 agreement 1.000, 距离计数 0.98× faiss ndis; soft-delete 证书全败, traversal-safe 与 clean-rebuild 通过 (0 hash/count mismatches); region-targeted 删除下 Recall@10 比原生过滤高 4.3–42.2 个百分点 (删除比例 0.5–0.9)
- 自我定位: 作者明言 contribution is "audit-not-algorithm", enforcement 是已知的 predicate-aware ANN label-filtered traversal; at-rest 可恢复性是 Ghost Vectors 的题目, 不在证书范围内
