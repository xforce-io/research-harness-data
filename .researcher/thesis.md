# Thesis — data pillar (research-harness)

> agent-harness 研究矩阵中 `data` 支柱的 working thesis。
> 锚定于超级仓 CHARTER:`data` 为 agent 供给 KNOWLEDGE(结构化 + 非结构化 + views + 其自有的 catalog/metadata),与 `execution`(供给动作)对偶。Researcher 报告矛盾,但从不编辑本文件 —— thesis 的变更是人类的决定。

## Working thesis

长程 agent 的可靠性受限于其 **data supply layer(数据供给层)**:不是受限于其推理有多巧妙,而是受限于它能否在每一步 *发现、选择并访问正确的数据*。核心主张:这要求一个 **覆盖数据资源的带类型 metadata catalog**,而非对原始语料做逐查询的 LLM 扫描。若生产级 agent 系统在完全没有任何 catalog/metadata 层的情况下实现稳定的长任务数据锚定,则证伪。

结构化、非结构化与 vector 数据应统一在 **单一 catalog + 访问接口** 下,而非定制的逐源粘合。机制:用 metadata 描述每个源的形态、freshness 与访问路径,使 agent 按描述选择源、而非硬编码接线。若统一 catalog 在锚定质量上持续不及定制的逐源集成,则证伪。

**Data views / semantic layers** 为 agent 提供覆盖异构、变化物理源的稳定逻辑把手 —— agent 在 views 上推理;view 吸收物理变动。若 agent 面对原始源比面对一个精选的 semantic layer 表现更好,则证伪。

data catalog 的 metadata 必须 **符合共享 ontology/schema**(到 `ontology` 支柱的接口),使语义在多步执行间保持一致。若无 schema 的 catalog 足以保证多步语义一致性,则证伪。

## Design Context

构建 agent harness 的 `data` 支柱:为执行稳定、目标导向长任务的 agent 供给知识的组件。具体缺口:大多数"data for LLM"工作优化单发检索准确率;很少有工作像生产级 harness 所必须的那样,把数据当作一个 *受管资源*(catalog + connection + view + freshness)。成功标准:一个可实现的数据资源 catalog + 访问层模型,它(a)横跨 structured/unstructured/view,(b)符合共享 ontology,(c)在长任务中保持供给正确。

支柱边界(出自 CHARTER):本支柱管理 **知识的供给**。它 *不* 涵盖决策/编排(`decision`)、工具/动作执行(`execution`)或轨迹可观测性(`trace`)。数据 metadata 是本支柱自有的 catalog 维面;它所符合的共享 schema 即 `ontology`。

## Taste
- 偏好把数据当作受管资源(catalog、connector、view、freshness)的机制,而非一次性的检索技巧。
- 偏好把结构化 + 非结构化统一在一个抽象下的设计。
- 偏好 schema/ontology 锚定的 metadata,而非无 schema 的 blob 存储。
- 生产视角(freshness、访问控制、connection lifecycle)赋予额外权重。

## Anti-patterns
- 无 catalog/metadata/connection 贡献的 RAG 准确率调优论文。
- 仅单源、无路径通往异构 catalog 的系统。
- 无数据管理机制的纯基准论文。
- 实际归属于 decision/execution/trace 支柱的工作。

## Examples
_(尚无笔记时留空)_
