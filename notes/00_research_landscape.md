# Research landscape

> `data` 支柱的持续更新 survey。分类法由 working thesis 的四项主张与 CHARTER 授权播种而来:一个覆盖 **structured / unstructured / vector / view** 数据、并符合共享 ontology 的带类型 metadata **catalog** + connection + view + freshness。每篇论文归入下方最具体的层级桶;每个条目声明 ≥1 个显式 relation。引用以 `[N]` 表示,按 note 编号。

## §1 Catalog & Typed Metadata

_覆盖数据资源的带类型 metadata catalog —— thesis 的核心机制(查询片段,而非原始语料扫描)。_

- **Metadata Reasoner (MR)** 把 catalog 当作一个分阶段、逐片段的证据源,而非一个扁平大块:物理 schema + LLM 概括的统计画像("附加(attached)" metadata)在 catalog 上预计算,而实时工具提供查询时信号 [1]。附加 metadata 是选择质量的单一最大贡献项 —— 加入它使 set-aware F1 从 73.43% → 79.66%,*并且* 将平均推理步数从 13.51 → 8.36,即:让选择同时更准且更省的,是一个预计算的带类型 metadata 层(而不仅是实时探测)[1]。该结果预设了一个被丰富填充的 catalog(lineage、数据质量指标、使用模式、术语表),而论文承认大多数企业并不具备 [1] —— 这是一个使能前提,而非既定条件。
- **Semantic Layer (Cube)** [2] 从业务语义角度佐证了"typed metadata 作为上下文、而非推断"的核心主张:在受控配对设计中,在上下文里加入被描述的 semantic layer(相较于仅原始 schema),使三个前沿模型的首发分析准确率提升 +17.2–23.2 pp,每个 McNemar p ≤ 0.0015 [2: §5.1]。这是 RQ1 机制的第二个、独立的实例 —— 此处 metadata 是非正式的手写自然语言散文,而非预计算的统计画像 [1]。
  - relation: **builds-on** thesis RQ1 [high] —— 供给被描述的 metadata,让模型去查阅而非推断;可与 [1] MR(选择)组合,因为本文锚定的是含义、而非选表 [2: §3.1]。

## §2 Retrieval & Selection

_为一个任务发现并选择正确的数据资源。_

### §2.1 Agentic / multi-step selection

- **Metadata Reasoner (MR)** [1] —— 一个编排 agent,它把数据任务分解为搜索约束,然后在 semantic search ↔ 推理 ↔ 工具调用之间交替,返回一个 *充分且最小* 的表子集加上一段供下游 agent 使用的自然语言理由。它在 KramaBench 上达到 83.16% 的平均 set-aware F1,比最佳排序基线高约 32 个百分点,仅用约 10.1 个推理步 [1]。增益在高基数搜索空间中最大(Astronomy 约 1,498 张表:72.31% vs. vector search 32.80%)[1]。两个值得注意的子机制:*面向区分(discrimination-oriented)* 的 metadata embeddings,用以对抗近重复表的语义同质化;以及 *状态感知去重(state-aware deduplication)*,抑制重复命中以迫使注意力转向新候选 [1]。
  - relation: **competes-with** §2.2 排序基线 [high] —— 把"agentic 组合式选择"与"基于排序的个体检索"对置,但复用一个 vector 引擎作为搜索 *子例程*,而非拒绝检索 [1]。
  - relation: **builds-on** thesis RQ1 [high] —— 片段化的 catalog 查询 vs. 逐查询的原始语料 LLM 扫描,是 thesis 核心主张的一个直接、可实现的实例 [1]。

### §2.2 Single-shot / ranking retrieval baselines

- **Vector Search**(0.7 cosine 阈值 + semantic reranker)与 **Pneuma**(混合全文 + vector + LLM-judge 重排)是被点名的非 agentic 比较对象;两者都被 MR 压制(F1 分别为 50.77% 与 45.12%,vs. 83.16%),且 vector 基线让 35.6% 的噪声进入其 Top-10,而 MR 保持 99% 无噪声 [1]。此处记录为 agentic 选择所竞争的基线类别,而非独立贡献。

## §3 Unified Access (structured · unstructured · view · vector)

_横跨异构源类别的单一 catalog + 访问接口(thesis RQ2)。_

- **Metadata Reasoner (MR)** 在一个搜索/选择接口下统一了结构化 + 半结构化表(并通过注入的 lineage 把派生/分区表当作 view 式资源处理),但 **显式排除了非结构化数据**,留给未来工作 [1]。因此它只覆盖了 thesis 预期跨度中结构化/view 的那一半。
  - relation: **orthogonal** 于 thesis RQ2 的非结构化分支 [med] —— 仅为结构化/view 源支持"单一 catalog + 访问接口"目标 [1]。

## §4 Semantic Layers & Views

_覆盖变动中物理源的稳定逻辑把手(thesis RQ3)。_

- **Semantic Layer (Cube)** [2] —— 首篇专门针对 RQ3 的论文。在 prompt 中加入一个约 4 KB 的手写 markdown semantic-layer 文档(度量公式、维度层级、数据约定、消歧规则),使三个前沿模型的首发分析准确率提升 +17.2–23.2 pp(例如 45.5%→68.7%),每个配对 McNemar p ≤ 0.0015 [2: §5.1]。该效应是 *结构性* 的:有该文档时三个模型在统计上不可区分,没有时也不可区分 —— 是文档的存在、而非模型档次,解释了几乎全部成对方差 [2: §5.2]。机制:semantic layer 将主导的 text-to-SQL 错误类别(schema-linking + 业务逻辑,占失败的 >80%)从开放式推断转化为受约束的查阅 [2: §3.1, §6.1]。**注意:** 只测量了 *上下文形式*(prompt 中的咨询性散文);真正能吸收物理变动的运行时/view 形式 —— RQ3 的"覆盖变化源的稳定把手"属性 —— 被断言为更优的下界,却留给了未来工作,且未运行任何陈旧化 / 错误文档条件 [2: §6.2, §6.5]。
  - relation: **orthogonal** 于 [1] MR [med] —— MR 通过分阶段画像 catalog 选择 *哪些表*;本文通过业务语义锚定 *所供给 schema 的含义*。相同的主导错误类别、不同的流水线阶段,可组合而非竞争;两者互不引用 [2: §3.1]。
  - relation: **builds-on** thesis RQ3 [med] —— 仅以上下文形式演示了 semantic-layer 收益;吸收变动的 view 形式仍未被演示 [2: §6.2]。
- MR 基于 lineage 将派生表映射到干净基表祖先 [1] 是一个相邻信号 —— 一个原始的"view 覆盖物理变动"把手 —— 但 MR 并未把 views 当作一等的 semantic layer。

## §5 Ontology Conformance & Freshness

_符合共享 ontology 的 catalog metadata(thesis RQ4);作为生产关切的 freshness / lifecycle。_

- **Metadata Reasoner (MR)** 声明 catalog 可选地包含术语表/ontology 与"语义模型(semantic model)"作为 agent 可查询的 catalog 级 metadata [1],其生命周期后缀标记(`_prod/_stg/_test`)编码了一个 freshness/质量维面 —— 但 ontology conformance 从不是硬性要求,也未被测量,因此该联系是设计上兼容、而非已演示 [1]。
  - relation: **builds-on** thesis RQ4 [med] —— 与 ontology 支柱接口设计上兼容;效应未测量 [1]。
- **Semantic Layer (Cube)** [2] 把 RQ4 从设计上兼容提升为已测量:§6.3 明确把 markdown semantic layer 的因果机制等同于形式化 ontology 方法 —— Sequeda 等人的 OWL ontology(16.7%→54.2%)与 Allemang & Sequeda 的基于 ontology 的校验(→72%)—— 且锚定效应跨 ontology 研究得到独立佐证 [2: §6.3, §2.6]。跨源一致,尽管此处的"ontology"是非正式自然语言散文、而非形式化 schema。关于 freshness:其前提是一个 *权威静态* 文档,没有陈旧化或 lifecycle 机制,也没有错误文档测试 [2: §6.5] —— 与本桶的 freshness 关切相邻,但被显式搁置。
  - relation: **builds-on** thesis RQ4 [high] —— 受控配对效应加上跨研究 ontology 佐证;效应现已被测量,而非仅设计上兼容 [2: §5.1, §6.3]。

## Relations index

| From | Kind | To | Conf |
|------|------|----|------|
| [1] MR | competes-with | Pneuma / Vector Search 基线 | high |
| [1] MR | builds-on | thesis RQ1(带类型 metadata catalog vs 原始扫描) | high |
| [1] MR | orthogonal | thesis RQ2(非结构化分支 —— 已排除) | med |
| [1] MR | builds-on | thesis RQ4(符合 ontology 的 metadata) | med |
| [2] Semantic Layer | orthogonal | [1] MR(不同的流水线阶段) | med |
| [2] Semantic Layer | builds-on | thesis RQ1(typed metadata 作为上下文) | high |
| [2] Semantic Layer | builds-on | thesis RQ3(semantic layers / views —— 上下文形式) | med |
| [2] Semantic Layer | builds-on | thesis RQ4(符合 ontology 的 metadata) | high |
| [2] Semantic Layer | contradicts | thesis RQ5(freshness / lifecycle) | low |
