# Research landscape

> Living survey for the `data` pillar. Taxonomy seeded from the working thesis's
> four claims and the CHARTER mandate: a typed-metadata **catalog** + connection
> + view + freshness over **structured / unstructured / vector / view** data,
> conforming to a shared ontology. Each paper slots into the most-specific layer
> bucket below; every entry declares ≥1 explicit relation. Citations are `[N]`
> by note number.

## §1 Catalog & Typed Metadata

_The catalog of typed metadata over data resources — the thesis's core mechanism
(consult fragments, not raw-corpus scans)._

- **Metadata Reasoner (MR)** treats the catalog as a staged, fragment-by-fragment
  evidence source rather than a flat blob: physical schema + LLM-summarized
  statistical profiling ("attached" metadata) is precomputed on the catalog,
  while live tools supply query-time signals [1]. Attached metadata is the single
  largest contributor to selection quality — adding it raises set-aware F1
  73.43% → 79.66% *and* cuts average reasoning steps 13.51 → 8.36, i.e. a
  precomputed typed-metadata layer (not just live probing) is what makes
  selection both more accurate and cheaper [1]. The result presumes a richly
  populated catalog (lineage, data-quality metrics, usage patterns, glossary),
  which the paper concedes most enterprises lack [1] — an enabling precondition,
  not a given.

## §2 Retrieval & Selection

_Discovering and selecting the right data resource(s) for a task._

### §2.1 Agentic / multi-step selection

- **Metadata Reasoner (MR)** [1] — an orchestration agent that decomposes a data
  task into search constraints, then alternates semantic search ↔ reasoning ↔
  tool calls to return a *sufficient + minimal* table subset plus an NL
  justification for downstream agents. Achieves 83.16% average set-aware F1 on
  KramaBench, a ~32-point margin over the best ranking baseline, in only ~10.1
  reasoning steps [1]. Gain is largest in high-cardinality search spaces
  (Astronomy ~1,498 tables: 72.31% vs. 32.80% vector search) [1]. Two notable
  sub-mechanisms: *discrimination-oriented* metadata embeddings to fight semantic
  homogenization of near-duplicate tables, and *state-aware deduplication* that
  suppresses repeat hits to force attention onto new candidates [1].
  - relation: **competes-with** §2.2 ranking baselines [high] — frames "agentic
    combination-selection" against "ranking-based individual retrieval," yet reuses
    a vector engine as a search *subroutine* rather than rejecting retrieval [1].
  - relation: **builds-on** thesis RQ1 [high] — fragmented catalog consult vs.
    per-query raw-corpus LLM scan is a direct, implementable instance of the
    thesis's core claim [1].

### §2.2 Single-shot / ranking retrieval baselines

- **Vector Search** (0.7 cosine threshold + semantic reranker) and **Pneuma**
  (hybrid full-text + vector + LLM-judge re-rank) are the named non-agentic
  comparators; both are dominated by MR (50.77% and 45.12% F1 resp. vs. 83.16%),
  and the vector baseline lets 35.6% noise into its Top-10 where MR stays 99%
  noise-free [1]. Recorded here as the baseline class agentic selection competes
  against, not as standalone contributions.

## §3 Unified Access (structured · unstructured · view · vector)

_One catalog + access interface across heterogeneous source kinds (thesis RQ2)._

- **Metadata Reasoner (MR)** unifies structured + semi-structured tables (and
  treats derived/partitioned tables as view-like resources via injected lineage)
  under one search/selection interface, but **explicitly excludes unstructured
  data**, deferred to future work [1]. It therefore covers only the
  structured/view half of the thesis's intended span.
  - relation: **orthogonal** to thesis RQ2's unstructured arm [med] — supports the
    "one catalog + access interface" goal for structured/view sources only [1].

## §4 Semantic Layers & Views

_Stable logical handles over churning physical sources (thesis RQ3)._

- _(no dedicated paper yet.)_ MR's lineage-based mapping of derived tables to a
  clean base ancestor [1] is the closest adjacent signal — a primitive
  view-over-physical-churn handle — but the paper does not treat views as a
  first-class semantic layer.

## §5 Ontology Conformance & Freshness

_Catalog metadata conforming to a shared ontology (thesis RQ4); freshness /
lifecycle as production concerns._

- **Metadata Reasoner (MR)** states the catalog may optionally include a
  glossary/ontology and "semantic model" as catalog-wide metadata the agent can
  consult [1], and its lifecycle-suffix markers (`_prod/_stg/_test`) encode a
  freshness/quality facet — but ontology conformance is never a hard requirement
  nor measured, so the link is design-compatible, not demonstrated [1].
  - relation: **builds-on** thesis RQ4 [med] — design-compatible with the
    ontology-pillar interface; effect unmeasured [1].

## Relations index

| From | Kind | To | Conf |
|------|------|----|------|
| [1] MR | competes-with | Pneuma / Vector Search baselines | high |
| [1] MR | builds-on | thesis RQ1 (typed-metadata catalog vs raw scan) | high |
| [1] MR | orthogonal | thesis RQ2 (unstructured arm — excluded) | med |
| [1] MR | builds-on | thesis RQ4 (ontology-conformant metadata) | med |
