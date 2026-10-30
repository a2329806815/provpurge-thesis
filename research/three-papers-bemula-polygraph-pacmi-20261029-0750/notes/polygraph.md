# PolyGraph / Keep Your Receipts 全文笔记
- 来源: https://arxiv.org/html/2610.17719 （verified live, 2026-10-29）；abs https://arxiv.org/abs/2610.17719v1 提交 2026-10-13；单作者 Vishisht Choudhary (TU Munich)；NeurIPS 2026 workshop "Who Verifies the Agents?" preprint, camera-ready 可能不同
- 定义: assertion x=<d,s,p,o>; artifact 四层 claims/merges/entailed edges/summaries; receipt/support label L(a)={S1..Sm}, Si⊆X, 上限 8 组取最小; 存活判据: ∃Si 与 D 不交
- 删除: retract-source (按文档), revoke-relation (按谓词)
- 语料: 300 PubMed oncology abstracts (61 真撤稿), 2,785 assertions/2,784 claims/165 merges/59 summaries/21,373 entailed edges=24,381 artifacts; 第二域 150 Supreme Court opinions (1,571/136/60/6,137); constructor Claude Sonnet (2026-08, sonnet alias, content-addressed cache), 后续 pin claude-sonnet-5 / claude-opus-5-5; 第二 constructor Opus
- 审计: re-derivation R=5, τ=0.2 (≥80% alive, ≤20% dead, 中间 undecided 排除); liveness ≠ groundedness (作者明示)
- 结果: deployed 污染 32.2%→28.0% (retract), revoke 36.8% 不变; replay floor 29,475 vs deletion effect 6,248 (5×); cleanup 1-21 ms 无模型调用; merges agreement 68.4%/84.6%, summaries flat 33.3%/36.4% 且几乎全 leaky; 十次额外删除 pooled: merges leaky 18.1%, summaries 41.3%; sentence-level receipts 100%/87.6%, 保留 33/35 touched summaries (5/7 + 28/28); whole-summary 100%/90.5% 但不 informative (全删); layered policy tainted@10 0.4%/0.8%, recall@10 99.9%/99.8% (vs trust-all 0.8%/2.1%); 第二域 distrust 后 0.0% 污染, recall 99.9-100%
- 成本: labels 1.35MB/5.45MB graph, 边际 ~10% bytes; receipts ≈ 构建输出 token 的 1/4, 本次约 $0.30
- 代码: 论文脚注给 https://github.com/VishiATChoudhary/polygraph 并称 prompt templates released verbatim with the code; **2026-10-29 实际打开该 URL 返回 404，未能确认公开可用**（未重复请求）
- 局限 (§8): 只测 liveness 非 groundedness（erasure 合规最关键的失败测不到，人审未做）; 两域两 constructor 同一厂商; 语料 150-300 文档规模未测; BM25 检索非路径推理; 只研究删除不研究新增; entailed edges 由构造一致被排除, faithfulness 只覆盖模型作者的少数 artifacts; R=5 τ=0.2 粗粒度
