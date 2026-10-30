# BEMULA 论文 + 仓库（2026-10-29 读取）

## 论文 arXiv:2610.18052v1（index 打开 HTML 全文 1233 行）
- URL: https://arxiv.org/html/2610.18052v1 ; abs: https://arxiv.org/abs/2610.18052v1
- 50 实例 / 746 sessions / 36,900 turns per version / 135 请求 / 269 target facts / 1,742 题；retention 224, efficacy 224, relearning 114, attack 1030, preservation 150（每实例恰 3 题）。
- 请求分布：F1 32, F2 23, F3 26, F4 13, F5 20, F6 21；5 种表达 style 各 24–29；target 提及 0–5 次平均 2.5；52 请求多于 1 个 target fact。
- 长度：short 15 实例 96–164 turns；medium 16, 334–472；long 16, 916–1464；xlong 3, 2688–3500 turns, 平均 84.8K tokens/实例, max 379K。
- 指标：RA/FE/PR/RLA item-level，AR=5 类攻击平均，RPR 定义（请求全部题∧本实例全部 preservation），Score=√(FE·AR)×√(PR·RLA)（以分数计；任一为 0 则总分 0）。
- 有效正确性：FE/PR/AR 都以 control 探针为条件（z̃=z∧c）；F6 新值题与 retention 无 control 对应 c=1。
- 主结果：RPR 最高 1.5%（LangMem/DeepSeek-V4.1-Flash），Score 最高 31.0 同配置；GPT-5.6-Luna 最高 Score 23.1 Full Context；Qwen3-32B-AWQ 最高 9.5 SimpleMem。
- LangMem GPT backbone：FE 37.9 但 PR 20.0（过删，删到未被请求的 street address）；DeepSeek 上 FE 40.6/PR 74.0 — backbone 依赖极强。
- Full Context DeepSeek：PR 46.0，81 个 preservation 失败中 66 个是 blanket "I don't have that information" 过度拒答。
- F6：强 backbone 上 FE ≥61.9 近乎解决，但旧值逐字存活、AR ≤30.5（全系统）；F3/F5 最难（跨多条目范围删除），只有 Full Context 与 LangMem 曾 >30；F1 Mem0/GPT 为 0.0。
- AR 天花板 ≤35.7；Mem0/GPT AR 4.7。攻击分项（附录 H）：删太少的系统最怕 indirect 与 Boolean（无需技巧）；Full Context 最怕 jailbreak（9.7 GPT）。
- 失败机理（§4.3+附录 C）：RAG 无删除通道、请求被当普通 turn 存；Mem0 删除靠其内部 add/update/delete 决策、失败答案常逐字给出值；LightMem update/merge 离线；SimpleMem 窗口压缩一次永不改；A-MEM 无删除但 evolution 可覆写相关 note（FE 33.0/29.5，F1 46.2/F2 52.0 GPT 最高）；LangMem 是唯一对话内 removal 通道（stock 默认关闭、作者启用；且只作用于当前 batch 检索到的 notes）。
- 长度效应（附录 J）：LangMem FE 随长度 66.7→21.9；PR 普遍随长度下降；AR 随长度几乎不动=结构性泄漏；Qwen 上 backbone 能力是绑定约束。
- 协议（附录 A/C.1）：统一 adapter 三调用 reset/observe/query；同 agent LLM 答题；embedding 统一 text-embedding-3-small；检索预算 stock（RAG 5, A-MEM/LangMem/LightMem 10）；每实例前 reset；问题在 fresh session 问、答案不写回；两遍：control pass（retention+pres/attack 的 control 探针）+ forget pass（reset 后 efficacy/preservation/attack/relearning 顺序）。
- F4 语义（附录 A）：源事实不在 preservation 探针内；保源+抑制推断 或 连源一起删 都合规；control 必须能推出结论。
- Judge：全题 LLM judge，DeepSeek-V4.1-Flash，temperature 0（run 内固定）；人工验证 100 题（正负平衡采样、盲评）一致率 98%、κ=0.96；数据生成用 GPT-5.6-Luna seed 42 temp 0.9；人工抽检 10 实例 388 题。
- 统计（附录 K）：对实例 cluster bootstrap 10,000 次（同抽实例集合并两版本全部题、保条件化），配对比较用相同 instance draws 的差值 CI；例 LangMem vs Full Context DeepSeek Score 差 2.5, 95%CI [-2.4,7.6]=并列；vs SimpleMem 差 14.5 [9.5,19.8]。不对 RPR 报 CI（几乎全 0）。per-type/per-tier 单元少（F4 仅 13 请求、xlong 仅 3 实例）只作描述性。
- Lite（附录 F）：31 个 short+medium 实例、848 题，丢 long/xlong（成本主导）；版本/套件/指标不变、与全量可比。Lite 上 DeepSeek LangMem Score 36.1 / RPR 1.6。
- 局限（作者自述）：合成数据、单一生成管线、仅对话记忆（不含工具 cache/log）；judge 可能有模型偏差。
- 复现：seed 42（configs/default.yaml），run_meta 记录 agent/judge/config；无专用硬件，但需 LLM API（OpenAI 兼容端点）。

## 仓库 github.com/Nboxff/bemula-benchmark（verified live 2026-10-29, commit 862aff6, 仅 1 commit, 2026-10-10）
- README（index+live）：uv sync（Python>=3.10）；cp .env.example .env 填 OPENAI_BASE_URL/OPENAI_API_KEY/BEMULA_*_MODEL；所有 LLM 调用走单一 OpenAI 兼容端点。
- 跑评测：uv run python -m scripts.run_eval --baseline simplemem --dialogues data/core/dialogues.jsonl --questions data/core/questions.jsonl --out results/... --workers 8 --resume（README 示例）；可用 baselines: rag, mem0, amem, simplemem, lightmem, langmem, fullcontext；--limit N 只跑前 N 实例（smoke）；--workers 每 worker 一个 adapter；--resume 跳过 out/items.jsonl 已记录实例；输出 items.jsonl/metrics.json/report.md/run_meta.json。
- ⚠️路径不一致（live 实测）：实际 data/ 下只有 full/、lite/、quick_check/ 三目录 + human_eval_annotations.csv；data/lite 内含 dialogues.jsonl/profiles.jsonl/questions.jsonl。README 写的 data/core、data/quickcheck、data/smallcheck 与实际树不一致；run_eval.py docstring 示例写 data/quickcheck/ 亦然。clone 后须以实际树为准。
- baselines/base.py（live）：MemoryAdapter(ABC)：reset() 必须清全部用户状态；observe(turn: Turn) 每 turn 按序调用（control 与 forget 两版本都走）；query(question: str)->str 只从当前记忆答、不得写回（configs probe_writeback:false）；可选钩子 begin_instance(instance)/end_instance()/begin_session(session)/end_variant 等（TurnProgress 用）。name 属性。
- scripts/run_eval.py（live）：_build_baseline(name, llm, embedder_provider) 是硬编码 if 链注册表（rag/mem0/amem/simplemem/memoryos/lightmem/langmem/fullcontext），自研系统须在此加分支+新建 bemula/baselines/<x>.py（无插件/配置注册机制——此为读代码得出的结论）；judge=LLMJudge(get_llm(BEMULA_JUDGE_MODEL), temp=config evaluation.judge_temperature 默认 0)；agent=get_llm(BEMULA_AGENT_MODEL)；--embedder-provider openai|huggingface（由 BEMULA_EMBED_BACKEND api|local 推导）；checkpoint 每实例即时 append items.jsonl；metrics 的 report 自动含 by_attack_type、by_forget_type、by_length_bucket、RA-gated 与 raw/conditioned 两套 FE/PR。
- ⚠️ run_eval 注册了 memoryos 并 from bemula.baselines.memoryos import，但 baselines/ 目录 live 列表（amem.py, base.py, fullcontext.py, langmem.py, lightmem.py, mem0_adapter.py, rag.py, serving.py, simplemem.py）无 memoryos.py —— 用该 baseline 前须核实（可能缺文件）。
- configs/default.yaml（live）：seed 42；evaluation.judge_temperature 0.0；probe_writeback false；generation gen/sim model gpt-5.6-luna。
- .env.example（live）：OPENAI_BASE_URL 例 https://api.deepseek.com/v1；BEMULA_GEN_MODEL=gpt-5.6-luna；BEMULA_JUDGE_MODEL=deepseek-flash；BEMULA_AGENT_MODEL=deepseek-flash；BEMULA_THINKING=disabled（避免 DeepSeek/gpt-5*/Qwen3 隐藏 reasoning tokens）；BEMULA_EMBED_BACKEND=api|local；BEMULA_EMBED_MODEL=text-embedding-3-small；local 时 BEMULA_HF_EMBED_MODEL=all-MiniLM-L6-v2；基线官方仓库路径 BEMULA_AMEM_PATH/SIMPLEMEM_PATH/MEMORYOS_PATH/LIGHTMEM_PATH（未设则找 sibling ../<Repo>）。
- README 基线坑：amem/simplemem/memoryos/lightmem 从官方仓库 clone 加载；simplemem 需 --with 额外包且 lancedb<0.37（production 示例钉 lancedb==0.25.3, pyarrow==17.0.0, pylance, dateparser, tantivy）；或 uv sync --extra baselines 一次装齐。
- schemas.py（live）：Turn{speaker, text, timestamp, linked_fact_ids[]}；DialogueInstance 含 forget_requests[]（含 target_fact_ids, request_text, time_scope, replacement）与 control_sessions；TestItem 含 gold_answer/rubric/related_fact_ids。⚠️污染风险：observe 收到完整 Turn（含 linked_fact_ids），begin_instance 收到完整 instance（含答案侧元数据）——合规实现只能用 speaker/text/timestamp，必须自设字段白名单并在论文声明，否则等于偷看标注。
- RequestStyle 5 种：direct/polite/indirect/imperative/implicit；F5 TimeScope 有 dialogue_time 与 event_time 两模式（schemas 注释）。
