# research-harness-data

> [research-harness](https://github.com/xforce-io/research-harness)
> agent-harness 研究矩阵的 **`data` 支柱**。

**范围:** 面向 agent harness 的数据资源管理 —— 一个覆盖 **结构化、非结构化、view 与 vector** 数据、并符合共享 ontology 的 **带类型 metadata + connection 的 catalog**。该支柱为执行稳定、目标导向长任务的 agent 供给 *知识*(与供给 *动作* 的 `execution` 对偶)。

**边界(依据超级仓 CHARTER):** 仅管理 *知识的供给* —— 不涉及决策/编排(`decision`)、工具/动作执行(`execution`)或轨迹可观测性(`trace`)。数据 metadata 是本支柱自有的 catalog 维面;它所符合的共享 schema 即 `ontology`。

working thesis 见 [`.researcher/thesis.md`](.researcher/thesis.md),thesis 驱动的综述见 [`report.md`](report.md),持续更新的 survey 见 [`notes/00_research_landscape.md`](notes/00_research_landscape.md)。

## Thesis (summary)

- 长程 agent 的可靠性受限于其 **data supply layer(数据供给层)** —— 即它在每一步发现、选择并访问到正确数据的能力。
- 这要求一个覆盖异构源的 **带类型 metadata catalog**,而非对原始语料做逐查询的 LLM 扫描。
- 单一统一的 catalog + 访问接口应横跨结构化、非结构化、view 与 vector 数据,并以 **semantic-layer views** 吸收物理变动。
- catalog metadata 必须 **符合共享 ontology**,使语义在多步执行间保持一致。

完整的 working thesis 见 [.researcher/thesis.md](.researcher/thesis.md)。

## Papers

_Last Updated: 2026-06-03_

| # | 标题 | Layer / Axis | 优先级 | 已读 |
|---|-------|--------------|----------|------|
| 01 | [An Agentic Approach to Metadata Reasoning](notes/01_an_agentic_approach_to_metadata_reasoning.md) | retrieval/selection(主),catalog/metadata(次)· structured + view | 高 | ✅ |
| 02 | [Semantic Layers for Reliable LLM-Powered Data Analytics](notes/02_semantic_layers_for_reliable_llm_powered.md) | view/semantic layer(主),catalog/metadata(次)· structured | 高 | ✅ |
