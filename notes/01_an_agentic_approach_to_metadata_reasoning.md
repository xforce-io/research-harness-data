# 01 — An Agentic Approach to Metadata Reasoning

- **arXiv**: 2604.20144
- **Authors**: Jiani Zhang, Sercan Ö. Arık, Cosmin Arad, Fatma Özcan, Alon Halevy (Google)
- **Axes**: data_kind = structured (+ semi-structured, view); layer = retrieval_selection (primary), catalog_metadata (secondary)

## Claims

- Metadata Reasoner 在 KramaBench 数据选择上达到 83.16% 的平均 set-aware F1,而 vector-search 基线为 50.77%、Pneuma 为 45.12% —— 比 SOTA 高约 32 个百分点 [1: §5.1.1, Table 4]。
- 增益在高基数搜索空间中最大:在 Astronomy 域(约 1,498 张表),Full MR 达到 72.31% F1,是 vector search(32.80%)与 Pneuma(27.70%)的两倍多 [1: §5.1.1, Table 4]。
- 在合成放大的杂乱 BIRD data lake 上,MR 保持 85.5% 的平均 F1,而 Top-10 vector 基线崩溃到 30.0% [1: §5.1.2, Table 5]。
- MR 在 99.0% 的情况下选出严格无噪声的表;vector 基线让 35.6% 的噪声(subsets 13.5%、test splits 10.7% 等)进入其 Top-10 [1: §5.3, Fig. 4]。
- 高精度选择会向下游传导:把 MR 选出的表喂给 Gemini-3-Pro 的 Text-to-SQL 生成器,使平均执行准确率从 Top-10 vector 检索的 56.38% 提升到 71.28% [1: §5.2, Table 6]。
- 附加(经统计合成的)metadata 是单一最大贡献项:加入它(MR-Search → MR-Search+Attached)使 F1 从 73.43% → 79.66%,*并且* 将平均步数从 13.51 → 8.36 [1: §5.4, Table 3]。
- 即时工具 metadata 给出最高准确率,但若没有附加 metadata 的"热启动",步数会翻三倍到 24.14,因为 agent 会做穷尽式试错 [1: §5.4, Table 3]。
- 面向区分的 embeddings 在高基数 / 高冗余域的 Recall@5 上优于仅 schema 的 embeddings(Astronomy Rec@1 27.98% vs. 14.85%;Legal Rec@5 64.23% vs. 47.06%)[1: §5.5, Table 7]。
- Full MR 高效:它在每个任务平均仅 10.10 个推理步内达到其峰值 F1 [1: §5.1.1, Table 3]。

## Assumptions

- 一个 **全面的 metadata catalog 已经存在** 且被丰富填充 —— 不只是 schema 与文本,还有 lineage、数据质量指标、使用模式,以及术语表/ontology 这类 catalog 级语义 [1: §3.1]。论文承认大多数企业缺少这样的高质量 metadata [1: §6],因此这是一个使能前提、而非既定条件。
- 数据资源被当作 **表**(结构化 / 半结构化)处理;非结构化数据被显式排除在当前范围之外,留给未来工作 [1: §3.1, §6]。
- 那些专用工具 —— `data_finder()`、`joinability_check()`、`column_profiler()` —— 是可用的、确定性的、作为证据源可信赖的 [1: §3.5.2]。
- 附加 vs. 即时 metadata 之间的最优划分取决于部署(可用性、片段大小、延迟、agent 编排能力),被当作一个可调的设计选择、而非已解决的问题 [1: §3.5]。
- 对 BIRD 评测,注入的 lineage 标签准确地把每张派生表映射到其干净基表祖先,而基于后缀的质量标记(`_stg`、`_subset`、`_dups`、`_broken_fk`、`_nulls`)忠实地编码了 ground-truth 质量 [1: §4.2, §4.4.2]。

## Method

- **输入**:一个数据任务 `Q`(可由 SQL/Python 回答)+ 一个 metadata catalog `T`。**输出**:一个表子集 `T* ⊆ T`,它既 *充分*(覆盖所有必需的实体/属性且可正确 join)又 *最小*(没有更小的子集足够),外加一段自然语言理由 `J`,为下游 agent 锚定这些表为何合适 [1: §3.1]。
- **核心设计原则**:不同 metadata 类型在不同流水线阶段最优;一次性供给全部 metadata 会导致上下文饱和并劣化推理。一个编排 agent 自主地在每一步只获取所需片段 [1: §3.2, §3.5]。
- **三个操作阶段**:(1)*retrieval* —— 表 metadata 被编码为 embedding 向量供搜索;(2)*evaluation* —— 在显式附加到检索候选上的 metadata 上推理;(3)*on-the-fly* —— 专用工具提供高度具体的查询时 metadata 信号 [1: §3.2]。
- **查询分解与规划**:把 `Q` 分解为搜索约束(命名实体、度量、时间范围、粒度),构建一个多维搜索计划,然后在 search ↔ 推理 ↔ 工具调用之间交替 [1: §3.3]。
- **Semantic search 工具**:在 embeddings 上做 ANN 搜索。为对抗 *语义同质化*(相似表 → 不可区分的 embeddings),采用两阶段的 **面向区分的 metadata 构造**:(1)对相关表分组,让 LLM 识别共享 vs. 独有变量并产出一个组感知的 prompt 模板;(2)对每张表套用该模板,产出一段强调区分性特征(如时间范围)的描述 [1: §3.4, §3.4.1]。
- **状态感知去重**:一个会话字典 `S` 跟踪已出现的表 ID 与出现次数;重复命中被抑制,并替换为一个复现指示("Table ID: xxx (Appeared N times)"),迫使注意力转向新表;一轮只含重复项的搜索返回一个终止信号,促使修订策略 [1: §3.4.2]。
- **附加 metadata**(在 `T` 上预计算):物理 schema + LLM 概括的统计画像(值域、top-K 基数、null 比率),以自然语言渲染以节省 token [1: §3.5.1]。
- **即时工具**:`column_profiler()`(精确的去重计数、分布、直方图)、`data_finder()`(确定性的值级存在性检查,查询时)、`joinability_check()`(对特定候选对的即时 FK 重叠 / 引用对齐检查)[1: §3.5.2]。

## Eval

- **数据集**:(1)**KramaBench** —— 真实世界的杂乱 data lake,6 个域,最多约 1,498 张表(Astronomy),由端到端流水线改编以隔离 *数据发现* 阶段;(2)**合成放大的 BIRD** —— 5 个数据库各有 clean + messy 版本,带水平分区、生命周期重复(`_prod/_stg/_test`)与低质量变体(broken FK、注入的 NULL/dups、subsets);758 个分析问题 [1: §4.1, §4.2]。
- **基线**:Vector Search(非 agentic;0.7 cosine 阈值 + semantic reranker;同时兼作 MR 自己的搜索工具),以及 Pneuma(混合全文 + vector + LLM judge 重排,跑在 Gemini-3-Flash 上)[1: §4.3]。消融:MR-Search、MR-Search+Attached、MR-Search+Tools、Full MR。所有 MR 变体通过 ADK 框架使用 Gemini-3-Flash;embeddings 用 text-embedding-005 [1: §4.3]。
- **指标**:set-aware **F1**(Recall = 充分性/覆盖度,Precision = 最小性/简洁度),外加用于成本的平均步数。排序基线按其 *跨所有 K 的最佳 F1* 计分(一种慷慨的 oracle-K 比较)[1: §4.4.1]。对 BIRD(没有直接的 ground-truth 集),正确性通过 lineage 验证:一张被选中的表只有在其基表祖先位于 gold SQL 中、不带噪声后缀、且(对分区而言)其分区值匹配 gold filter 时才算正确 [1: §4.4.2]。
- **下游**:用 Gemini-3-Pro 生成器测 Text-to-SQL 执行准确率,比较 Top-10 vector 表 vs. MR 选出的表(+ MR 理由)[1: §5.2, Table 6]。
- **Embedding 消融**:Recall@{1,5,10},比较仅 schema、表内容概括、与面向区分的 metadata [1: §5.5, Table 7]。
- **失败分析**:126 个 KramaBench 失败被归为 5 种模式 —— 缺失关系依赖、粒度不匹配、多阶段规划失败、冗余关系选择、不完整的分区检索 [1: §5.6]。

## Weaknesses

- **单一 backbone 混淆**:每个 MR 变体与 Pneuma 都跑在 Gemini-3-Flash 上,配一个 embedding 模型(text-embedding-005)[1: §4.3]。agentic 编排的增益与该 backbone 的能力不可分割;论文从未测试更弱的模型,因此无法表明是架构(而非模型)驱动了那 83% —— 然而摘要却把它框定为一个通用方法结果。
- **标题低估了对 catalog 丰富度的依赖**:KramaBench 结果预设 lineage/质量/术语表 metadata 存在,但论文自己指出大多数企业缺少高质量 catalog [1: §6]。83.16% 这个数字可能无法迁移到实践中常见的仅 schema 与文本的 catalog —— 一个摘要未浮现的范围限制。
- **近乎循环的噪声鲁棒性设置**:在合成 BIRD 上,"正确性"由那些作者 *注入、并随后暴露在每张表 metadata 描述中* 的后缀标记(`_stg`、`_subset` 等)定义 [1: §4.2, §4.4.2]。MR 的 99% 避噪可能部分反映的是读取那些注入的 lineage 标签,而非真正地就数据质量进行推理;这一混淆未被承认 [1: §5.3]。
- **除步数外没有延迟/成本核算**:`data_finder()` / `joinability_check()` 触及实际数据值与 join 路径,带有未被量化的真实计算与 I/O 成本;仅靠步数低估了即时工具的开销 [1: §3.5.2, §4.4.1]。
- **索引构建成本未报告**:面向区分的 embeddings 需要对表 *分组* 做两阶段 LLM 处理 [1: §3.4.1];论文没有给出在一个 1,500 表(或更大企业级)catalog 上构建该索引的成本/可扩展性数字,尽管它把该方法定位为对抗 context-window 扩展极限。

## Relations

这是 `data` 支柱的种子 note —— `notes/` 中没有先前的 note 可供关联。下方的 relations 把本文锚定到 working thesis 与研究问题,使后续 note 有一个显式的挂接点。

- builds-on (thesis RQ1) [high]:本文的核心机制 —— 一个以片段方式查询的带类型 metadata catalog,而非对整个语料做逐查询的 LLM 扫描 —— 是 thesis 主张("长任务可靠性受限于一个 *覆盖数据资源的带类型 metadata catalog*,而非原始语料扫描")的一个直接、可实现的实例 [1: §1, §3.2]。
- orthogonal (thesis RQ2: 统一结构化 + 非结构化) [med]:MR 在一个 catalog/搜索接口下统一了结构化 + 半结构化表,但显式排除了非结构化数据(留给未来工作)[1: §3.1, §6];因此它仅为 thesis 预期跨度中结构化/view 的那一半支持"单一 catalog + 访问接口"目标。
- builds-on (thesis RQ4: 符合 ontology 的 metadata) [med]:catalog 被声明可选地包含术语表/ontology 与"语义模型(semantic model)"作为 agent 可查询的 catalog 级 metadata [1: §3.1],与 thesis 到 ontology 支柱的接口对齐 —— 不过论文从未把 ontology conformance 作为硬性要求或测量其效应,因此该联系是设计上兼容、而非已演示。
- competes-with Pneuma [high]:Pneuma 是本文点名的 SOTA 混合检索比较对象,并被表明处于劣势(45.12% vs. 83.16% F1)[1: §4.3, §5.1.1];框定为"agentic 组合式选择 vs. 基于排序的个体检索",且 MR 复用一个 vector 引擎作为 *子例程*,而非彻底拒绝检索。
