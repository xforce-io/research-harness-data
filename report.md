# Data Pillar: Research Report

> **Version:** v1 (1 paper)
> **Last Updated:** 2026-06-02
> **Papers:** [01](notes/01_an_agentic_approach_to_metadata_reasoning.md)
> **Thesis:** [.researcher/thesis.md](.researcher/thesis.md)

## Framing

The `data` pillar's bet is that a long-horizon agent's reliability is bounded not
by reasoning cleverness but by its **data supply layer** — whether it can
discover, select, and access the right data at each step. The central design
claim is sharp and falsifiable: this requires a **catalog of typed metadata over
data resources**, consulted in fragments, rather than per-query LLM scans of a
raw corpus. The first paper read is an unusually direct test of exactly this
claim — it builds an agent whose entire job is metadata reasoning over a catalog —
so v1 of this report is mostly about how much weight that single, well-aimed data
point can bear, and which parts of the thesis it leaves completely untouched.

## Goal A — Catalog of typed metadata vs. raw-corpus scan

**What the literature now says.** Strong early support for the core claim. The
Metadata Reasoner (MR) [1] is an orchestration agent that selects a *sufficient +
minimal* table subset for a data task by consulting catalog metadata in stages
(embedding search → attached statistical profiles → on-the-fly tools), never
scanning raw rows wholesale. It reaches 83.16% set-aware F1 on KramaBench vs.
50.77% for a vector-search baseline and 45.12% for Pneuma — a ~32-point margin —
in ~10.1 reasoning steps [1]. The most thesis-relevant ablation: the **precomputed
"attached" metadata layer** (schema + LLM-summarized profiling) is the single
largest contributor, raising F1 73.43% → 79.66% *while cutting* steps 13.51 → 8.36
[1]. That a precomputed typed-metadata layer makes selection simultaneously more
accurate and cheaper is precisely the mechanism the thesis posits. The gap widens
in high-cardinality catalogs (Astronomy ~1,498 tables: 72.31% vs. 32.80%) [1],
which is where "scan the corpus per query" most obviously breaks.

**Residual gap / tension.** Two confounds keep this from being a clean win for the
*architecture*. (i) Every MR variant and the Pneuma baseline run on a single
backbone (Gemini-3-Flash, text-embedding-005); the paper never tests a weaker
model, so the agentic-orchestration gain cannot be cleanly separated from
backbone capability [1]. (ii) The headline presumes a richly populated catalog
(lineage, quality metrics, glossary) that the paper itself concedes most
enterprises lack [1] — so the 83% may not transfer to the schema-and-text-only
catalogs common in practice. The thesis's mechanism is validated; its *real-world
precondition* is not.

**What we should do.** Treat the typed-metadata catalog as the spine of our data
layer, but design for catalog *poverty*: our access layer must degrade gracefully
when only schema+text exists, and ideally bootstrap richer metadata (profiling,
lineage) itself. MR's "attached profiling is precomputed and cheap" finding [1]
argues for making statistical profiling a standing catalog-build step, not a
query-time luxury.

## Goal B — One catalog across structured / unstructured / view / vector

**What the literature now says.** Partial coverage. MR unifies structured +
semi-structured tables under one search/selection interface, and handles
derived/partitioned tables as view-like resources via injected lineage [1]. But it
**explicitly excludes unstructured data**, deferring it to future work [1].

**Residual gap / tension.** The unstructured arm of the thesis's unification claim
(RQ2) has *zero* evidence so far. We have one demonstration that structured +
view can share a catalog interface, and no evidence yet that the same catalog
abstraction extends to unstructured/vector sources without bespoke glue. This is
the single largest open span in the pillar.

**What we should do.** Hold the unification claim as unproven for the
unstructured half. Next reads should prioritize work that puts unstructured or
vector sources under the *same* catalog + selection interface, not separate RAG
pipelines — that is where the thesis is most exposed.

## Goal C — Semantic layers / views as churn absorbers

**What the literature now says.** Almost nothing direct. The closest signal is
MR's lineage-based mapping of every derived/partitioned table back to a clean
base ancestor [1], which functions as a primitive logical handle over physical
variants — but the paper does not treat views as a first-class semantic layer, and
the lineage labels are an evaluation construct (injected, then exposed in
metadata) rather than a runtime view mechanism [1].

**Residual gap / tension.** The thesis claim that agents should reason over
*stable views* while the view absorbs physical churn (RQ3) is untested. MR
reasons over physical tables with lineage hints, not over a curated semantic
layer.

**What we should do.** Keep the semantic-layer claim open and actively seek
evidence. A useful near-term probe: does selection quality improve when the agent
selects over curated views vs. raw physical tables + lineage? MR's lineage
mechanism is a hint that *some* logical indirection helps, but not that a full
semantic layer is the right abstraction.

## Goal D — Ontology conformance as the consistency contract

**What the literature now says.** Design-compatible, unmeasured. MR states the
catalog *may optionally* include a glossary/ontology and "semantic model" as
catalog-wide metadata the agent can consult [1], aligning with the thesis's
interface to the `ontology` pillar. But conformance is never a hard requirement
and its effect is never isolated [1].

**Residual gap / tension.** No measurement of whether ontology-conformant metadata
improves multi-step semantic consistency — the thesis's RQ4 mechanism. MR shows
that *some* catalog-wide semantics help selection, but cannot attribute the gain
to ontology conformance specifically.

**What we should do.** Treat ontology conformance as a hypothesis to instrument,
not an assumption. When we build, we should be able to A/B a schema-free vs.
ontology-grounded catalog and measure consistency drift across long tasks — the
exact comparison MR does not run.

## Goal E — Keeping supply correct over long tasks (freshness / lifecycle)

**What the literature now says.** Thin. MR's only freshness-adjacent mechanism is
lifecycle-suffix markers (`_prod/_stg/_test`, plus quality suffixes like
`_broken_fk`, `_nulls`) used to define "noise-free" selection [1]. MR avoids noise
99% of the time vs. 35.6% noise leakage in the vector baseline [1] — but the
markers were *injected by the authors and exposed in each table's metadata*, so
the noise-avoidance result partly reflects reading injected labels rather than
genuine data-quality reasoning [1]. There is also no latency/cost accounting
beyond step count, and no index-build cost for the discrimination-oriented
embeddings [1].

**Residual gap / tension.** "Keep supply correct over long tasks" (Design Context
goal c) is essentially untested. We have a single-task selection benchmark, not a
long-horizon run where freshness, connection lifecycle, or staleness matter.

**What we should do.** Do not over-read MR's noise-robustness number as freshness
evidence. Our design must make freshness/lifecycle a *measured* catalog facet, and
our own evaluation must run multi-step tasks where the right data changes between
steps — the regime no paper has yet stressed.

## 可证伪点追踪

- **可证伪点 1 — catalog is necessary** ("Falsified if production agent systems
  achieve stable long-task data grounding with **no** catalog/metadata layer at
  all"). Current evidence: *pro*. MR shows a catalog-driven agent strongly
  outperforms catalog-free single-shot retrieval (83.16% vs. ≤50.77% F1) [1], and
  attached metadata independently improves both accuracy and cost [1]. Caveat:
  single-backbone confound leaves room that a stronger model alone could close
  the gap. Next observation that would resolve it: a catalog-free agent matching
  catalog-driven selection on a high-cardinality lake, or the same MR ablation on
  a weaker backbone showing the catalog gain persists.

- **可证伪点 2 — unified catalog ≥ bespoke per-source** ("Falsified if unified
  catalogs consistently underperform bespoke per-source integrations on grounding
  quality"). Current evidence: *insufficient*. MR demonstrates a unified interface
  over structured + view sources only [1]; no head-to-head vs. bespoke
  per-source glue, and unstructured/vector excluded. Next observation: a study
  comparing one catalog interface against tuned per-source pipelines across
  heterogeneous kinds.

- **可证伪点 3 — views beat raw sources** ("Falsified if agents do better against
  raw sources than against a curated semantic layer"). Current evidence: *near-
  zero*. MR reasons over physical tables + lineage hints, not curated views [1].
  Next observation: a selection/grounding comparison of curated-view access vs.
  raw-table access for the same agent.

- **可证伪点 4 — schema/ontology conformance needed for multi-step consistency**
  ("Falsified if schema-free catalogs suffice for multi-step semantic
  consistency"). Current evidence: *insufficient*. MR's optional glossary/semantic
  model is design-compatible but unmeasured [1]. Next observation: an A/B of
  schema-free vs. ontology-grounded catalog measuring semantic drift over a
  multi-step task.

## 版本更新日志
| 版本 | 日期 | 新增论文 | 关键变化 |
|------|------|---------|---------|
| v1 | 2026-06-02 | [01] An Agentic Approach to Metadata Reasoning | Bootstrapped report. Strong support for 可证伪点 1 (catalog necessity) from MR's 83.16% vs ≤50.77% F1 and the attached-metadata ablation; 可证伪点 2/3/4 opened with little-to-no evidence (unstructured arm, semantic layer, and ontology conformance all untested). Flagged single-backbone and catalog-richness confounds. |
