# 01 — An Agentic Approach to Metadata Reasoning

- **arXiv**: 2604.20144
- **Authors**: Jiani Zhang, Sercan Ö. Arık, Cosmin Arad, Fatma Özcan, Alon Halevy (Google)
- **Axes**: data_kind = structured (+ semi-structured, view); layer = retrieval_selection (primary), catalog_metadata (secondary)

## Claims

- The Metadata Reasoner achieves an average set-aware F1 of 83.16% on KramaBench data selection, vs. 50.77% for a vector-search baseline and 45.12% for Pneuma — a ~32-point margin over SOTA [1: §5.1.1, Table 4].
- The gain is largest in high-cardinality search spaces: in the Astronomy domain (~1,498 tables), Full MR reaches 72.31% F1, more than double vector search (32.80%) and Pneuma (27.70%) [1: §5.1.1, Table 4].
- On the synthetically-scaled messy BIRD data lake, MR holds 85.5% average F1 while the Top-10 vector baseline collapses to 30.0% [1: §5.1.2, Table 5].
- MR selects strictly noise-free tables 99.0% of the time; the vector baseline lets 35.6% noise (subsets 13.5%, test splits 10.7%, etc.) into its Top-10 [1: §5.3, Fig. 4].
- High-precision selection translates downstream: feeding MR-selected tables to a Gemini-3-Pro Text-to-SQL generator raises average execution accuracy to 71.28%, from 56.38% with Top-10 vector retrieval [1: §5.2, Table 6].
- Attached (statistically-synthesized) metadata is the single largest contributor: adding it (MR-Search → MR-Search+Attached) raises F1 73.43% → 79.66% *and* cuts average steps 13.51 → 8.36 [1: §5.4, Table 3].
- On-the-fly tool metadata gives the highest accuracy but, without an attached-metadata "warm start," triples steps to 24.14 as the agent does exhaustive trial-and-error [1: §5.4, Table 3].
- Discrimination-oriented embeddings beat schema-only embeddings on Recall@5 in high-cardinality / high-redundancy domains (Astronomy Rec@1 27.98% vs. 14.85%; Legal Rec@5 64.23% vs. 47.06%) [1: §5.5, Table 7].
- Full MR is efficient: it reaches its peak F1 in an average of only 10.10 reasoning steps per task [1: §5.1.1, Table 3].

## Assumptions

- A **comprehensive metadata catalog already exists** and is richly populated — not just schemas and text, but lineage, data-quality metrics, usage patterns, and catalog-wide semantics like glossaries/ontologies [1: §3.1]. The paper concedes most enterprises lack such high-quality metadata [1: §6], so this is an enabling precondition, not a given.
- Data resources are treated as **tables** (structured / semi-structured); unstructured data is explicitly out of current scope and left to future work [1: §3.1, §6].
- The specialized tools — `data_finder()`, `joinability_check()`, `column_profiler()` — are available, deterministic, and trustworthy as evidence sources [1: §3.5.2].
- The optimal split between attached vs. on-the-fly metadata is deployment-dependent (availability, fragment size, latency, agent orchestration capacity) and is treated as a tunable design choice rather than solved [1: §3.5].
- For BIRD evaluation, injected lineage labels accurately map every derived table to its clean base ancestor, and suffix-based quality markers (`_stg`, `_subset`, `_dups`, `_broken_fk`, `_nulls`) faithfully encode ground-truth quality [1: §4.2, §4.4.2].

## Method

- **Inputs**: a data task `Q` (answerable by SQL/Python) + a metadata catalog `T`. **Outputs**: a table subset `T* ⊆ T` that is *sufficient* (covers all required entities/attributes and is correctly joinable) and *minimal* (no smaller subset suffices), plus a natural-language justification `J` that grounds why the tables fit, for downstream agents [1: §3.1].
- **Core design principle**: different metadata types are optimal at different pipeline stages; supplying all metadata at once causes context saturation and degrades reasoning. An orchestration agent autonomously fetches only the fragments needed at each step [1: §3.2, §3.5].
- **Three operational stages**: (1) *retrieval* — table metadata encoded as embedding vectors for search; (2) *evaluation* — reason over metadata explicitly attached to retrieved candidates; (3) *on-the-fly* — specialized tools provide highly specific, query-time metadata signals [1: §3.2].
- **Query decomposition & planning**: decompose `Q` into search constraints (named entities, measures, temporal scopes, granularity), build a multi-faceted search plan, then alternate search ↔ reasoning ↔ tool calls [1: §3.3].
- **Semantic search tool**: ANN search over embeddings. To fight *semantic homogenization* (similar tables → indistinguishable embeddings), a two-stage **discrimination-oriented metadata construction**: (1) group related tables, have an LLM identify shared vs. unique variables and emit a group-aware prompt template; (2) apply the template per table to produce a description emphasizing distinguishing features (e.g., temporal range) [1: §3.4, §3.4.1].
- **State-aware deduplication**: a session dictionary `S` tracks surfaced table IDs and occurrence counts; repeat hits are suppressed and replaced by a recurrence indicator ("Table ID: xxx (Appeared N times)"), forcing attention onto new tables; a search cycle of only duplicates returns a termination signal prompting strategy revision [1: §3.4.2].
- **Attached metadata** (precomputed on `T`): physical schema + LLM-summarized statistical profiling (value ranges, top-K cardinality, null ratios) rendered as NL to save tokens [1: §3.5.1].
- **On-the-fly tools**: `column_profiler()` (exact distinct counts, distributions, histograms), `data_finder()` (deterministic value-level presence checks, query-time), `joinability_check()` (just-in-time FK overlap / referential alignment for specific candidate pairs) [1: §3.5.2].

## Eval

- **Datasets**: (1) **KramaBench** — real-world messy data lake, 6 domains, up to ~1,498 tables (Astronomy), adapted from end-to-end pipelines to isolate the *data-discovery* phase; (2) **Synthetically-scaled BIRD** — 5 databases each in clean + messy versions, with horizontal partitions, lifecycle duplicates (`_prod/_stg/_test`), and low-quality variants (broken FKs, injected NULLs/dups, subsets); 758 analytical questions [1: §4.1, §4.2].
- **Baselines**: Vector Search (non-agentic; 0.7 cosine threshold + semantic reranker; doubles as MR's own search tool), and Pneuma (hybrid full-text + vector + LLM judge re-rank, on Gemini-3-Flash) [1: §4.3]. Ablations: MR-Search, MR-Search+Attached, MR-Search+Tools, Full MR. All MR variants use Gemini-3-Flash via the ADK framework; embeddings via text-embedding-005 [1: §4.3].
- **Metrics**: set-aware **F1** (Recall = sufficiency/coverage, Precision = minimality/conciseness), plus average #steps for cost. Ranking baselines are scored at their *best F1 across all K* (a generous, oracle-K comparison) [1: §4.4.1]. For BIRD (no direct ground-truth set), correctness is verified via lineage: a selected table is correct only if its base ancestor is in the gold SQL, it carries no noise suffix, and (for partitions) its split values match the gold filter [1: §4.4.2].
- **Downstream**: Text-to-SQL execution accuracy with a Gemini-3-Pro generator, comparing Top-10 vector tables vs. MR-selected tables (+ MR justification) [1: §5.2, Table 6].
- **Embedding ablation**: Recall@{1,5,10} comparing schema-only, table-content-summarization, and discrimination-oriented metadata [1: §5.5, Table 7].
- **Failure analysis**: 126 KramaBench failures categorized into 5 modes — missing relation dependencies, granularity mismatch, multi-stage planning failure, redundant relation selection, incomplete partition retrieval [1: §5.6].

## Weaknesses

- **Single-backbone confound**: every MR variant and Pneuma run on Gemini-3-Flash with one embedding model (text-embedding-005) [1: §4.3]. The agentic orchestration gains are inseparable from this backbone's capability; the paper never tests a weaker model, so it cannot show the architecture (not the model) drives the 83% — yet the abstract frames it as a general method result.
- **Catalog-richness dependency understated in the headline**: the KramaBench result presumes lineage/quality/glossary metadata is present, but the paper itself notes most enterprises lack high-quality catalogs [1: §6]. The 83.16% figure may not transfer to the schema-and-text-only catalogs that are common in practice — a scope limit the abstract does not surface.
- **Near-circular noise-robustness setup**: on synthetic BIRD, "correctness" is defined by suffix markers (`_stg`, `_subset`, …) that the authors *injected and then exposed in each table's metadata description* [1: §4.2, §4.4.2]. MR's 99% noise avoidance may partly reflect reading those injected lineage labels rather than genuinely reasoning about data quality; this confound is not acknowledged [1: §5.3].
- **No latency/cost accounting beyond step count**: `data_finder()` / `joinability_check()` touch actual data values and join paths, which carry real compute and I/O cost not quantified; step count alone understates the on-the-fly tool overhead [1: §3.5.2, §4.4.1].
- **Index-build cost unreported**: discrimination-oriented embeddings require a two-stage LLM pass over table *groups* [1: §3.4.1]; the paper gives no cost/scalability figures for constructing this index over a 1,500-table (or larger enterprise) catalog, despite positioning the method against context-window scaling limits.

## Relations

This is the seed note for the `data` pillar — no prior notes in `notes/` exist to relate to. Relations below anchor the paper to the working thesis and research questions so subsequent notes have an explicit attachment point.

- builds-on (thesis RQ1) [high]: The paper's central mechanism — a typed metadata catalog consulted in fragments instead of per-query LLM scans of the whole corpus — is a direct, implementable instance of the thesis claim that long-task reliability is bounded by a *catalog of typed metadata over data resources*, not raw-corpus scans [1: §1, §3.2].
- orthogonal (thesis RQ2: unified structured + unstructured) [med]: MR unifies structured + semi-structured tables under one catalog/search interface, but explicitly excludes unstructured data (deferred to future work) [1: §3.1, §6]; it therefore supports the "one catalog + access interface" goal only for the structured/view half of the thesis's intended span.
- builds-on (thesis RQ4: ontology-conformant metadata) [med]: The catalog is stated to optionally include a glossary/ontology and "semantic model" as catalog-wide metadata the agent can consult [1: §3.1], aligning with the thesis interface to the ontology pillar — though the paper never makes ontology conformance a hard requirement or measures its effect, so the link is design-compatible rather than demonstrated.
- competes-with Pneuma [high]: Pneuma is the paper's named SOTA hybrid-retrieval comparator and is shown to be dominated (45.12% vs. 83.16% F1) [1: §4.3, §5.1.1]; framing is "agentic combination-selection vs. ranking-based individual retrieval," and MR reuses a vector engine as a *subroutine* rather than rejecting retrieval outright.
