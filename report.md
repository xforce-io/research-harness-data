# Data Pillar: Research Report

> **Version:** v2 (2 papers)
> **Last Updated:** 2026-06-03
> **Papers:** [01](notes/01_an_agentic_approach_to_metadata_reasoning.md), [02](notes/02_semantic_layers_for_reliable_llm_powered.md)
> **Thesis:** [.researcher/thesis.md](.researcher/thesis.md)

## 定位

`data` 支柱押注的核心是:长程 agent 的可靠性并非受限于推理的巧妙程度,而是受限于其 **data supply layer(数据供给层)** —— 即它能否在每一步发现、选择并访问到正确的数据。这一中心设计主张既尖锐又可证伪:它要求一个 **覆盖数据资源的、带类型的 metadata catalog**,以片段方式被查询,而非对原始语料做逐查询的 LLM 扫描。目前已读的两篇论文从同一条流水线的两端攻击这一主张:[1] 构建了一个 agent,其全部职责就是通过在 catalog 上做 metadata 推理来 *选择哪些数据*;[2] 则做了一个受控实验,围绕通过 semantic layer 来 *锚定所选数据的含义*。两者从不互相引用,却落到了同一机制上 —— 供给经过描述/带类型的 metadata,让模型去查阅而非推断 —— 并且都报告了由此带来的显著效应。本报告 v2 关注的是:这种收敛在多大程度上强化了核心主张(RQ1)、[2] 在何处新开了 thesis 此前毫无证据的 semantic-layer 这条线(RQ3),以及 thesis 的哪些部分(非结构化统一、长任务中的 freshness)仍未被触及或新近陷入张力。

## Goal A —— 带类型 metadata 的 catalog vs. 原始语料扫描

**当前文献的结论。** 两个独立的数据点,得出同一结论。Metadata Reasoner(MR)[1] 是一个编排 agent,它通过分阶段查询 catalog metadata(embedding 搜索 → 附加的统计画像 → 即时工具)来选出一个 *充分且最小* 的表子集,从不整体扫描原始行;它在 KramaBench 上达到 83.16% 的 set-aware F1,而 vector search 为 50.77%、Pneuma 为 45.12%,仅用约 10.1 个推理步 [1]。其中与 thesis 最相关的消融实验:**预计算的"附加(attached)" metadata 层**(schema + LLM 概括的画像)是单一最大贡献项,将 F1 从 73.43% 提升到 79.66%,*同时* 将步数从 13.51 削减到 8.36 [1] —— 一个带类型的 metadata 层让选择同时更准且更省。[2] 在不同阶段、以更干净的设计检验了同一"metadata 作为上下文、而非推断"机制:在 prompt 中加入单个约 4 KB 的手写 semantic-layer 文档,使三个前沿模型的首发分析准确率提升 +17.2–23.2 pp(例如 45.5%→68.7%),每一对配对 McNemar 检验 p ≤ 0.0015 [2: §5.1]。关键在于,[2] 的效应是 *结构性* 的:有该文档时三个模型在统计上不可区分,没有时也不可区分 —— 是文档的存在、而非模型档次,解释了几乎全部成对方差 [2: §5.2]。这直接回应了 MR 最严重的混淆因素(见下):[2] 中的锚定增益 *并非* backbone 能力的产物。

**残余缺口 / 张力。**(i)MR 的单一 backbone 混淆在 [1] 中依然存在(所有变体 + Pneuma 都跑在 Gemini-3-Flash 上;没有更弱模型的测试)—— 但 [2] 的模型不变性结果是有力的间接证据,表明 *带类型 metadata 作为上下文* 这一杠杆是真实的,独立于 backbone 强度 [2: §5.2]。(ii)两篇论文都预设了 *丰富* 的 metadata,而这可能并不存在:MR 承认大多数企业缺少它所假设的 lineage/质量/术语表 catalog [1];[2] 的文档是由一位见过该数据集的分析师手写的 [2: §4.2]。机制得到了双重验证;但其 *现实前提*(丰富的 metadata 从何而来、由谁维护)两者都未验证。(iii)在 *访问方式* 上存在一处真切的跨论文张力:[2] 用一句无数据的话披露,一个具备等价语义知识的内部 agentic 工具调用系统 **并未** 击败全上下文的纯 schema 基线 [2: §6.5] —— 这与 MR 的 agentic 检索前提相悖。两者不能直接比较(选择 vs. 生成;F1 vs. pass-rate),但它们共同框定了一个开放的设计问题:分阶段 / agentic 的 catalog 检索何时优于直接把 metadata 内联进 prompt?

**我们应当怎么做。** 把带类型的 metadata catalog 当作主干,但把 catalog 的 *丰富度* 当作一等问题、而非假设:我们的访问层必须在仅有 schema + 文本的 catalog 上优雅降级,并最好能自行 *自举(bootstrap)* 出更丰富的 metadata(MR 的"附加画像是预计算且廉价的"[1] 支持把画像作为常驻的 catalog 构建步骤)。在访问方式上,结构化地化解 [1]/[2] 的张力:当 catalog 较大时 agentic 地选出充分且最小的集合,再在该集合能放进 context window 时内联它 + 一个 semantic layer 用于锚定 —— 把"相关 metadata 是否小到足以内联?"做成一个显式的路由决策,而非永远检索或永远内联。

## Goal B —— 跨 结构化 / 非结构化 / view / vector 的统一 catalog

**当前文献的结论。** 仍是部分覆盖,如今更显不足。MR 把结构化 + 半结构化表统一在一个搜索/选择接口下,并通过注入的 lineage 把派生/分区表当作 view 式资源处理,但 **显式排除了非结构化数据** [1]。[2] 加入了一个 semantic-layer 锚定机制,但同样 **仅限结构化**(25 张表的 ClickHouse 零售数仓,text-to-SQL)[2: §4.1]。两篇论文,对非结构化/vector 这条线零覆盖。

**残余缺口 / 张力。** 统一主张中的非结构化分支(RQ2)在读完两篇论文后 *仍无任何* 证据 —— 这是该支柱中最大的一段空白。我们如今有两项论证表明结构化(+ view)源能从共享 metadata 接口中获益,却仍无证据表明同一 catalog 抽象能扩展到非结构化/vector 源而不需定制粘合。

**我们应当怎么做。** 把非结构化统一主张视为未被证明,并主动优先攻克它。后续阅读应锁定那些把非结构化或 vector 源纳入 *同一* catalog + 选择/锚定接口的工作,而非另起一条并行的 RAG 流水线 —— 那正是 thesis 最暴露、而迄今两篇论文都缄默之处。

## Goal C —— semantic layer / view 作为变动吸收器

**当前文献的结论。** 这个目标从"几乎没有"翻转为"首个直接、受控的证据 —— 但只覆盖主张的一半"。[2] 是第一篇瞄准 RQ3 的论文:一个手写的 semantic layer(度量公式、维度层级、数据约定、消歧规则)以上下文形式供给后,将 text-to-SQL 的主导失败类别(schema-linking + 业务逻辑,占错误的 >80%)从开放式推断转化为受约束的查阅 [2: §3.1, §6.1],带来 +17–23 pp 的增益 [2: §5.1]。增益恰好集中在物理源会误导 agent 之处:snapshot-vs-flow 语义(原始条件对每日库存快照求和,数值大了约 1000 倍)、哨兵键、字符串型布尔值、以及针对一个截至 2009-12-31 的数据集做时间锚定 [2: §5.4]。MR 基于 lineage 将派生表映射到干净基表祖先 [1] 仍是一个较弱的相邻信号 —— 一个原始的"view 覆盖物理变动"把手,但还算不上一等的 semantic layer。

**残余缺口 / 张力。** [2] 验证了 semantic layer 的 *锚定* 收益,但仅限其 **上下文形式**(prompt 中的咨询性自然语言散文)。thesis 真正主张的属性 —— *在 view 吸收物理变动的同时提供一个稳定的逻辑把手* —— 需要 **运行时 / view 形式**(由 compiler 强制、确定性),[2] 断言这是更优且有下界保证的替代方案,却把它完全留给了未来工作 [2: §6.2]。也没有任何陈旧化或错误文档的测试,因此"当物理源变化时该把手是否保持稳定?"—— RQ3 的核心 —— 未被演示 [2: §6.5]。还有一处混淆:[2] 的基准与文档由同一团队(Cube)共同编写,因此增益可能部分衡量的是"文档把该基准刻意设入的那些坑约定的答案直接交了出来",而非通用的迁移 [2: §4.1, §5.4, §6.5]。

**我们应当怎么做。** 把"*某种* semantic layer 有助于锚定"这一结果收入囊中,但保持变动吸收主张为开放:近期的探针现在比 v1 更锐利 —— 当物理 schema 在其底下发生变化时,一个 *运行时 / view* 的 semantic layer 能否保持准确率,相较于一个必须重新编写的上下文形式文档?把 semantic layer 构建为一个被强制执行的运行时产物(使其能吸收变动),而非一个 prompt 字符串([2] 只在静态快照上证明了后者),并在刻意制造的物理源漂移下测试它。

## Goal D —— ontology conformance 作为一致性契约

**当前文献的结论。** 从"设计上兼容、未测量"升级为"经代理指标测量、并被跨源佐证"。MR 仍只 *可选地* 允许把术语表/ontology 作为 catalog 级 metadata,从未隔离其效应 [1]。但 [2] §6.3 明确把 markdown semantic layer 的因果机制等同于形式化 ontology 方法 —— Sequeda 等人的 OWL ontology(16.7%→54.2%)以及 Allemang & Sequeda 的基于 ontology 的校验(→72%)—— 且锚定效应在多个 ontology 研究中得到独立佐证(Sequeda;Luo 等人的临床 QA,借助 ontology-grounded GraphRAG 从 37%→98%)[2: §2.6, §6.3]。跨源一致提升了"*被描述的语义* 驱动了增益"的可信度。

**残余缺口 / 张力。** 这些佐证针对的是"上下文中的结构化语义"这一 *整族* —— 但 [2] 自身的产物是非正式的自然语言散文,而非形式化 schema,它并未将 *对共享 ontology 的 conformance(一致符合)*(thesis 特有的 RQ4 机制,即到 `ontology` 支柱的接口)从"任何被描述的语义都有帮助"中隔离出来。因此我们有了更强的证据表明语义即上下文是有效的,却仍无受控测试表明 *ontology conformance 本身* 相较于一个临时但被描述的层,能换来多步语义一致性。

**我们应当怎么做。** 把 ontology conformance 保留为一个待测量的假设,但锐化 A/B:不只是无 schema vs. ontology-grounded,而是 *临时被描述的语义*(Cube 风格的 markdown 文档)vs. *符合 ontology 的被描述语义*(同样内容,但针对共享 schema 做了类型化),衡量多步任务中的一致性漂移。[2] 表明被描述的语义有帮助;它并未表明承担作用的是 *conformance*。

## Goal E —— 长任务中保持供给正确(freshness / lifecycle)

**当前文献的结论。** 仍然单薄,如今还略有张力。MR 唯一与 freshness 相邻的机制是生命周期/质量后缀标记(`_prod/_stg/_test`、`_broken_fk`、`_nulls`),用于定义"无噪声"选择,但这些标记是作者注入并暴露在 metadata 中的,因此 99% 的避噪数字部分反映的是读取注入标签 [1]。[2] 让这一缺口更显性、而非弥合它:它的整个前提是一个 *权威静态* 的手写文档,**没有** 测试任何陈旧化、部分文档或错误文档的条件 —— 尽管它引用了 BIRD 审计发现 7–10% 的 evidence 标注有误 [2: §2.6, §6.5]。

**残余缺口 / 张力。** "在长任务中保持供给正确"(Design Context 目标 c)是 thesis 中测试最少的主张,如今还与 [2] 的框架 *直接张力*:[2] 演示的是来自一个冻结、正确产物的一次性锚定效应,而 thesis 主张数据层的职责是 *在源发生变动时* 保持供给正确 —— 一个一次性写入的文档并不具备这一属性。(本轮记录为一处 vs-thesis 矛盾。)两篇论文都没有运行任何长程任务,让正确数据、或 semantic layer 本身在步骤之间变得陈旧。

**我们应当怎么做。** 不要把任意一篇论文的静态快照结果当作 freshness 证据。把 freshness/lifecycle 做成一个 *被测量* 的 catalog 维面,并把 semantic layer 自身视为必须保持新鲜且 conformant 的 metadata —— 而非一次性写入的文档。我们自己的评测必须运行多步任务,让正确数据(及描述它的层)在步骤之间发生变化 —— 这正是尚无论文施压过的状态。

## 可证伪点追踪

- **可证伪点 1 — catalog is necessary** ("Falsified if production agent systems
  achieve stable long-task data grounding with **no** catalog/metadata layer at
  all")。当前证据:*支持,已强化*。MR 表明 catalog 驱动的选择优于无 catalog 的单发检索(83.16% vs. ≤50.77% F1)[1];[2] 表明上下文中被描述的 metadata 优于仅原始 schema 的锚定(+17–23 pp,配对 p ≤ 0.0015)[2: §5.1] —— 并且重要的是,[2] 的模型不变性结果通过表明该杠杆是结构性、而非能力驱动的,反驳了削弱 [1] 的单一 backbone 混淆 [2: §5.2]。未决保留:[2] 的 §6.5 报告(无数据)称一个 agentic 工具调用变体 *并未* 击败全上下文 —— 因此"catalog is necessary"成立,但"agentic catalog *retrieval* 是正确的访问模式"则有争议。能解决它的下一个观察:在固定准确率、变动 catalog 规模的条件下,做 agentic 检索 vs. 内联 metadata 的正面对决。

- **可证伪点 2 — unified catalog ≥ bespoke per-source** ("Falsified if unified
  catalogs consistently underperform bespoke per-source integrations on grounding
  quality")。当前证据:*不足,未变*。[1] 和 [2] 都仅限结构化 [1; 2: §4.1];没有与定制化逐源粘合的正面对决,且非结构化/vector 这条线两篇论文均未触及。下一个观察:一项跨异构源类别,将单一 catalog 接口与调优过的逐源流水线进行比较的研究。

- **可证伪点 3 — views beat raw sources** ("Falsified if agents do better against raw
  sources than against a curated semantic layer")。当前证据:*上下文形式支持;view 形式开放*。[2] 是首个直接测试:一个精选的 semantic layer 比原始 schema 高 +17–23 pp [2: §5.1],效应集中在恰恰是物理源陷阱的地方(snapshot-vs-flow、时间锚定、哨兵键)[2: §5.4]。但只测量了咨询性的 *上下文形式*;能吸收物理变动的 *运行时 / view* 形式未被测试 [2: §6.2],也没有陈旧化条件 [2: §6.5]。因此"views beat raw sources"对静态精选语义成立;而"views *作为变动吸收器*"仍未被证明。下一个观察:在 *刻意制造的物理 schema 漂移下*,同一 agent 在运行时 view vs. 原始表上的准确率。

- **可证伪点 4 — schema/ontology conformance needed for multi-step consistency**
  ("Falsified if schema-free catalogs suffice for multi-step semantic consistency")。当前证据:*部分支持,但靶向偏差*。[2] §6.3 将其效应与形式化 ontology 结果挂钩(Sequeda 16.7%→54.2%;Allemang & Sequeda →72%),并跨 ontology 研究佐证 [2: §2.6, §6.3] —— 是 *被描述的语义* 有帮助的有力证据。但 [2] 自身的产物是非正式散文,它从未将 *ontology conformance* 从"任何被描述的语义"中隔离出来,也未测量多步一致性。下一个观察:临时被描述 vs. 符合 ontology 的被描述语义的 A/B,衡量多步任务中的语义漂移。

## 版本更新日志
| 版本 | 日期 | 新增论文 | 关键变化 |
|------|------|---------|---------|
| v1 | 2026-06-02 | [01] An Agentic Approach to Metadata Reasoning | 报告自举。来自 MR 的 83.16% vs ≤50.77% F1 以及 attached-metadata 消融,为可证伪点 1(catalog 必要性)提供强支持;可证伪点 2/3/4 开启时几乎无证据(非结构化分支、semantic layer、ontology conformance 均未测试)。标记了单一 backbone 与 catalog 丰富度的混淆因素。 |
| v2 | 2026-06-03 | [02] Semantic Layers for Reliable LLM-Powered Data Analytics | 可证伪点 3 从近乎零翻转为 *支持(上下文形式)* —— 首个直接的 semantic-layer 测试(+17–23 pp,模型不变)。可证伪点 1 强化:[2] 的模型不变性反驳了 MR 的单一 backbone 混淆。可证伪点 4 经跨研究 ontology 佐证升级为部分支持,但 conformance 仍未被隔离。记录了新的跨论文张力(agentic 检索 vs. 全上下文,§6.5)以及一处 vs-thesis 的 freshness 矛盾(静态权威文档,无陈旧化测试)。在两篇仅限结构化的论文之后,非结构化分支(可证伪点 2)仍零证据。 |
