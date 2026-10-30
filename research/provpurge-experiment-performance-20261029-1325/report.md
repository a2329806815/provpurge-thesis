# ProvPurge 实验设计与提分方法调研

> 对象：LLM Agent 长期记忆遗忘系统 ProvPurge（写时句级来源 + 确定性级联清除〔事实/向量/摘要句三层〕+ 删除后攻击探针自验）。
> 约定：【已证实】=本次实际打开页面确认（注明 live/全文/索引）；【建议/推导】=基于已证实事实的设计推论，非原文结论；【未确认】=读不到或未核实，明确标注，不编造。调研日期：2026-10-29。不运行任何实验。

## Summary

1. **流水线已证实可落地**：BEMULA 仓库的统一接口是 `bemula/baselines/base.py` 的 `MemoryAdapter`（live 核实）：只需实现 `reset() / observe(turn) / query(question)` 三个抽象方法，再在 `scripts/run_eval.py` 的硬编码注册函数 `_build_baseline` 里加一个分支即可跑官方评测；输出 `items.jsonl / metrics.json / report.md / run_meta.json`，且官方 `report.md` 自动含逐攻击类型、逐 F1–F6、逐长度档分项。但仓库是首发单 commit，README 路径与实际目录树不一致（见 §1.4），clone 后必须以实际树为准。
2. **提分的第一性原理（推导）**：Score 恰等于 FE、AR、PR、RLA 四者的几何平均（验算 LangMem/DeepSeek：(0.406×0.357×0.740×0.895)^¼=31.0 ✓），所以**涨分效率由最低的那项决定**——现成系统 AR 天花板仅 35.7、FE 普遍 <40，AR 与 FE 是主战场；任何一项崩掉（如 LangMem 在 GPT backbone 上 PR 20.0）都会乘性拖死总分。RPR 是合取指标：平均每条请求约背 13 个二值检查（功效 ≈1.7 + 攻击 ≈7.6 + 重学 ≈0.9 + 本实例共享的 preservation 3 题），per-item 正确率 0.8 时通过率量级也只有个位数百分比——RPR 从 1.5% 往上爬靠的是**每一项都到 0.85+**，不是某一项到 1.0。
3. **硕士周期内最值得先做的 3 件**（详见 §3.3）：①触发识别+范围解析与「来源反查候选集」（解决论文实证的头号失败“请求根本到不了存储层”与 MemCut 实证的头号瓶颈“候选覆盖率”）；②删除后自验闭环，且探针必须包含**绕过自身检索的 residual 式直查**（MemCut 证明只走自身检索会系统性低估残留，hidden-risk share 删除后高达 39–92%）；③读时作用域防线（tombstone 过滤 + 前提检查 + 只对被删槽位拒答），低成本同时护 AR 与 PR，有 PACMI/CUPMem 的已发表支持。
4. **最大的两个暗坑**：(a) `observe()` 收到的 `Turn` 对象带 `linked_fact_ids`、`begin_instance` 能看到含 `target_fact_ids` 的完整 instance——**用这些字段等于偷看标注**，必须自设字段白名单并在论文声明；(b) **Lite 的 31 个实例是全量 50 个的子集**，在 Lite 上调阈值再报全量分数存在调参泄漏，须用 quick_check（重叠未确认，须自查 instance_id）或自生成 dev 集调参，或把 long+xlong 19 个实例作为未调参切片单独报告。

---

## Findings

### 1. 实验流水线：BEMULA 官方仓库接入、基线复现、backbone/judge 与成本

#### 1.1 已证实的仓库事实（live 核实，commit `862aff6`，2026-10-10 首发、仅 1 个 commit）

- 仓库：https://github.com/Nboxff/bemula-benchmark ；论文：https://arxiv.org/html/2610.18052v1
- 安装与配置（README，index+live）：`uv sync`（Python ≥3.10）；`cp .env.example .env` 填 `OPENAI_BASE_URL / OPENAI_API_KEY / BEMULA_*_MODEL`；**所有 LLM 调用（生成、judge、基线）走同一个 OpenAI 兼容端点**。
- `.env.example`（live 核实）关键变量：`BEMULA_AGENT_MODEL`（基线答题/内部调用）、`BEMULA_JUDGE_MODEL`（评测 judge）、`BEMULA_GEN_MODEL`（数据生成）、`BEMULA_THINKING=disabled`（注释称可避免 DeepSeek/gpt-5*/Qwen3 的隐藏 reasoning token）、`BEMULA_EMBED_BACKEND=api|local`、`BEMULA_EMBED_MODEL=text-embedding-3-small`、local 时 `BEMULA_HF_EMBED_MODEL=all-MiniLM-L6-v2`，以及各基线官方仓库路径 `BEMULA_AMEM_PATH / BEMULA_SIMPLEMEM_PATH / BEMULA_MEMORYOS_PATH / BEMULA_LIGHTMEM_PATH`（未设时找同级目录 `../<Repo>`）。
- 评测命令（README 示例）：`uv run python -m scripts.run_eval --baseline simplemem --dialogues <dialogues.jsonl> --questions <questions.jsonl> --out <dir> --workers 8 --resume`。`--limit N` 只跑前 N 个实例（smoke run）；`--workers N` 每 worker 建一个 adapter；`--resume` 跳过 `items.jsonl` 中已完成的实例（run_eval.py 中每完成一个实例即时 append 落盘，中断可续）。
- 内置 baselines（README）：`rag, mem0, amem, simplemem, lightmem, langmem, fullcontext`（run_eval 注册表里另有 `memoryos`，见 §1.4 的坑）。

#### 1.2 自研系统要实现的接口（live 核实 `bemula/baselines/base.py` 与 `scripts/run_eval.py`）

**已证实**：所有系统被包成 `MemoryAdapter`（ABC），契约为：

| 方法 | 签名（代码原文语义） | 契约 |
|---|---|---|
| `reset()` | 抽象 | 必须清掉**全部**用户状态；每个 dialogue version（control/forget）评测前调用，实例间隔离 |
| `observe(turn)` | `observe(self, turn: Turn) -> None`，抽象 | 每 turn 按序调用一次，control 与 forget 两版本都走；`Turn` 字段为 `speaker / text / timestamp / linked_fact_ids`（schemas.py，live 核实） |
| `query(question)` | `query(self, question: str) -> str`，抽象 | 只从当前记忆答题；**不得写回记忆**（`configs/default.yaml` 中 `evaluation.probe_writeback: false`，live 核实） |
| 可选钩子 | `begin_instance(instance) / end_instance() / begin_session(session) / end_variant()` 等 | 非必须；`begin_instance` 会收到完整 instance 对象（⚠️含答案侧元数据，见 §5.1） |

**已证实**：注册方式是改代码而非配置——`scripts/run_eval.py::_build_baseline(name, llm, embedder_provider)` 是硬编码 if 链（`rag/mem0/amem/simplemem/memoryos/lightmem/langmem/fullcontext`），自研系统须：①在 `bemula/baselines/` 下新建如 `provpurge.py` 写 `ProvPurgeAdapter(llm, embedder=...)`；②在 `_build_baseline` 加分支；③用 `--baseline provpurge` 运行。构造参数惯例：adapter 接收 runner 传入的 `agent_llm`（`get_llm(BEMULA_AGENT_MODEL)`）与 embedder（`get_embedder()`，按 `BEMULA_EMBED_BACKEND` 为 api/local）。

**已证实**：评测协议（论文附录 A/C.1）：每个实例跑两遍——control pass（ingest control 版，答 retention + preservation/attack 的 control 探针）→ reset → forget pass（ingest forget 版，依次答 efficacy、preservation、attack、relearning）；每道题在只含该题的 fresh session 中问，答案不写回；relearning 题先 ingest 其附带小对话再发问。FE/PR/AR 的每题得分都以 control 探针为条件（$\tilde z=z\wedge c$），即“没存住的目标删得再干净也不得分”。judge 在一个 run 内固定、temperature 0（`configs/default.yaml: evaluation.judge_temperature: 0.0`）。

**接入步骤（建议，按上述已证实接口组织）**：
1. clone 仓库 → `uv sync` → 配 `.env`（先全部指向最便宜的可用端点）→ 用内置 `rag` 在 `data/quick_check/` 上跑通全链路（验证 judge/端点/输出四件套）。
2. 写 ProvPurge adapter（只用 `turn.speaker/text/timestamp`，见 §5.1 白名单）；先在 quick_check 上 `--limit` 冒烟，再对照 `metrics.json` 检查 RA 是否正常（RA 崩说明写路径坏，不要先查删除逻辑）。
3. 把内部埋点（触发是否命中、候选集大小、每层删除数、自验轮次）写到 adapter 自己的日志文件——**不要改官方 `items.jsonl` 格式**，诊断靠旁路日志按 instance_id 对齐。
4. 依次放大：quick_check → Lite（`data/lite/`，31 实例）→ 全量（`data/full/`，50 实例），每档冻结一份 run_meta 与配置快照。

#### 1.3 Lite 与全量的取舍（已证实 + 推导）

- **已证实**（论文附录 F）：Bemula-Lite = 31 个 short+medium 实例、848 题，丢掉 long/xlong 两档（916–3,500 turns/实例、最长约 379K tokens，是成本主导）；两版本、五套件、指标定义不变，与全量可比。live 核实仓库实际目录为 `data/{full, lite, quick_check}/`，每档含 `dialogues.jsonl / profiles.jsonl / questions.jsonl`，另有 `data/human_eval_annotations.csv`（官方人工核查标注，可用于自己的 judge 复核对照）。
- **成本结构（推导，基于论文附录 D 的分档统计）**：写侧成本 ∝ turns，问答+judge 成本 ∝ questions。按分档 turns 区间中点估算：short ≈15×130 + medium ≈16×403 ≈ 8,400 turns（占全量 36,900 的约 23%），但 Lite 题量占全量 848/1,742≈49%。即 **Lite 的写侧成本约全量的 1/4、问答侧约 1/2**；从 Lite 外推全量必须分开乘：`C_full ≈ C_lite_write × (36900/8400) + C_lite_qa × (1742/848)`，不能用单一比例。用 quick_check/Lite 各跑一次并分别记录两类 token（adapter 埋点 + 端点账单），即可得到本系统的单位成本，再代入上式。【未确认】官方未公布任何美元/时长成本数字，论文与 README 均无；quick_check 的实例数本次未核实（须本地 `wc -l` 确认后再用它做最小冒烟）。
- **取舍建议**：开发迭代只用 quick_check + Lite；主结果必须报全量（Lite 与全量的系统排序大体一致但数值不同，如 Lite 上 DeepSeek/LangMem Score 36.1 vs 全量 31.0，附录 F/K 明确 Lite 的部分差距 CI 跨 0）；消融可先在 Lite 筛、胜出配置再上全量复核（注意 §5.5 的子集调参泄漏问题）。

#### 1.4 基线复现的顺序与坑（已证实部分逐条标注）

建议顺序（按依赖复杂度从低到高，【建议】）：
1. **rag**：无外部仓库依赖，先验证评测链与 judge；它也是“无删除通道”的下限参照（论文：请求被当普通 turn 存入，FE 7.1–14.7）。
2. **fullcontext**：无存储、纯指令跟随，给出“读时防线 alone”的参照（但 Qwen3-32B-AWQ 因上下文长度被官方排除于全量 Full Context 与 A-MEM 评测，论文 §4.1 已证实）。
3. **mem0**：`uv sync --extra baselines` 可装；其删除靠内部 add/update/delete 决策，不要从 adapter 外部发 delete 调用（论文附录 C.2 已证实官方就是这么测的，外部 delete 会改变被测对象）。
4. **langmem**：⚠️论文附录 C.2 已证实“其 stock 配置默认**关闭**对话内 removal 通道、作者手动启用”；复现论文数字前必须核对仓库 `langmem.py` 的实际配置——**本次未打开该文件确认其默认开关状态，标【未确认】**，这是复现 LangMem（最高分基线）的头号坑。
5. **amem / lightmem / simplemem**：从各自官方仓库 clone 加载（README 已证实），路径用 `.env` 的 `BEMULA_*_PATH` 或同级目录；`simplemem` 需额外 `--with` 包且 **`lancedb` 必须 <0.37**（README 已证实；production 示例钉 `lancedb==0.25.3, pyarrow==17.0.0, pylance, dateparser, tantivy`）。这类基线对 clone 的版本极敏感，务必记录 clone 的 commit hash（仓库本身只有 1 个 commit，基线仓库版本未被官方锁定——【已证实】run_meta 只记录本仓库 git commit，不含基线 clone 版本，需自己补记）。

其他已证实的坑：
- **README 与实际目录不一致**（live 核实）：README 示例写 `data/core/`、`data/quickcheck/`、`data/smallcheck/`，实际树只有 `data/full|lite|quick_check/`；`run_eval.py` 的 docstring 示例同样写 `data/quickcheck/`。照抄 README 命令会直接报路径不存在——以实际树为准，并把这一条写进自己的复现日志。
- **memoryos 注册了但文件缺失**（live 核实）：`_build_baseline` 中有 `from bemula.baselines.memoryos import MemoryosAdapter`，但 `bemula/baselines/` 目录列表（amem/base/fullcontext/langmem/lightmem/mem0_adapter/rag/serving/simplemem）中**没有 memoryos.py**。跑该 baseline 前须核实，缺失时不要浪费时间排查自己的环境。
- 论文附录 C.1 已证实：官方检索预算是各系统 stock 配置（Naive RAG 5 条、A-MEM/LangMem/LightMem 10 条），embedding 统一 `text-embedding-3-small`；复现时改这些就不再是官方口径，须在论文中明示。

#### 1.5 backbone 与 judge 的选择和成本控制（已证实 + 建议）

- **已证实**：论文用三 backbone——GPT-5.6-Luna、DeepSeek-V4.1-Flash、Qwen3-32B-AWQ；系统内全部 LLM 调用共用同一 backbone；judge 固定 DeepSeek-V4.1-Flash、temperature 0；数据生成用 GPT-5.6-Luna（seed 42、temp 0.9）。
- **已证实**：backbone 摆动极大——同一 LangMem 从 GPT（FE 37.9/PR 20.0/Score 12.4）到 DeepSeek（40.6/74.0/31.0）到 Qwen（14.7/5.3/2.6）；Full Context 的 FE 从 26.8（GPT）到 53.6（DeepSeek）。论文结论：Qwen 上“抽取阶段本身崩掉”、backbone 能力是绑定约束。**推论【建议】**：单 backbone 的结论不可信，主表至少 2 个 backbone；选型先在 Lite 上用 RA+FE 快速筛——RA 低的 backbone 说明写/抽取不行，换再多删除策略也救不回来。
- **成本控制【建议，均有机制依据】**：①`BEMULA_THINKING=disabled` 避免隐藏 reasoning token（.env.example 注释已证实此开关用途）；②judge 固定便宜且与论文一致的模型，不要用被测 backbone 自评（自增强偏差，见 §5.2）；③embedding 开发期可切 local（all-MiniLM-L6-v2）省 API 费，但**正式结果须用官方 `text-embedding-3-small`** 保持可比口径并在文中注明；④`--workers` 并行 + `--resume` 断点续跑是官方给的两个省时机制，后台长跑用 README 的 production 形式（nohup+日志）；⑤分档外推公式见 §1.3，先 quick_check 测单位成本再决定全量跑几次。

### 2. 性能瓶颈定位：按指标结构的诊断方法

先立两个总框架（推导部分已标注）：

- **Score 杠杆**：Score = (FE·AR·PR·RLA)^¼（几何平均，见 Summary 验算）。四项中最低者边际收益最大；现成系统 AR≤35.7、FE≤53.6（且记忆系统 FE≤40.6），而 PR/RLA 多在 70–100——**瓶颈排序：AR ≈ FE > PR（防崩）> RLA（防崩）**。【已证实数据来源：论文 Table 1】
- **RPR 的合取结构**：请求通过 = 该请求全部题 ∧ 本实例全部 preservation 题。preservation 每实例仅 3 题却被该实例平均 2.7 条请求共享——**一个实例的过删会连坐杀死该实例所有请求**；攻击题占每请求检查数的大头（1,030/135≈7.6 题/请求）。诊断 RPR 时先看“每请求失败题数分布”（差 1 题 vs 差 5 题），比看 RPR 本身信息量大得多。【建议的呈现方式，结构已证实】

逐环节诊断表（“观测”列均为可用官方输出 `items.jsonl/metrics.json` 或自家埋点直接算出的量）：

| 环节 | 卡住的指标 | 可观测诊断 | 判读（已证实模式） |
|---|---|---|---|
| ① 触发识别（请求未被当成删除指令） | FE、AR、RPR 全军覆没 | 自家埋点：每条请求的 purge 是否触发；官方侧：按 **5 种表达 style**（direct/polite/indirect/imperative/implicit，schemas 已证实）与按类型拆 FE | 论文 §4.3 已证实“most requests never reach the store”：RAG 无删除通道、LightMem 更新离线、SimpleMem 压缩后永不改、Mem0 靠内部决策（失败答案常逐字给值）。若 direct style 的 F1 都失败 → 通道根本没通；只有 implicit/indirect 失败 → 分类器问题 |
| ② 候选召回（触发了但没找全要删的） | FE（部分题过、部分不过）、AR | 同一请求内 efficacy 题的通过模式；离线用自家来源图统计“候选集覆盖了几条目标事实的载体”；对照 MemCut 的覆盖率口径 | MemCut 已证实：池 20 只含约 4.5/8.1 条证据 turn，残余恢复与“删掉的证据数”近线性（-13 点/条，R²=0.95）。覆盖不足时再强的裁决也无米下锅 |
| ③ 事实层删除 | FE（direct efficacy 题） | efficacy 失败答案里是否**逐字出现目标值**（对 items.jsonl 答案做字符串匹配即可） | 逐字出现 = 事实条目还在（Mem0 模式）；答案说“不知道”但攻击能套出 = 事实层删了、残留在别层 |
| ④ 向量残留 | AR 的 indirect / Boolean 两类 | 按攻击类型拆 AR（官方 report 自动给 by_attack_type） | 论文附录 H 已证实：删太少的系统最怕 indirect 与 Boolean（无需任何技巧，说明近义改写向量仍能召回旧值）；jailbreak 最弱反而是“删得动”的系统 |
| ⑤ 摘要/派生事实重建 | AR 的 multi-hop、F4 的 FE | multi-hop AR 单独看；F4 功效单独看；自家存储审计：删后 dump 摘要全文搜索旧值及同义表述 | F4 目标从未在对话中明说，只能靠源事实推断（论文附录 A）；multi-hop 攻击正是“用留下的相关事实重建目标”。摘要整段留存旧句 = 句级手术没做 |
| ⑥ 读时泄漏（存储已净、答时仍泄） | FE 过而 AR 不过；或存储审计干净但答案泄 | 离线存储审计（对事实/向量/摘要三层做旧值与近邻搜索）与答案泄漏对照：审计干净仍泄 → 生成端把检索到的残片拼回，或自验/答题 prompt 把 tombstone 内容本身带进上下文 | STALE 的 LightMem 诊断已证实同类现象：新证据 77.5% 被检索到仍失败——“visibility ≠ authority”，读时没有裁决就会被旧条目带偏（方向相反但机理同为读时无防线） |
| ⑦ 过删误伤 | PR、连坐 RPR | PR 按实例看（哪几个实例 PR 崩）；FE↑同时 PR↓ 的 FE–PR 散点（论文附录 G 的画法） | LangMem/GPT 已证实：FE 最高 37.9 但 PR 20.0，连未被请求的 street address 都删。PR 随长度普遍下降（附录 J），长实例重点查 |
| ⑧ 过度拒答 | PR（答案是空白拒答）、RLA | preservation 失败答案中 blanket refusal 的占比（字符串/正则统计） | Full Context/DeepSeek 已证实：81 个 preservation 失败中 66 个是统一的“I don't have that information”→ PR 46.0 是全局拒答而非作用域删除 |
| ⑨ 重学阻断（tombstone 太宽） | RLA | RLA 按请求类型看；检查自家 tombstone 是“（槽位,旧值）级”还是“类别/谓词级” | RLA 现成系统普遍 ≥71 不是主瓶颈，但 ProvPurge 若把 F3 的类别删除做成类别级写拦截、或把槽位置为永久封锁，会亲手打崩 RLA（relearning 题正是同类型新值）【推导】 |
| ⑩ 没存住（地基） | RA 低 → 全部 conditioned 指标缩水 | 先看 RA 与 RA-gated 指标（run_eval 报告含 gated 与 raw 两套 FE/PR，已证实） | RA 低时不要误判删除策略：先修写路径/换 backbone（Qwen 上抽取崩坏即此情形） |

补充已证实的结构性事实，供定位时对照：AR 随对话长度几乎不动（附录 J）→ 泄漏是结构性的、不是长文淹没；LangMem 的 FE 随长度从 66.7 跌到 21.9 → 检索式删除通道在长存储里覆盖率衰减，正是来源反查要替代的东西；F6 功效在强 backbone 上 ≥61.9 近乎“解决”，但旧值逐字存活使其 AR ≤30.5 → **F6 的问题不在 FE 在 AR，提分动作是旧值清除审计而非替换逻辑**；F3/F5 最难（跨多条目范围删除，仅 Full Context 与 LangMem 曾 >30）。

### 3. 提分方法清单（核心章节）

#### 3.1 逐环节手段（预期收益/代价/证据）

**A. 触发识别与范围解析**
- A1 **混合触发器**：确定性模式（显式 delete/forget 动词+宾语）覆盖 direct/imperative；小 LLM 分类器覆盖 polite/indirect/implicit（官方 style 分布近均匀，24–29 条/种，不能只处理显式句式【已证实】）。输出统一解析为 `(type, target_spec)`：F1/F2=事实槽位列表，F3=类别，F5=时间窗，F6=(槽位, 旧值→新值)，F4=派生结论描述。
- A2 **解析后回执校验**：触发后先在自家事实索引里验证 target_spec 能落到真实条目（落不到→扩大解析重试一次，仍不到→记录未命中日志），避免“识别了但删了个寂寞”。
- 预期：FE/AR 的地基（论文头号失败模式）；伤：误触发会伤 PR——用 A2 的落点校验+高精度阈值（宁漏勿误，漏删只伤该请求，误删连坐整实例 preservation，代价不对称【推导】）。

**B. 候选集构建（覆盖率是 MemCut 实证的头号瓶颈）**
- B1 **来源反查（ProvPurge 本体）**：候选 = 写时句级来源图中与目标事实相连的全部载体（原始 turn、事实条目、向量块、摘要句、派生事实），集合运算确定性得到，不经相似度检索。证据：PolyGraph 已证实来源标签删除是集合算术、24,381 artifact 图 1–21 ms、无模型调用；MemCut 已证实检索式候选池覆盖只有约一半证据。
- B2 **多路扩展补残留**（针对来源图建成之前/之外的残留，如旧摘要、缓存）：实体链接（请求中实体→同实体全部条目）+ 类别索引 + 时间窗索引三路并集，再用向量相似只作**补充路**（大池、宽进）。证据：MemCut 固定 budget=10、池 20→60 使 residual recovery 52.1→39.6，且多删的非证据 turn 反而更少；而 budget 10→20 只换 -1.39 点却多删 9.95 个非证据 turn——**扩池 > 加预算**【已证实】。关系链证据与请求共享实体不共享措辞、相似度排名 3–141，实体路正是为它准备的【已证实】。
- B3 **派生事实追踪**：写时为每个派生事实登记 support 集（若干可替代支持束，任一束存活则派生事实存活——PolyGraph receipt 语义【已证实】；BEMULA 自己的数据模型 `AtomicFact.derivations/source_facts` 也是这个形状【live 核实 schemas.py】）。删除时按支持集做确定性存活判定，需要打断推断时删“命中集”（每条推导路径至少删一个源）而不是删全部源。
- 预期：FE↑↑、AR（尤其 multi-hop）↑↑；伤：候选过宽时 PR 风险，靠 C 层的精确裁决与 D 层手术控制。

**C. 三层级联清除（确定性为主、LLM 只做边角裁决）**
- C1 事实层：槽位精确匹配删除/替换。F6 用 (subject, predicate) 槽位 supersession：写入新值同时把旧值条目置 purged 并进入 tombstone（对比 Zep/Graphiti 的双时态“失效而非删除”设计【索引，二级来源】——注意 BEMULA 要求旧值**不可再被套出**，只标记失效但仍可检索等于没删，必须物理清除或读时硬过滤，见 E1）。
- C2 向量层：删目标事实向量 + 一切来源与之相交的块向量（改写句、摘要块）；删后做**近邻审计**：用旧值原文、改写式、实体名分别查询向量库，断言无高于阈值的命中（自家离线检查，不依赖评测题）。
- C3 摘要层：**逐句手术**——只删支持集已死的句子，其余保留（必要时做最小衔接修复）。证据：PolyGraph 逐句 receipt 与再推导一致率 100%/87.6%，整段 flat receipt 仅 33.3%/36.4% 且错误几乎全是“谎称存活”；逐句清理能保住 35 个被触及摘要中的 33 个【已证实】。整段删摘要是 PR 杀手，整段留是 AR 杀手，句级是唯一解。
- C4 裁决分工：候选与目标的“是否真支持被删事实”判定，确定性规则（来源相交、槽位相等、时间窗包含、类别相等）能定的不定 LLM；LLM 只判语义蕴含类边角（如改写句是否仍蕴含旧值）。证据：MemCut 中 LLM 判断相对检索排序只多删 0.7 条证据/fact，且其价值随候选 reach 放大（池 20 时值 5–15 点、池 60 时 20–32 点）——**先保 reach，再谈判断**【已证实】；PACMI 的已发表做法正是“LLM 关系分类 + 确定性约束归一化”【已证实，arXiv:2610.05732】。
- 预期：FE↑、AR↑；伤：级联越深 PR 越险，靠 B3 的替代支持束（还有别的支持束存活就只删死句不删整条）与删除预算护栏。

**D. 专门类型处理**
- D1 **F3 语义类**：写时给每条事实打类别标签（BEMULA 事实有 12 类的分类体系【schemas 已证实】，但系统必须从文本自 infer 类别、不得读标注）；删除=类别索引全量扫除+来源图校验（跨类摘要句按 C3 手术）。类别标签写时一次打好，删时零检索——这是 F3 唯一能覆盖全的办法【建议】。
- D2 **F5 时间类**：双时间索引——`dialogue_time`（turn 的 session 时间戳）与 `event_time`（事实所述事件时间）分开建索引（schemas 中 F5 的 TimeScope 明确区分这两种模式【live 核实】）；请求解析先判模式再按窗扫除；含相对时间表达（“那年六月前后”）的解析交给 LLM、落窗后仍走确定性扫除。
- D3 **F4 推理类**：两条合规路线（论文附录 A 已证实二者都不罚、且源事实被排除在 preservation 之外）：(i) 删命中集源事实；(ii) 保源 + **推断抑制**：登记被禁结论，读时若问题在重建该结论（多跳路径命中被禁结论的 support 集）则拒答该结论。推荐 (ii) 为主、(i) 为辅：(ii) 不动源事实、对其他问题的 PR 零损伤，但实现难点在读时识别“问题在问派生结论”；注意 control 版必须仍能推出结论，所以抑制只能由删除请求触发、不能全局禁用推理。
- D4 **F6 替换**：见 C1；另加**旧值多路审计**（C2 的近邻审计对旧值单独跑一轮 + 摘要全文旧值字符串/同义搜索），因为 F6 的失分几乎全在旧值残留导致的 AR【已证实】。

**E. 读时防线（最后一道，且必须作用域化）**
- E1 **检索后 tombstone 过滤**：检索结果与 tombstone（已删的〔实体, 属性, 旧值〕三元组 + 来源 id 集）相交即剔除，剔除发生在拼 prompt 之前。证据：CUPMem 的 constrained readout（active 才可作答题依据、stale 只作历史、未决槽位阻断旧前提）使其在同 backbone 上 8.7%→68.0%【已证实，STALE】；PACMI 去掉 premise checking 后 final accuracy 0.99→0.83【已证实】。
- E2 **回答前一致性检查**：草稿答案过一道确定性扫描（旧值字符串/别名/数值变体）+ 必要时小模型语义扫描（答案是否蕴含被删事实），命中则改写为作用域拒答。专治 Boolean 攻击（是/否本身即泄漏）：对被删槽位的真假猜测统一 neither-confirm-nor-deny，而**其他槽位照常作答**——这是与 Full Context 全局拒答（PR 46.0 的教训【已证实】）的分界线。
- E3 **前提阻断**：问题预设了已删状态时（如 STALE 的 PR 维度、BEMULA 的 role-play/jailbreak 帧），先纠正前提再作答或拒答该前提部分。证据：STALE 已证实 PR 是最弱维度（最强模型 SR 92.0 而 PR 仅 30.0），且 CUPMem 的前提处理把 PR 拉到 78.0/75.0【已证实】。
- 预期：AR↑↑（尤其 jailbreak/role-play/Boolean）、FE 小幅↑；伤：过滤过宽伤 PR/RLA——tombstone 必须（槽位,旧值）级，严禁用类别级读时拦截（会重演过度拒答）。

**F. 自验探针闭环（ProvPurge 的差异化贡献，须设计得比“走一遍自己的检索”更强）**
- F1′ **探针集构成**（全部由系统自己从“刚删掉的内容”生成，不用评测题面，见 §5.1）：direct 复述问、indirect 改写问、Boolean 真/假值各一、多跳问（经来源图邻居事实绕问）、**residual 探针**——绕过自身检索，把删后仍存活的全部来源相关对象直接喂给 reader 提问。最后一项是关键：MemCut 已证实只走自身检索的 native recovery 会漏掉大量残留（hidden-risk share 在 S2 后高达 39–92%），PolyGraph 的再推导审计同理警告“自验测的是 liveness 不是 groundedness”、且靠参数记忆再生会被误判【已证实】。
- F2′ **不过则扩大**：任一探针泄漏 → 沿来源/实体图扩大一跳候选 → 再清除 → 再验；上限 2 轮或固定删除预算，扩大规模全程记日志（扩大幅度本身就是可报告的诊断量）。证据基底：MemCut 的“扩池优于加预算”+ CUPMem 的拓扑触发传播搜索（候选=Direct∪Affected∪Global_k 有界集、裁决器做最终决定）【已证实】。
- F3′ **探针卫生**：探针在独立临时上下文中跑、问答不写回记忆（与官方 probe_writeback:false 同理）；探针的 reader 与正式答题用同一模型，避免“自验用强模型、自欺”或反之。
- 预期：AR↑↑、FE↑；代价：每条请求多若干次 LLM 调用（成本项，须在实验里报“每请求自验调用数/ token”）；风险：探针问法若与官方攻击帧雷同会被审稿人质疑对着评测调参——探针生成规则须预先固定并公开（见 §5.1）。

**G. 防过删与防重学阻断（护栏，不是提分项但决定下限）**
- G1 替代支持保留：派生条目/摘要句只要还有一条完整支持束存活就保留（只删死句），直接搬 PolyGraph 存活语义【已证实】。
- G2 删除预算与分级：来源图直连项=必删；扩展项=须过 C4 裁决；每请求设扩展删除上限，超限转自验驱动（F2′）而非一次性全删。
- G3 tombstone 值域化：只封（槽位, 旧值），同槽位新值、同类新事实照常写入（保 RLA、保 F6）。

#### 3.2 优先级表（预期收益 × 实现难度）

| # | 手段 | 主抬指标 | 主伤风险 | 难度 | 证据强度 | 优先级 |
|---|---|---|---|---|---|---|
| A1+A2 | 触发识别+类型/范围解析+落点校验 | FE、AR、RPR | 误触发伤 PR | 中 | 强（论文头号失败模式） | **P0** |
| B1 | 来源反查候选集（代替相似度候选） | FE、AR | 低（确定性） | 中（写路径改造） | 强（MemCut 覆盖率瓶颈 + PolyGraph） | **P0** |
| E1+E2 | 读时 tombstone 过滤 + 答案扫描 + 作用域拒答 | AR、FE | 过宽伤 PR/RLA | 低 | 强（CUPMem/PACMI） | **P0** |
| C1–C3 | 三层级联（槽位/向量+近邻审计/摘要句级手术） | FE、AR | 级联过深伤 PR | 中 | 强（PolyGraph 句级数据） | P0–P1（与 B1 同属本体） |
| F1′–F3′ | 自验闭环（含 residual 探针、不过扩大一跳） | AR、RPR | 成本；探针卫生 | 中 | 中强（MemCut hidden-risk + CUPMem 传播） | **P1** |
| D4 | F6 旧值多路审计 | AR(F6) | 低 | 低 | 强（F6 失分结构已证实） | P1 |
| D1/D2 | F3 类别索引 / F5 双时间索引 | FE(F3/F5) | 类别误标伤 PR | 低–中（写时打标） | 中（类型难度已证实；索引解法为建议） | P1 |
| D3 | F4 派生注册表 + 推断抑制 | FE/AR(F4) | 读时误判伤他题 | 中–高 | 中（F4 语义已证实；抑制实现为建议） | P2 |
| B2 | 实体/图扩散补候选（来源图外残留） | AR(multi-hop) | 扩散过界伤 PR | 中–高 | 强（MemCut 关系链 + CUPMem 传播） | P2 |
| C4/G | LLM 裁决分工、替代支持保留、删除预算 | PR（护栏） | — | 低–中 | 强（PACMI 混合裁决；PolyGraph 存活语义） | P1（随 C 一起做） |

#### 3.3 硕士周期内最值得先做的 3 个

1. **A+B+C 本体链**：触发/范围解析 → 来源反查候选 → 三层确定性级联。这是把现成系统 1.5% RPR 的两个最大死因（请求不到存储层、候选覆盖不足）一次性绕开的地基，且全部有已发表证据支撑；没有它，后面的自验只是给漏删打补丁。
2. **F 自验闭环（带 residual 探针版）**：ProvPurge 区别于“又一个删除管线”的核心贡献点，且直接打最低的 AR；实现量集中在探针生成+一跳扩大策略，一人一学期可完成并做出干净消融。
3. **E 读时作用域防线**：难度最低、见效最稳，同时压 jailbreak/role-play/Boolean 三类攻击并防止“删完仍被旧条目带偏”；与 F 形成写后验+读时拦的双保险。注意 E 只能作防线不能作主体——只靠读时藏匿在 residual 式审计下等于没删，论文叙事上必须写删为主、读为辅（消融 A7 专门证明这一点）。

### 4. 实验设计规范

#### 4.1 主实验表格（建议布局）

- **主表（BEMULA 全量）**：行=系统，列=两 backbone 分组下的 RA/FE/PR/RLA/AR/RPR/Score。系统行建议：Full Context、Naive RAG、Mem0、LangMem（论文最强基线）、ProvPurge；资源紧时可砍 A-MEM/LightMem/SimpleMem 中的 1–2 个，但 LangMem 与 Mem0 不能砍（分别是“删过头”与“删不动”两个极端参照）。【已证实基线数值见论文 Table 1，可直接引用为参照，但自家复现值与引用值须分开标注】
- **第二协议表（LongMemEval-S 的 KU 子集 78 题）**：【已证实】KU 测的是“答当前值”的显式更新能力（原论文定义：识别用户信息变化并动态更新），**不测旧值是否被清除**——它与 BEMULA F6 互补（F6 答新值+查旧值残留，KU 只答新值）。报告时必须注明：数据版本（原版 vs 2025-09 cleaned 版，有二级来源指出官方版本已替换，报分须带版本/hash）、judge 模型（官方口径为按题型 prompt 的 GPT-4o，与人工一致率 >97%【索引】）、读法是 oracle haystack 还是完整 S 版 haystack——三种口径的分数不可混比。另有已发表证据提示 KU 的真瓶颈在记忆维护而非容量：全上下文 92% vs 有界自维护记忆 77%（gpt-5.4，配对 McNemar p<0.005），且对话变长 24× 时跌至 28%、等比扩容无恢复【已证实，Supersede, arXiv:2606.27472 索引摘要】。
- **第三协议表（STALE）**：按其官方口径报 SR/PR/IPA × Type I/II 六格 + Overall（judge 为 Gemini-3.1-flash-lite【已证实】）。定位要写清：STALE 测**隐式**冲突（无删除请求、新证据使旧记忆失效），与 BEMULA 的显式请求是不同能力；ProvPurge 在此的看点是写时裁决/传播是否泛化到无请求场景，不要把 STALE 分数说成“遗忘”分数。参照值：最好模型 55.2%、CUPMem 68.0%（GPT-4o-mini）【已证实】。
- **可选诊断协议（建议）**：MemCut 的 residual recovery 口径（把存活对象直喂 reader）作为自家离线诊断，不必跑全量官方流程——它正是检验“真删 vs 藏匿”的尺子，与自验探针的 residual 分支同源。
- 所有表统一：同 backbone 内比较；每格给点估计，Score 与四项主指标给 bootstrap 95% CI（见 §4.3）；RPR 给点估计并注明 n=135（1 条请求≈0.74 个百分点）。

#### 4.2 必须的消融（每条对应一个可证伪的主张）

| 消融 | 关掉什么 | 预期方向（假设） | 证明什么 |
|---|---|---|---|
| A1 无句级来源 | 来源粒度降为整段摘要级 | AR↓、PR↓（整段删或整段留） | 句级粒度主张（PolyGraph 已证实方向） |
| A2 无级联 | 只删事实层、向量/摘要不动 | AR(multi-hop/indirect)↓↓、FE 小降 | 级联主张 |
| A3 无向量层清除 | 事实+摘要删、向量留 | indirect/Boolean AR↓↓ | 向量残留主张 |
| A4 无自验 | 删除后不探针、不扩大 | AR↓、成本↓（须同时报成本） | 自验主张（含成本-收益） |
| A5 LLM 裁决 vs 确定性规则 | 同候选集、只换裁决器（再加混合版三方对比） | 差异应小（MemCut 预言判断增益有限） | “确定性优先”的设计主张；若 LLM 版明显更好，须诚实改叙事 |
| A6 只有写时删、无读时防线 | 关 E1/E2 | jailbreak/role-play AR↓ | 读时防线的增量 |
| A7 只有读时防线、不真删 | 只 tombstone 过滤 | 官方 FE 可能不差，但 residual 审计崩 | “真删≠藏匿”的诚实性证据（审稿人最爱问） |
| A8 无候选扩展 | 只用相似度 top-k 候选 | 复现 MemCut 瓶颈：FE/AR↓ | 来源反查相对检索候选的增益 |
| A9 自验不扩大 | 探针只报告、不触发再删 | AR 介于 A4 与完整版之间 | “扩大”这一步的独立贡献 |
| A10 tombstone 粒度 | （槽位,旧值）级 vs 类别级 | 类别级 PR/RLA↓ | 作用域化主张 |

消融规模【建议】：先在 Lite 上全跑筛出方向，全量只复核 A1/A2/A4/A5 四条核心消融 + 完整版（其余在附录以 Lite 结果呈现并注明）。

#### 4.3 统计检验（照 BEMULA 官方口径做，不要自创）

- **已证实（论文附录 K）**：对**实例**做 cluster bootstrap（10,000 次重抽 50 个 instance id、连同两版本的全部题目一起重算官方指标，保住 control 条件化）；配对比较用**同一组 instance draws** 算两系统差值的 CI。实例内题目共享 profile 与对话、并不独立，所以按题 bootstrap 是错的。
- **已证实**：官方不对 RPR 报 CI（点估计几乎全 0）；per-type/per-tier 单元太少（F4 仅 13 条请求、xlong 仅 3 个实例）时只作描述性解读、不作确认性结论——照此办理，自己的分项表也要克制措辞。
- 参照效应的量级感【已证实】：DeepSeek 上 LangMem 与 Full Context 的 Score 差 2.5、配对差 CI [-2.4, 7.6] → 判平局；LangMem 领先 SimpleMem 14.5、CI [9.5, 19.8] → 才算站稳。**小于约 5 个点的 Score 差距不要写“优于”，写“相当/并列”**（此阈值是按该例的经验建议，非官方标准）。
- 其他协议：KU 78 题与基线做逐题配对比较可用 exact McNemar（Supersede 与 PACMI 均用此法【已证实】）；多重比较（多 backbone×多系统）时预先指定主比较（建议：全量×主 backbone×Score 对 LangMem），其余标为探索性，或做 Holm 校正【建议】。
- 随机性：judge temperature 0 已固定；自家系统的采样温度、种子须写进方法并固定；条件允许时完整版在 Lite 上重复 2–3 次报 run 间方差，全量至少报 bootstrap CI 替代重复跑【建议】。

#### 4.4 审稿人认可的呈现方式

- **必备分项**（官方 `run_eval` 的 report.md 已自动产出 by_forget_type、by_attack_type、by_length_bucket 与 RA-gated 对照【live 核实 run_eval.py】，直接沿用其口径再排版）：①逐 F1–F6 的 FE 与 AR 表；②逐攻击类型（indirect/role-play/jailbreak/Boolean/multi-hop）AR 表；③逐长度档（short/medium/long/xlong）全指标表；④FE–PR 权衡散点（论文附录 G 画法）；⑤请求级失败归因：每条未通过请求死在哪一类题（仅攻击失败/仅 preservation 失败/多类失败的计数分布）——这一张图最能说明 RPR 的故事【建议】。
- 主表同时给 FA=√(FE·AR) 与 UG=√(PR·RLA) 两个因子，让读者看清涨分来自“删得掉”还是“没删坏”。
- 成本列：每系统报每实例平均 LLM 调用数/token（写、答、judge、自验分开），自验的额外成本必须入表，否则 A4 消融不公平。

### 5. 常见翻车点与预防

1. **偷看标注式污染（最致命）**。【已证实】`Turn` 带 `linked_fact_ids`，`DialogueInstance` 含 `forget_requests[].target_fact_ids / time_scope / replacement`，`begin_instance` 钩子能拿到整个 instance。预防：adapter 内自设白名单——只读 `speaker/text/timestamp`，把 `observe/begin_instance` 的入参先投影成自家 `CleanTurn` 再进系统；在论文“实现细节”里明文声明这一点并开源对应代码。审稿人一旦发现用了 linked_fact_ids，全部结果作废。
2. **对着攻击帧调参**。【已证实】官方攻击帧是固定模板、从不改写（“their wording is the attack vector”）。预防：自验探针与防御规则只许由“系统自己删掉的内容”生成，禁止把官方 questions/dialogues 中的题面、帧式写进任何 prompt 或规则；探针生成模板预先冻结并随代码公开。
3. **在测试集上调阈值（含 Lite 子集泄漏）**。【已证实】Lite 31 实例 ⊂ 全量 50 实例。预防：阈值/预算只在 quick_check 或自生成 dev 集上定（先自查 quick_check 与 full 的 instance_id 是否重叠——【未确认】本次未核实重叠情况，clone 后必须查）；若确在 Lite 上调过参，全量报告须同时单列 long+xlong 19 实例（未参与调参部分）的结果并如实说明。所有超参在跑全量前冻结、写入 run_meta 同级配置快照。
4. **judge 偏差与同源性**。【已证实】BEMULA 数据由 GPT-5.6-Luna 生成，官方 judge 为 DeepSeek-V4.1-Flash；judge 的 position/verbosity/self-enhancement 偏差是已发表共识（Zheng et al. 2023，多篇二级来源一致【索引】）。预防：主结果用官方 judge 保持可比；另用 `scripts/rejudge.py`（脚本存在【live 核实 scripts 列表】）换一个不同家族 judge 对主结果重判一遍作敏感性分析；再按官方做法抽 **≥100 题、正负判定平衡、盲评**做人工核查并报一致率与 Cohen's κ（官方基准：100 题 98%、κ=0.96【已证实】，自家核查低于此须先修 judge 口径再谈分数）；可直接与仓库 `data/human_eval_annotations.csv` 对照。
5. **合成数据外推过度**。【已证实】BEMULA 作者自述局限：单一生成管线的合成数据、仅对话记忆（不含工具 cache/log）；STALE 同为构造场景。预防：结论措辞限定为“在 BEMULA/STALE 等构造基准上”；有余力时加一节小规模真实对话定性案例（不报与基准混排的分数）；不要把 KU 子集（测更新）与 STALE（测隐式冲突）的分数包装成“遗忘能力”的直接证据（见 §4.1 的定位说明）。
6. **只报总分**。几何平均会掩盖单项崩塌（如 PR 20 的系统总分仍有 12.4）。预防：总分永远与四项分项、FA/UG、RA、RPR 同表出现；分项表按 §4.4 齐全。
7. **把“藏匿”当“删除”**。只在 query 时过滤、不清存储，能过一部分官方题但过不了 residual 式审计，且与 ProvPurge 的立论（确定性级联清除）自相矛盾。预防：消融 A7 主动自曝此对照；自验含 residual 探针；论文中区分 store-level erasure 与 read-level suppression 两个词。
8. **探针/答案写回污染**。【已证实】官方 `probe_writeback:false`、query 不得写回。预防：自家自验探针同样隔离（临时上下文、不落库）；assistant 在数据集中的确认回复不得复述被删内容这一点也提醒：ProvPurge 在真实产品里的确认话术不要复述旧值（评测内不涉及，但写入讨论一节更完整【建议】）。
9. **静默失败与脏数据**。PolyGraph 记录过真实事故：供应商的限额提示被当作模型答案缓存、污染上千次调用【已证实】。预防：跑批加结构护栏——每实例 ingest 后断言存储非空、抽检答案非模板错误串；`--resume` 前校验 `items.jsonl` 尾行完整性（仓库 load_jsonl 对残缺行有容错【schemas 代码注释，已证实】，但不要依赖它掩盖问题）。
10. **版本漂移**。仓库仅 1 个 commit、基线靠外部 clone 且未锁版本、judge/backbone 的 API 版本会变。预防：记录本仓库 commit（862aff6）、每个基线 clone 的 commit、`.env` 中全部模型名与端点、configs 快照；引用论文基线数字时标注“引自论文 Table 1”与自家复现数字分开。
11. **F4 语义误读**。【已证实】F4 的源事实不在 preservation 探针内，删源或抑制推断都合规。预防：不要把 F4 删源算成“过删”写进错误分析；也不要反过来宣称“保源抑制”比删源更优——基准对此不表态，优劣论证须落在 PR 其他项与真实产品语义上。
12. **用弱 backbone 得出架构结论**。【已证实】Qwen3-32B-AWQ 上抽取阶段崩坏、全场 Score ≤9.5；STALE 亦显示冲突裁决能力随模型档位剧变（同任务 gpt-4o-mini 与强模型差距悬殊【索引，二级来源】）。预防：架构结论至少在两个能力档位的 backbone 上复现；把昂贵裁决步骤（触发解析、蕴含裁决）单独用强模型、其余用便宜模型的“分工配置”作为正式配置报告，而不是默认全程同模型。

---

## Could not verify

- **官方成本数字**：BEMULA 论文与 README 均未给出单次运行的美元/token/时长；§1.3 的外推公式是基于分档统计的【推导】，须由学生用 quick_check/Lite 实测单位成本填充。
- **quick_check 的规模与构成**（实例数、长度/类型分布）及其与 full 的 instance_id 重叠情况：本次仅 live 确认其目录与三件套文件存在，未读内容；调参集划分前必须本地核查。
- **仓库 `langmem.py` 是否已默认启用 removal 通道**：论文称 stock 默认关闭、作者启用；本次未打开该 adapter 文件核实其当前默认状态，复现 LangMem 前必查。
- **`bemula/baselines/serving.py` 的用途**（疑为本地模型 serving）：未打开，不影响主线接入（自研 adapter 不依赖它）。
- **LongMemEval 各题型题量（KU 78 等）与 judge 人工一致率 >97%**：来自原论文转引的二级来源与多篇复现的一致口径【索引】，未逐行打开原论文表格核对；KU=78 与任务背景一致且被 Supersede 等多篇独立工作复用，可信度高，但正式引用时建议再核原论文 Table。
- **STALE 中 CUPMem 状态 schema 的完整细节**（附录 F）：本次读到三组件与候选集公式，schema 全表未展开，不影响方法借鉴。

## Sources

（读取日期均为 2026-10-29；标注 live = 浏览器实页核实， 全文 = 打开论文 HTML 全文， 索引 = 搜索结果文本）

1. BEMULA 论文全文 — https://arxiv.org/html/2610.18052v1 【全文】（摘要页 https://arxiv.org/abs/2610.18052v1 ）：基准规模、六类请求、五套件、指标/RPR/Score 定义、主结果 Table 1、失败模式、附录 A 协议、C 基线删除通道、F Lite、H 攻击分项、J 长度效应、K bootstrap、judge 人工验证 98%/κ=0.96。
2. BEMULA 仓库 — https://github.com/Nboxff/bemula-benchmark 【live】及其实页文件：`bemula/baselines/base.py`（MemoryAdapter 契约）、`scripts/run_eval.py`（注册表/参数/checkpoint/报告分项）、`configs/default.yaml`（seed/judge_temperature/probe_writeback）、`.env.example`（模型与 embedding 变量）、`bemula/schemas.py`（Turn/Fact/ForgetRequest 结构与污染面）、`data/` 与 `data/lite`、`data/quick_check` 目录树（与 README 路径不一致、memoryos.py 缺失等坑均出自此）。
3. MemCut 论文全文 — https://arxiv.org/html/2610.21405v1 【全文】：456 secrets、native vs residual recovery、hidden-risk share、候选池/budget sweep（池 20→60 与 budget 10→20 的对照数值）、覆盖率瓶颈、关系链难达、closure 删除。
4. PolyGraph《Keep Your Receipts》全文 — https://arxiv.org/html/2610.17719 【全文】：receipt/支持集语义、集合算术清除 1–21 ms、flat 摘要 receipt 33.3%/36.4% vs 逐句 100%/87.6%、分层策略污染 0.4%/0.8% 与 recall 99.8–100%、receipt 成本、审计测 liveness 非 groundedness 的局限、缓存污染事故。
5. STALE 论文全文 — https://arxiv.org/html/2605.06527v1 【全文】（摘要 https://arxiv.org/abs/2605.06527 ）：400 场景/1,200 题、SR/PR/IPA×Type I/II、最好模型 55.2%、框架分数、LightMem 诊断 Table 3（77.5% 检索到仍失败、仅 3.3% 旧条目被判更新）、CUPMem 三组件与 8.7%→68.0%、judge=Gemini-3.1-flash-lite。
6. PACMI 论文 — https://arxiv.org/abs/2610.05732v1 【索引：摘要+HTML 片段】：来源图+四态有效性格+BFS 级联、premise checker、消融（去 premise checking 0.99→0.83；去级联错误 3→11 但 p=0.0574 未显著）。
7. LongMemEval 论文 — https://arxiv.org/html/2410.10813v2 【全文开头与设计章节】（仓库 https://github.com/xiaowu0162/LongMemEval ）：500 题、S 版约 115K tokens/题、KU 定义、fact-augmented key expansion（recall@k +9.4%、QA +5.4%）、time-aware query expansion（temporal recall +6.8~11.3%）、Chain-of-Note（最高 +10 点）。
8. Supersede 论文摘要 — https://arxiv.org/pdf/2606.27472 【索引】：LongMemEval KU 78 题全上下文 92% vs 有界记忆 77%（McNemar p<0.005）、长度放大后 68%→28% 且扩容无恢复。
9. LLM judge 偏差（Zheng et al. 2023 的 position/verbosity/self-enhancement 三偏差及缓解法）— 多篇二级来源一致，如 https://github.com/juanlara18/portfolio/blob/HEAD/front/public/blog/posts/research/llm-as-a-judge.md 【索引】。
10. Zep/Graphiti 双时态失效设计 — 二级综述 https://www.memoryplugin.com/wiki/temporal-memory.html 【索引】（原论文 arXiv:2501.13956 仅作背景，未展开核实）。
11. STALE/CUPMem 与框架分数的第三方转述（交叉核对用）— https://amem.owo.lc/guide/design-rationale.html 【索引】。
