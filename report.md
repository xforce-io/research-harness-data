# Data Pillar: Research Report

> **Version:** v2 (2 papers)
> **Last Updated:** 2026-06-03
> **Papers:** [01](notes/01_an_agentic_approach_to_metadata_reasoning.md), [02](notes/02_semantic_layers_for_reliable_llm_powered.md)
> **Thesis:** [.researcher/thesis.md](.researcher/thesis.md)

## Framing

The `data` pillar's bet is that a long-horizon agent's reliability is bounded not
by reasoning cleverness but by its **data supply layer** — whether it can
discover, select, and access the right data at each step. The central design
claim is sharp and falsifiable: this requires a **catalog of typed metadata over
data resources**, consulted as fragments, rather than per-query LLM scans of a raw
corpus. The two papers read so far attack this claim from opposite ends of the
same pipeline: [1] builds an agent whose entire job is *selecting which data* via
metadata reasoning over a catalog; [2] runs a controlled experiment on *grounding
what the selected data means* via a semantic layer. They never cite each other,
yet both land on the same mechanism — supply described/typed metadata so the model
looks up rather than infers — and both report large effects from it. v2 of this
report is about how much that convergence strengthens the core claim (RQ1), where
[2] newly opens the semantic-layer arm (RQ3) the thesis had no evidence for, and
which parts of the thesis (unstructured unification, freshness over long tasks)
remain untouched or newly in tension.

## Goal A — Catalog of typed metadata vs. raw-corpus scan

**What the literature now says.** Two independent data points, same conclusion. The
Metadata Reasoner (MR) [1] is an orchestration agent that selects a *sufficient +
minimal* table subset by consulting catalog metadata in stages (embedding search →
attached statistical profiles → on-the-fly tools), never scanning raw rows
wholesale; it reaches 83.16% set-aware F1 on KramaBench vs. 50.77% (vector search)
and 45.12% (Pneuma) in ~10.1 reasoning steps [1]. Its most thesis-relevant ablation:
the **precomputed "attached" metadata layer** (schema + LLM-summarized profiling)
is the single largest contributor, raising F1 73.43% → 79.66% *while cutting* steps
13.51 → 8.36 [1] — a typed-metadata layer makes selection simultaneously more
accurate and cheaper. [2] tests the same "metadata-as-context, not inference"
mechanism on a different stage and with a cleaner design: a single ~4 KB
hand-authored semantic-layer document added to the prompt lifts first-shot
analytical accuracy +17.2–23.2 pp across three frontier models (e.g. 45.5%→68.7%),
every paired McNemar p ≤ 0.0015 [2: §5.1]. Crucially, [2]'s effect is *structural*:
with the document the three models are statistically indistinguishable and without
it also indistinguishable — document presence, not model tier, accounts for
essentially all pairwise variance [2: §5.2]. That directly addresses MR's worst
confound (below): the grounding gain in [2] is *not* a backbone-capability artifact.

**Residual gap / tension.** (i) MR's single-backbone confound persists in [1] (every
variant + Pneuma run on Gemini-3-Flash; no weaker-model test) — but [2]'s
model-invariance result is strong indirect evidence that the *typed-metadata-as-
context* lever is real independent of backbone strength [2: §5.2]. (ii) Both papers
presume *rich* metadata that may not exist: MR concedes most enterprises lack the
lineage/quality/glossary catalog it assumes [1]; [2]'s document is hand-authored by
an analyst who had seen the dataset [2: §4.2]. The mechanism is doubly validated;
its *real-world precondition* (where does the rich metadata come from, and who
maintains it) is validated by neither. (iii) A genuine cross-paper tension on
*access method*: [2] discloses in one data-free sentence that an internal
agentic-tool-use system with equivalent semantic knowledge did **not** beat the
all-context schema-only baseline [2: §6.5] — adverse to MR's agentic-retrieval
premise. The two are not directly comparable (selection vs. generation; F1 vs.
pass-rate), but they bracket an open design question: when does staged/agentic
catalog retrieval beat simply inlining the metadata?

**What we should do.** Treat the typed-metadata catalog as the spine, but make
catalog *richness* a first-class problem, not an assumption: our access layer must
degrade gracefully on schema+text-only catalogs and ideally *bootstrap* richer
metadata itself (MR's "attached profiling is precomputed and cheap" [1] argues for
profiling as a standing catalog-build step). On access method, resolve the [1]/[2]
tension structurally: agentically select a sufficient+minimal set when the catalog
is large, then inline that set + a semantic layer for grounding when it fits the
context window — make "is the relevant metadata small enough to inline?" an explicit
routing decision rather than always-retrieve or always-inline.

## Goal B — One catalog across structured / unstructured / view / vector

**What the literature now says.** Still partial, and now doubly so. MR unifies
structured + semi-structured tables under one search/selection interface and treats
derived/partitioned tables as view-like resources via injected lineage, but
**explicitly excludes unstructured data** [1]. [2] adds a semantic-layer grounding
mechanism but is also **structured-only** (25-table ClickHouse retail warehouse,
text-to-SQL) [2: §4.1]. Two papers, zero coverage of the unstructured/vector arm.

**Residual gap / tension.** The unstructured arm of the unification claim (RQ2) has
*zero* evidence after two reads — the single largest open span in the pillar. We
now have two demonstrations that structured (+ view) sources benefit from a shared
metadata interface, and still no evidence that the same catalog abstraction extends
to unstructured/vector sources without bespoke glue.

**What we should do.** Hold the unstructured unification claim as unproven and
actively prioritize it. Next reads should target work that puts unstructured or
vector sources under the *same* catalog + selection/grounding interface, not a
parallel RAG pipeline — that is where the thesis is most exposed and where both
papers so far are silent.

## Goal C — Semantic layers / views as churn absorbers

**What the literature now says.** This goal flips from "almost nothing" to "first
direct, controlled evidence — for half the claim." [2] is the first paper aimed at
RQ3: a hand-authored semantic layer (measure formulas, dimensional hierarchies,
data conventions, disambiguation rules) supplied in-context converts the dominant
text-to-SQL failure class (schema-linking + business-logic, >80% of errors) from
open-ended inference into constrained lookup [2: §3.1, §6.1], for a +17–23 pp gain
[2: §5.1]. The wins concentrate exactly where physical sources mislead an agent:
snapshot-vs-flow semantics (raw sums daily inventory snapshots, ~1000× too large),
sentinel keys, string-valued booleans, time anchoring against a dataset ending
2009-12-31 [2: §5.4]. MR's lineage-based mapping of derived tables to a clean base
ancestor [1] remains a weaker adjacent signal — a primitive view-over-physical-churn
handle, but not a first-class semantic layer.

**Residual gap / tension.** [2] validates the *grounding* benefit of a semantic
layer but only in its **context form** (advisory NL prose in the prompt). The
property the thesis actually asserts — a *stable logical handle while the view
absorbs physical churn* — requires the **runtime/view form** (compiler-enforced,
deterministic), which [2] asserts is a superior lower-bounded alternative but leaves
entirely to future work [2: §6.2]. There is also no staleness or wrong-document
test, so "does the handle stay stable as physical sources change?" — the core of
RQ3 — is undemonstrated [2: §6.5]. And a confound: [2]'s benchmark and document were
co-authored by one team (Cube), so the wins may partly measure "the document hands
over the answers to the trick conventions this benchmark deliberately included"
rather than generic transfer [2: §4.1, §5.4, §6.5].

**What we should do.** Bank the result that *some* semantic layer helps grounding,
but keep the churn-absorption claim open: the near-term probe is now sharper than in
v1 — does a *runtime/view* semantic layer hold accuracy when the physical schema
changes underneath it, vs. a context-form document that must be re-authored? Build
the semantic layer as an enforced runtime artifact (so it can absorb churn), not a
prompt string (which [2] only proves for a static snapshot), and test it under
deliberate physical-source drift.

## Goal D — Ontology conformance as the consistency contract

**What the literature now says.** Upgraded from "design-compatible, unmeasured" to
"measured by proxy, cross-source corroborated." MR still only *optionally* allows a
glossary/ontology as catalog-wide metadata, never isolating its effect [1]. But [2]
§6.3 explicitly equates the markdown semantic layer's causal mechanism with formal
ontology approaches — Sequeda et al.'s OWL ontology (16.7%→54.2%) and Allemang &
Sequeda's ontology-based validation (→72%) — and the grounding effect is
independently corroborated across multiple ontology studies (Sequeda, Luo et al.
clinical QA 37%→98% with ontology-grounded GraphRAG) [2: §2.6, §6.3]. Cross-source
agreement raises confidence that *described semantics* drive the gain.

**Residual gap / tension.** The corroboration is for the *family* "structured
semantics in context" — but [2]'s own artifact is informal NL prose, not a formal
schema, and it does not isolate *conformance to a shared ontology* (the thesis's
specific RQ4 mechanism, the interface to the `ontology` pillar) from "any described
semantics help." So we have stronger evidence that semantics-as-context works, and
still no controlled test that *ontology conformance specifically* buys multi-step
semantic consistency over an ad-hoc-but-described layer.

**What we should do.** Keep ontology conformance as a hypothesis to instrument, but
sharpen the A/B: not just schema-free vs. ontology-grounded, but *ad-hoc described
semantics* (a Cube-style markdown doc) vs. *ontology-conformant described semantics*
(same content, but typed against the shared schema), measuring consistency drift
across a multi-step task. [2] shows described semantics help; it does not show the
*conformance* is what carries the load.

## Goal E — Keeping supply correct over long tasks (freshness / lifecycle)

**What the literature now says.** Still thin, and now in mild tension. MR's only
freshness-adjacent mechanism is lifecycle/quality suffix markers
(`_prod/_stg/_test`, `_broken_fk`, `_nulls`) used to define "noise-free" selection,
but those markers were author-injected and exposed in metadata, so the 99%
noise-avoidance number partly reflects reading injected labels [1]. [2] makes the
gap explicit rather than closing it: its entire premise is an *authoritative static*
hand-authored document, with **no** staleness, partial-document, or wrong-document
condition tested — even though it cites BIRD audits finding 7–10% of evidence
annotations wrong [2: §2.6, §6.5].

**Residual gap / tension.** "Keep supply correct over long tasks" (Design Context
goal c) is the least-tested thesis claim and is now in *direct tension* with [2]'s
frame: [2] demonstrates a one-shot grounding effect from a frozen, correct artifact,
while the thesis asserts the data layer's job is to keep supply correct *as sources
churn* — a property a write-once document does not have. (Logged as a vs-thesis
contradiction this run.) Neither paper runs a long-horizon task where the right data,
or the semantic layer itself, goes stale between steps.

**What we should do.** Do not read either paper's static-snapshot result as freshness
evidence. Make freshness/lifecycle a *measured* catalog facet, and treat the semantic
layer itself as metadata that must stay fresh and conformant — not a write-once doc.
Our own evaluation must run multi-step tasks where the right data (and the layer
describing it) changes between steps — the regime no paper has yet stressed.

## 可证伪点追踪

- **可证伪点 1 — catalog is necessary** ("Falsified if production agent systems
  achieve stable long-task data grounding with **no** catalog/metadata layer at
  all"). Current evidence: *pro, strengthened*. MR shows catalog-driven selection
  beats catalog-free single-shot retrieval (83.16% vs. ≤50.77% F1) [1]; [2] shows
  described metadata in context beats raw-schema-only grounding (+17–23 pp, paired
  p ≤ 0.0015) [2: §5.1] — and, importantly, [2]'s model-invariance result rebuts the
  single-backbone confound that weakened [1] by showing the lever is structural, not
  capability-driven [2: §5.2]. Open caveat: [2]'s §6.5 reports (no data) that an
  agentic-tool-use variant did *not* beat all-context — so "catalog is necessary"
  holds, but "agentic catalog *retrieval* is the right access pattern" is contested.
  Next observation that would resolve it: a head-to-head of agentic retrieval vs.
  inlined metadata at fixed accuracy, varying catalog size.

- **可证伪点 2 — unified catalog ≥ bespoke per-source** ("Falsified if unified
  catalogs consistently underperform bespoke per-source integrations on grounding
  quality"). Current evidence: *insufficient, unchanged*. Both [1] and [2] are
  structured-only [1; 2: §4.1]; no head-to-head vs. bespoke per-source glue, and the
  unstructured/vector arm is untouched by two papers. Next observation: a study
  comparing one catalog interface against tuned per-source pipelines across
  heterogeneous source kinds.

- **可证伪点 3 — views beat raw sources** ("Falsified if agents do better against raw
  sources than against a curated semantic layer"). Current evidence: *pro for the
  context form; open for the view form*. [2] is the first direct test: a curated
  semantic layer beats raw schema by +17–23 pp [2: §5.1], the effect concentrating on
  exactly the physical-source traps (snapshot-vs-flow, time anchoring, sentinel keys)
  [2: §5.4]. But only the advisory *context form* is measured; the *runtime/view*
  form that would absorb physical churn is untested [2: §6.2], and there is no
  staleness condition [2: §6.5]. So "views beat raw sources" is supported for static
  curated semantics; "views *as churn absorbers*" is still unproven. Next observation:
  same-agent accuracy on a runtime view vs. raw tables *under deliberate physical
  schema drift*.

- **可证伪点 4 — schema/ontology conformance needed for multi-step consistency**
  ("Falsified if schema-free catalogs suffice for multi-step semantic consistency").
  Current evidence: *partial pro, but mis-targeted*. [2] §6.3 ties its effect to
  formal-ontology results (Sequeda 16.7%→54.2%; Allemang & Sequeda →72%) and
  corroborates across ontology studies [2: §2.6, §6.3] — strong evidence that
  *described semantics* help. But [2]'s own artifact is informal prose, and it never
  isolates *ontology conformance* from "any described semantics," nor measures
  multi-step consistency. Next observation: an A/B of ad-hoc-described vs.
  ontology-conformant described semantics, measuring semantic drift over a multi-step
  task.

## 版本更新日志
| 版本 | 日期 | 新增论文 | 关键变化 |
|------|------|---------|---------|
| v1 | 2026-06-02 | [01] An Agentic Approach to Metadata Reasoning | Bootstrapped report. Strong support for 可证伪点 1 (catalog necessity) from MR's 83.16% vs ≤50.77% F1 and the attached-metadata ablation; 可证伪点 2/3/4 opened with little-to-no evidence (unstructured arm, semantic layer, and ontology conformance all untested). Flagged single-backbone and catalog-richness confounds. |
| v2 | 2026-06-03 | [02] Semantic Layers for Reliable LLM-Powered Data Analytics | 可证伪点 3 flips from near-zero to *pro (context form)* — first direct semantic-layer test (+17–23 pp, model-invariant). 可证伪点 1 strengthened: [2]'s model-invariance rebuts MR's single-backbone confound. 可证伪点 4 upgraded to partial-pro via cross-study ontology corroboration, but conformance still not isolated. New cross-paper tension (agentic retrieval vs. all-context, §6.5) and a vs-thesis freshness contradiction (static authoritative doc, no staleness test) logged. Unstructured arm (可证伪点 2) still zero evidence after two structured-only papers. |
