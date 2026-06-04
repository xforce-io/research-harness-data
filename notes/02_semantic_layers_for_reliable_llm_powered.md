# 02 — Semantic Layers for Reliable LLM-Powered Data Analytics

- **arXiv**: 2604.25149
- **Authors**: Michael Rumiantsau, Ivan Fokeev (Cube — team@cube.dev)
- **Axes**: data_kind = structured; layer = view_semantic_layer (primary), catalog_metadata (secondary)

## Claims

- 在 prompt 中加入一个约 4 KB 的手写 markdown semantic-layer 文档,使三个模型的首发分析准确率提升 +17.2 到 +23.2 pp —— Opus 4.7 50.5%→67.7%、Sonnet 4.6 46.5%→68.7%、GPT-5.4 45.5%→68.7% —— 每个配对 exact McNemar p ≤ 0.0015 [2: §5.1, Table 1]。
- 有该文档时,三个模型在统计上不可区分(67.7–68.7%);没有时也不可区分(45.5–50.5%)[2: §5.1]。
- 所有九个跨簇成对比较在 p < 0.01 上显著;semantic-layer 簇内的三个簇内比较均不显著(p ≥ 0.79),raw 簇内也均不显著(p ≥ 0.42)[2: §5.2, Table 2]。
- semantic-layer 文档的存在与否解释了该基准上几乎全部显著的成对方差;同档次内的模型选择则不然 [2: §5.2]。
- 准确率失败与自信的幻觉是同一个底层失败的两种视图:模型被迫推断 schema 未编码的业务语义 [2: §1, §2.1]。
- schema-linking 与业务逻辑错误占 LLM 生成 SQL 执行失败的 80% 以上 [2: §1, §6.1(引用 Shen et al. 2025)]。
- semantic-layer 效应是结构性的 —— 是任务被框定方式的属性,而非模型能力的属性 —— 因此带文档的较小模型持续优于不带文档的较大模型 [2: §6.1]。
- semantic layer 把主导的 text-to-SQL 错误类别从开放式推断转化为受约束的查阅 [2: §3.1, §7]。
- 增益(semantic 通过、raw 失败)集中在五类:多事实表消歧、snapshot-vs-flow(raw 条件对每日库存快照求和,产出大约 1000 倍过大的值)、计算度量公式、隐式默认值(哨兵 `PromotionKey=1`、字符串型布尔值),以及时间锚定(raw 对一个截至 2009-12-31 的数据集用 `today()`,返回空结果)[2: §5.4]。
- 残余约 30% 的 semantic-condition 失败集中在文档未编码的模式上 —— 百分位、相关性、标准差、ABC 分类、阈值排序、复杂的多 CTE 透视 —— 可通过用命名度量扩展文档来修复,而非改变机制 [2: §5.4, §6.4]。
- 在稳态生产中,prompt caching 摊销了 schema 加文档的前缀,使 semantic 文档的边际成本实际上为零;Sonnet 4.6 + Sem 以少约 28% 的 input token 匹配 Opus + Sem 的准确率 [2: §5.3]。

## Assumptions

- semantic-layer 文档是权威的,由一位见过该数据集(但未见最终基准问题)的分析师手写;文档质量与覆盖度限定了效应,一个真正盲写的文档可能编码更少的正确知识 [2: §4.2, §6.5]。
- 两种条件下均为单发、无工具、无迭代、无重试、固定"medium"推理强度 —— 被视为代表一个团队把前沿模型对准其数仓所得到的部署 [2: §4.2, §4.3]。
- 上下文形式的收益被当作一个确定性运行时形式 semantic layer(由 compiler 强制)所能提供之物的下界 [2: §3.2, §6.2]。
- analytical 的 LLM-judge 裁决(行级匹配,对良性改写放宽)被当作人类 BI 分析师判断正确性方式的忠实代理 [2: §4.4]。
- judge(claude-sonnet-4-6)与三个被测系统中的两个同属一个模型族;配对设计被假定能跨条件均衡任何自我偏好偏差,而非夸大 delta [2: §4.4, §6.5]。

## Method

- **配对单发基准。** 100 个自然语言问题,跨五个难度级别(simple → medium → complex → advanced → expert)每级均衡 20 个,基于 Cleaned Contoso Retail Dataset(25 张表,2007–2009 销售)载入 ClickHouse 的 `retail` schema 下。每个问题携带一个参考 `expected_sql` 作为 ground truth。q33 被排除(其参考查询超出 ClickHouse 的内存配额),所有配置下留下 n = 99 [2: §4.1, §5.1]。
- **恰在一处不同的两个条件。** *Raw*:问题 + 全部 25 张表的完整 CREATE-TABLE DDL,产出并执行一个 SQL 字符串,无工具/迭代/重试。*Semantic-layer*:相同,外加一个约 4 KB 的 markdown 文档(8,969 字符,约 2,200 token),涵盖事实表选择规则、度量公式(gross margin、return rate、AOV、inventory turnover、same-store growth、recency segments)、维度层级、数据约定/怪癖(字符串布尔值、快照语义、哨兵键)、隐式默认值,以及显式消歧规则。该文档是咨询性的自然语言散文、而非代码,且不被强制 [2: §4.2]。
- **6 个配置**:3 个模型(Claude Opus 4.7 经 `claude -p`、Claude Sonnet 4.6 经 `claude -p`、GPT-5.4 经 `codex exec`)× 2 个条件,每个配置跑全部 100 个问题 [2: §4.3]。
- **评分。** 执行 `expected_sql` 得到 ground-truth 行;执行每个配置的 SQL 得到候选行;一个 LLM judge(claude-sonnet-4-6,JSON-schema 强制输出,单次解析重试)按(问题,配置)返回 pass / fail / excluded。judge 对哪个配置产出了候选是盲的,也不被展示该文档。两种裁决:*strict*(行级匹配,忽略列序与 1% 以下的数值漂移)与 *analytical*(在 strict 基础上放宽,以接纳聚合到参考标量的分组拆解、单位等价、空类别行)。analytical 是主指标 [2: §4.4]。
- **统计。** 每配置 Wilson 95% CI;在配对二元结果上做双侧 exact McNemar,报告为不一致计数 b/c(前者通过 / 后者失败)[2: §4.4]。

## Eval

- **测量**:analytical pass rate(主)、strict pass rate(次要核对)、每模型配对 ΔPR(主效应量)、McNemar 不一致计数 b/c + 双侧 exact p、平均墙钟延迟、平均 input/output token 计数 [2: §4.6]。
- **基线**:raw 仅 schema 条件;每个模型作为自身的配对对照。没有外部 SOTA 比较对象 —— 该设计刻意将文档的效应从模型/族/prompt/工具的混淆中隔离出来 [2: §4, §4.2]。
- **数据 / 指标**:ClickHouse 中的 99 个 Contoso 零售问题;在两种裁决标准下与参考行的二元行级一致。
- **标题数字**(analytical, Table 1):raw 45.5–50.5% → semantic 67.7–68.7%;Opus / Sonnet / GPT-5.4 的 ΔPR 为 +17.2 / +22.2 / +23.2 pp;所有 p ≤ 0.0015。成对矩阵(Table 2):两个干净的簇,每个跨簇比较都显著(多数 p < 0.001),没有簇内比较显著 [2: §5.1–5.2]。
- **综合的外部佐证**(未重跑):BIRD 从一句证据 +20.01 pp;Sequeda 等人借助 ontology 16.7%→54.2%;Bayer/JAMIA 在药物警戒上 +70 pp;dbt Labs 在建模层上 84.1%→100%;Luo 等人的临床 QA 借助 ontology-grounded GraphRAG 37%→98% [2: §2.6, §2.7, §5.1, §7]。

## Weaknesses

- **文档与基准由同一团队共同设计。** 五个"增益"类别(字符串布尔值、哨兵键、快照语义、2009-12-31 锚点、customer-vs-store 地理)恰好既是文档枚举的约定清单,*也是* 基准问题刻意围绕构建的约定 [2: §4.1, §4.2, §5.4]。那 +20 pp 可能部分衡量的是"文档把该基准有意设入的坑题的答案交了出来",而非通用的业务语义迁移。论文只承认较温和的"文档作者未见最终问题"版本 [2: §6.5];它没有标明问题与文档共享同一个编写团队(Cube)。
- **厂商研究依赖厂商佐证。** 作者是 Cube,它销售一个 semantic-layer 运行时;"单一最大杠杆"的结论与产品立场对齐,且若干"跨厂商一致"来源(Snowflake、AtScale、Databricks、Tiger、dbt)本身就是厂商发布的。论文把它们标为"方向性的",却仍将其纳入结构性泛化主张 [2: §2.7, §7]。
- **与 thesis 相关的产物(运行时形式)未被测试,且一个有害的负面结果被埋没。** 只测量了咨询性的上下文形式;能吸收物理变动的运行时/view 形式被留给未来工作 [2: §6.2]。更尖锐的是,§6.5 用一句话 —— 没有数据 —— 报告一个具备等价语义知识的内部 agentic 工具调用系统 *并未* 击败仅 schema 基线,暗示基于搜索的 metadata 发现在单发准确率上不如全上下文入 prompt;一个直接不利于 agentic 数据访问的结果被无证据地披露 [2: §6.5]。
- **没有陈旧化 / 错误文档测试。** 整个前提是一个 *权威* 的静态文档,但没有任何带部分、过时或错误文档的条件 —— 尽管论文自己引用了 BIRD 审计发现 7–10% 的 evidence 标注有误 [2: §2.6]。当数仓演进时文档是否保持正确这一生产关切被完全搁置。
- **动机与验证之间的规模不匹配。** 动机立足于数百到数千列、模型会崩溃的企业级 schema(Spider 2.0、BEAVER)[2: §2.3],但验证是一个 25 表数据集配一个 4 KB 文档。一个手写文档在那种企业规模 —— 正是驱动这项工作的状态 —— 是否仍然可行或可维护,从未被考察。

## Relations

- orthogonal 01_an_agentic_approach_to_metadata_reasoning [med]:MR 处理 *选择哪些表*(schema-linking / 候选选择),通过一个分阶段、经统计合成的画像 metadata catalog [1: §3.2];本文处理 *已供给 schema 的含义*(度量公式、约定、消歧),通过手写的业务语义 [2: §3.1]。两者攻击同一个主导失败类别(schema-linking + 业务逻辑,占错误的 >80%),但处于不同的流水线阶段,且可组合而非竞争 —— MR 可供给一个充分且最小的表集,semantic-layer 文档随后在其中消歧度量。两篇论文互不引用,因此该联系由共享的错误分类法与重叠的 RQ 范围推断而来。
- builds-on (thesis RQ1: typed metadata 作为上下文、而非原始推断) [high]:与 MR [1] 一样,本文实例化了 thesis 的核心机制 —— 在上下文中供给带类型/被描述的 metadata,让模型去查阅而非推断 —— 并以一个受控配对效应(+17–23 pp)佐证之。如今有两个独立的 note 从不同角度支持 RQ1(catalog 选择 vs. 业务语义锚定)。
- builds-on (thesis RQ3: semantic layers / views) [med]:首个直接针对 RQ3 的 note。它演示了 semantic layer 改善锚定,但仅限 *上下文形式*(咨询性散文);能吸收物理变动、为变化源提供稳定把手的运行时/view 形式被断言为一个更优且有下界保证的替代方案 [2: §6.2],却从未被测试 —— 因此 RQ3 的"覆盖变动物理源的稳定把手"属性在此仍未被演示。
- builds-on (thesis RQ4: 符合 ontology 的 metadata) [high]:§6.3 明确把 markdown semantic layer 的因果机制等同于 Sequeda 等人的 OWL/ontology 方法(16.7%→54.2%)与 Allemang & Sequeda 的基于 ontology 的校验(→72%),且效应跨 ontology 研究得到独立佐证(Sequeda、Luo 等人)[2: §2.6, §6.3] —— 跨源一致把该 relation 提升到 high,尽管此处的"ontology"是非正式自然语言散文、而非形式化 schema。
- contradicts (thesis RQ5: freshness / lifecycle) [low]:我的推断 —— 论文的"权威静态手写文档"前提不携带任何 freshness 或 lifecycle 机制,也从未就陈旧化测试,与 thesis 主张(供给必须 *在长任务中保持正确*,而非只被发现一次)存在张力。标为 low,因为论文对 RQ5 是缄默、而非反对。
