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
- **Semantic Layer (Cube)** [2] corroborates the core "typed metadata as context,
  not inference" claim from the business-semantics angle: adding a described
  semantic layer in-context (vs. raw schema only) lifts first-shot analytical
  accuracy +17.2–23.2 pp across three frontier models in a controlled paired
  design, every McNemar p ≤ 0.0015 [2: §5.1]. A second, independent instance of
  RQ1's mechanism — here the metadata is informal hand-authored NL prose rather
  than a precomputed statistical profile [1].
  - relation: **builds-on** thesis RQ1 [high] — supply described metadata so the
    model looks up rather than infers; composable with [1] MR (selection) since
    this paper grounds meaning, not table choice [2: §3.1].

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

- **Semantic Layer (Cube)** [2] — first dedicated RQ3 paper. A ~4 KB hand-authored
  markdown semantic-layer document (measure formulas, dimensional hierarchies, data
  conventions, disambiguation rules) added to the prompt raises first-shot
  analytical accuracy +17.2–23.2 pp across three frontier models (e.g. 45.5%→68.7%),
  every paired McNemar p ≤ 0.0015 [2: §5.1]. The effect is *structural*: with the
  document the three models are statistically indistinguishable, and without it also
  indistinguishable — document presence, not model tier, accounts for essentially
  all pairwise variance [2: §5.2]. Mechanism: a semantic layer converts the dominant
  text-to-SQL error class (schema-linking + business-logic, >80% of failures) from
  open-ended inference into constrained lookup [2: §3.1, §6.1]. **Caveat:** only the
  *context form* (advisory prose in the prompt) is measured; the runtime/view form
  that would actually absorb physical churn — RQ3's "stable handle over changing
  sources" property — is asserted as a superior lower bound but left to future work,
  and no staleness / wrong-document condition is run [2: §6.2, §6.5].
  - relation: **orthogonal** to [1] MR [med] — MR selects *which tables* via a staged
    profiling catalog; this paper grounds *what the supplied schema means* via
    business semantics. Same dominant error class, different pipeline stage,
    composable not competing; neither cites the other [2: §3.1].
  - relation: **builds-on** thesis RQ3 [med] — demonstrates the semantic-layer
    benefit in context form only; the churn-absorbing view form remains
    undemonstrated [2: §6.2].
- MR's lineage-based mapping of derived tables to a clean base ancestor [1] is an
  adjacent signal — a primitive view-over-physical-churn handle — but MR does not
  treat views as a first-class semantic layer.

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
- **Semantic Layer (Cube)** [2] raises RQ4 from design-compatible to measured: §6.3
  explicitly equates the markdown semantic layer's causal mechanism with formal
  ontology approaches — Sequeda et al.'s OWL ontology (16.7%→54.2%) and Allemang &
  Sequeda's ontology-based validation (→72%) — and the grounding effect is
  independently corroborated across ontology studies [2: §6.3, §2.6]. Cross-source
  agreement, though here the "ontology" is informal NL prose, not a formal schema.
  On freshness: the premise is an *authoritative static* document with no staleness
  or lifecycle mechanism and no wrong-document test [2: §6.5] — adjacent to this
  bucket's freshness concern but explicitly unaddressed.
  - relation: **builds-on** thesis RQ4 [high] — controlled paired effect plus
    cross-study ontology corroboration; effect now measured, not just
    design-compatible [2: §5.1, §6.3].

## Relations index

| From | Kind | To | Conf |
|------|------|----|------|
| [1] MR | competes-with | Pneuma / Vector Search baselines | high |
| [1] MR | builds-on | thesis RQ1 (typed-metadata catalog vs raw scan) | high |
| [1] MR | orthogonal | thesis RQ2 (unstructured arm — excluded) | med |
| [1] MR | builds-on | thesis RQ4 (ontology-conformant metadata) | med |
| [2] Semantic Layer | orthogonal | [1] MR (different pipeline stage) | med |
| [2] Semantic Layer | builds-on | thesis RQ1 (typed metadata as context) | high |
| [2] Semantic Layer | builds-on | thesis RQ3 (semantic layers / views — context form) | med |
| [2] Semantic Layer | builds-on | thesis RQ4 (ontology-conformant metadata) | high |
| [2] Semantic Layer | contradicts | thesis RQ5 (freshness / lifecycle) | low |
