# LETHE / Control-Plane Placement — arXiv:2606.15903（全文已读，3665 行 HTML 全部翻完）+ 仓库核查
- 论文 URL: https://arxiv.org/html/2606.15903 ；仓库: https://github.com/deeplethe/lethe
- 仓库内早期版论文 PDF（2026-05-13，23 页，题 ForgetEval: Benchmarking the Forgetting Axis）: https://github.com/deeplethe/lethe/raw/refs/heads/main/paper/paper.pdf（全文关键节已读，§5.4 回执规格在此）
- 读取：2026-10-29。论文=index 全文；仓库=verified live（研究窗口实开：主页、recipes/、recipes/02_gdpr_purge_receipt.py）。
- 作者：Dongxu Yang（DeepLethe，单作者）。

## 论文（arXiv 版）关键事实
- 划分：recall plane（检索已存事实，只读热路径）vs control plane（mutate：supersede/release/purge）。论文问题=LLM 放在哪个面/何时（inscribe-time 写入时 / mutation-time 变更时 / 不放），而非有没有 LLM。遗忘层级明确：memory-store layer alone, no parameter access（§2）——外置记忆存储层，不做参数 unlearning/editing；执行态只涉及其自有 event log（审计），不治理 agent 运行态。
- ForgetEval：1000 模板（seed=42、4 distractors/案、确定性生成）+ 385 对抗（132 手工 core + 253 LLM-drafted(DeepSeek-V3 起草)/oracle-validated(Qwen-2.5-72B 两阶段准入)）。用例 dataclass：setup_facts、mutations、final_query、must_contain、must_not_contain；pass ⟺ must_contain ⊆ top-10 blob 且 must_not_contain ∩ top-10 = ∅；确定性子串匹配、无 LLM judge（judge 只用于数据准入）。五族：supersession/decay/amnesia/purge/drift；对抗 10 类。IAA：10 标注者 100 案 Fleiss κ=0.958；77 案外部作者子集（4 人、盲写）。
- Adapter Protocol：3 必需（reset/inscribe/recall_texts）+ 3 可选（supersede/release/purge，NotImplementedError→N/A）；行为契约，可用 add+delete 组合实现；适配器 ~130 行。
- 参考实现 Lethe：单 SQLite，memory/memory_vec/memory_fts 共 rowid + append-only event 表 + supersession 边表；单标量 depth（+∞ pinned、1.0 surface、(0,1) sinking、0 submerged but logged、void erased）；recall=RRF(vec+BM25)，purge 用纯 BM25 lexical（按 identifier 硬删，GDPR Art.17）；可选 llm hook = 三个窄 JSON prompt（supersede planner / purge match / release match，附录 B 只给标题）。
- purge 是硬删（仓库论文 Prop.3 证明：row 从 memory/memory_vec/memory_fts 三表全删，rowid 不复用 AUTOINCREMENT；event log 只留 purge 事件作审计、不可恢复）。release 是软删（depth 压至 submerged，TTL/OTP）。
- 主结果：模板集 Lethe 993/1000=99.3%、LangMem 99.5%、Mem0 88.8%（amnesia/purge 塌）、MemPalace 0。对抗 385：确定性 Lethe 63.4 / Mem0 68.3 / LangGraph 62.9（63–68% 带、Wilson CI 重叠、McNemar p=0.724）；Lethe+LLM(DeepSeek-V3 hook) 91.7%（ excl compound 93.3%）、LangGraph+LLM 93.2/94.2%。确定性在 identifier_obfuscation ≤5%、cross_lingual 0%；inscribe-LLM 规范化 100% 但 prefix_collision/compound_fact 0%；mutation hook compound 78–85%、prefix 79%。非 primitive-existence 的 345 案：确定性 70.1–70.7 → hook 93.3–94.2（+22.6~24.1pt）。成本 ~$0.17/385 案、hook 延迟 ~2.3s/案 vs 确定性 64–191ms、recall 热路径不变。
- 消融：embedder 换多语 MiniLM 不改变 cross-lingual（purge 走 BM25）；跨 LLM：Qwen-2.5-72B hook +13pt、Llama-3.1-70B JSON 解析失败回落确定性；跨后端 hook 提升一致（架构无关）；Mem0 infer=True 68.3→43.6%（规范化 +45~53、删除精度 −31~92）；naive SQL DELETE-LIKE 40.5% vs Lethe 63.4%（−23pt）。
- 跨评测：Memora-weekly 150 题（Lethe 31%、LangGraph 45%、MemPalace 40%——轴翻转）；FactConsolidation 400 题（单跳 100%、多跳 17–37%，hook 与确定性同分因 hook 不触发）；LongMemEval-S 附录 Q：Lethe R@5=93.8%（session 粒度，knowledge-update 78 题 0.987）【注：仓库 README 写 R@5=97.4% 是另一口径 sub-document QA，两数并存，引用须注明口径】。
- 外部子集 77 案：确定性带跌至 28–33%，hook +11.7/+18.1pt，Lethe+LLM cross_lingual 反为 0/8（外部脚本对其 prompt 规范化器未覆盖）。
- 局限（自述）：子串评分盲点（50 案审计 30% 为 scorer artifact，全在 prefix_collision/条款级 supersession）；英文为主、跨语言每脚本样本薄；compound_fact 是 primitive-existence 测而非遗忘能力测；253 案 LLM 起草+单 judge 准入的循环性不能全排除；Lethe 与 LangGraph 聚合分无显著差异、参考实现定位是 anchor 非竞争系统；Zep 未实测（docker 约束）等。
- 签名回执在 arXiv 论文正文中的地位：仅附录 A 一句——每次状态转移写一行 append-only event 表，「enabling time-travel queries and signed purge receipts」；附录 B–V 无回执评测；附录 V 给仓库 MIT 链接。回执不是论文的实验对象。

## 仓库（verified live，2026-10-29）
- Public，86 commits，最新 2026-06-16；目录 lethe/ bench/ docs/ paper/ recipes/ tests/；MIT；PyPI 名 pylethe；MCP 11 工具含 signed-purge receipts；README 自述 v1.0.0-alpha、pytest 14 passed。
- README Architecture：「Verifiable forgetting. Every signed purge returns an Ed25519-signed receipt anchored to a Merkle root over the event log. Tamper with any past event afterwards → receipt fails verification.」
- 回执字段（仓库论文 §5.4 原文）：purge_with_receipt() 发 PurgeReceipt，含 purged row ids、删除前 purged texts 的 SHA-256 hashes、对应 purge events 的 event-log IDs、签发时全 event log 的 Merkle root；以实例 Ed25519 私钥签名；签名对象=canonical CBOR payload；签发/验证 O(log n)、亚毫秒。第三方可验：(a) 删了哪些行 (b) 此后 event log 未被篡改 (c) 由合法运营者签发——且无需接触已删数据。
- 验证流程（recipes/02_gdpr_purge_receipt.py 实读，72 行）：generate_key() → Lethe(signing_key=...) → inscribe → purge_with_receipt([id]) → payload 打印键 purged_ids / purged_text_hashes / event_log_merkle_root / signature → verify_receipt(receipt, agent) 带 DB 验（重算 root+验签，result["valid"]）；verify_receipt(receipt, lethe=None) 仅验签（result["signature_valid"]）。【注：该脚本 docstring 称回执为「Ed25519-signed JSON document」，与仓库论文「canonical CBOR payload」表述不一致——版本/文档漂移】
- 路线图（README Status/Roadmap）：「Receipt-verification benchmark family」列为 next——即回执本身尚无基准评测。
