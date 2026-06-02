# Thesis — data pillar (research-harness)

> Working thesis for the `data` pillar of the agent-harness research matrix.
> Anchored to the super-repo CHARTER: `data` supplies KNOWLEDGE to the agent
> (structured + unstructured + views + its own catalog/metadata), dual to
> `execution` (which supplies action). Researcher reports contradictions but
> never edits this file — thesis changes are the human's decision.

## Working thesis

A long-horizon agent's reliability is bounded by its **data supply layer**: not
by how clever its reasoning is, but by whether it can *discover, select, and
access the right data* at each step. The core claim: this requires a **catalog
of typed metadata over data resources**, not per-query LLM scans of a raw
corpus. Falsified if production agent systems achieve stable long-task data
grounding with no catalog/metadata layer at all.

Structured, unstructured, and vector data should be unified under **one catalog
+ access interface**, not bespoke per-source glue. Mechanism: metadata describing
each source's shape, freshness, and access path, so the agent selects sources by
description rather than hardcoded wiring. Falsified if unified catalogs
consistently underperform bespoke per-source integrations on grounding quality.

**Data views / semantic layers** give agents stable logical handles over
heterogeneous, changing physical sources — the agent reasons over views; the
view absorbs physical churn. Falsified if agents do better against raw sources
than against a curated semantic layer.

The data catalog's metadata must **conform to a shared ontology/schema** (the
interface to the `ontology` pillar) so semantics stay consistent across
multi-step execution. Falsified if schema-free catalogs suffice for multi-step
semantic consistency.

## Design Context

Building the `data` pillar of an agent harness: the component that supplies
knowledge to an agent doing stable, goal-directed long tasks. Specific gap: most
"data for LLM" work optimizes single-shot retrieval accuracy; little treats data
as a *managed resource* (catalog + connection + view + freshness) the way a
production harness must. Success: an implementable model of a data-resource
catalog + access layer that (a) spans structured/unstructured/view, (b) conforms
to a shared ontology, and (c) keeps supply correct over long tasks.

Pillar boundary (from CHARTER): this pillar manages the **supply of knowledge**.
It does NOT cover deciding/orchestration (`decision`), tool/action execution
(`execution`), or trajectory observability (`trace`). Data metadata is this
pillar's own catalog facet; the shared schema it conforms to is `ontology`.

## Taste
- Favor mechanisms that treat data as a managed resource (catalog, connector, view, freshness) over one-off retrieval tricks.
- Prefer designs unifying structured + unstructured under one abstraction.
- Prefer schema/ontology-grounded metadata over schema-free blob stores.
- Production angle (freshness, access control, connection lifecycle) earns weight.

## Anti-patterns
- RAG accuracy-tuning papers with no catalog/metadata/connection contribution.
- Single-source-only systems with no path to a heterogeneous catalog.
- Benchmark-only papers without a data-management mechanism.
- Work that really belongs to the decision/execution/trace pillars.

## Examples
_(empty until the first notes land)_
