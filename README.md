# research-harness-data

> The **`data` pillar** of the [research-harness](https://github.com/xforce-io/research-harness)
> agent-harness research matrix.

**Scope:** data resource management for agent harnesses — a **catalog of typed
metadata + connections** over **structured, unstructured, view, and vector**
data, conforming to a shared ontology. The pillar that supplies *knowledge* to
an agent doing stable, goal-directed long tasks (dual to `execution`, which
supplies *action*).

**Boundary (per super-repo CHARTER):** manages the *supply of knowledge* only —
not deciding/orchestration (`decision`), tool/action execution (`execution`), or
trajectory observability (`trace`). Data metadata is this pillar's own catalog
facet; the shared schema it conforms to is `ontology`.

See [`.researcher/thesis.md`](.researcher/thesis.md) for the working thesis,
[`report.md`](report.md) for the thesis-driven synthesis, and
[`notes/00_research_landscape.md`](notes/00_research_landscape.md) for the living
survey.

## Thesis (summary)

- A long-horizon agent's reliability is bounded by its **data supply layer** —
  its ability to discover, select, and access the right data at each step.
- This requires a **catalog of typed metadata** over heterogeneous sources, not
  per-query LLM scans of a raw corpus.
- One unified catalog + access interface should span structured, unstructured,
  view, and vector data, with **semantic-layer views** absorbing physical churn.
- Catalog metadata must **conform to a shared ontology** so semantics stay
  consistent across multi-step execution.

See [.researcher/thesis.md](.researcher/thesis.md) for the full working thesis.

## Papers

_Last Updated: 2026-06-02_

| # | Title | Layer / Axis | Priority | Read |
|---|-------|--------------|----------|------|
| 01 | [An Agentic Approach to Metadata Reasoning](notes/01_an_agentic_approach_to_metadata_reasoning.md) | retrieval/selection (primary), catalog/metadata (secondary) · structured + view | High | ✅ |
