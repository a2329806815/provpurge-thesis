# MemForm
- URLs: https://arxiv.org/abs/2610.14431v1 ; https://arxiv.org/html/2610.14431
- 读于: 2026-10-30, index: 摘要+§1–§5.3 (方法/构造/主结果/防御) — index, 非附录深读
- 身份: Pan, Shao, Ma, Jin, Zhai, Wang, Yang; "MemForm: Benchmarking Memory Formation Poisoning in LLM Agents"; submitted 10 Oct 2026; PDF 标注 Under review as a conference paper at ICLR 2027; code https://github.com/opanhw/memform-code
- 主题/方法: 记忆形成投毒 — 对抗操纵持久记忆被生产/提交的接口; 五接口 Content/Evidence/Scope/Selection/Path Poisoning (Selection=固定候选池+固定预算下换保留子集; Path=保留多重集与 updater 不变只换 consolidation 顺序); 235 语义族/1016 编译案例/1891 format instances (994 MC + 897 free-form); 六 LLM; LangChain 主实现 + LlamaIndex 复现
- 数字: macro-avg ASR MC 35.2–84.0%, free-form 49.0–87.1% (Table 1); Path 列 MC 37.1–95.1%; 防御 (DeepSeek, Table 2): 最好 A-MemGuard 平均 ASR 58.8→50.6 (−8.2), 主要作用于 Path (56.8→21.4), 其余防御 −0.0 至 −4.5; 摘要称 Path 易感性取决于 updater, 接口适用性取决于记忆架构
- 与删除的关系: 不做删除/遗忘方法, 无 provenance closure/删除事务; 是形成端 (write-path) 威胁基准
