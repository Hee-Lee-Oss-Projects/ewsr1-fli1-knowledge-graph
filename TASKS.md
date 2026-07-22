# TASKS — ewsr1-fli1-knowledge-graph

> Status: Draft · Version: 0.2.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated

Itemized backlog for the open knowledge graph of EWSR1-FLI1 / EWSR1-ETS fusion biology in Ewing
sarcoma. See [`PLAN.md`](./PLAN.md) for context, the licensing gate, the not-advice framing, and the
roadmap (M0–M3).

## How these tasks map to Hee-Lee Oss

Each task below becomes a Hee-Lee Oss **Task JSON** validated against `packages/schema/src/schemas.ts`.
Field mapping:

- **id** — stable `ewsr1-fli1-knowledge-graph-<area>-NNN` (the table ID).
- **title** — the table Title.
- **project** — `ewsr1-fli1-knowledge-graph`.
- **type** — one of `code | research | writing | data | design-spec | maintenance` (table "Type").
- **lane** — `donated` for all tasks here. Any future metered extraction run would be `funded` and
  must add `fundedBudgetUsd` (hard per-task budget cap; runs via `packages/runner`).
- **priority** — `high | medium | low`.
- **domain** — e.g. `["cancer-research","open-data","bioinformatics","knowledge-graph","ewing-sarcoma"]`.
- **riskTier** — `low | medium | high`. License + extraction + therapeutic-evidence tasks are
  **medium**; scaffolding/docs are **low**. Any patient-facing item would be **high** (re-tiered).
- **urgent** — boolean (default `false`).
- **deliverable** — `pr | dataset | document | translation` (table "Deliverable").
- **tokenEstimate** — `small | medium | large` (table "Size").
- **status** — `open | in-progress | review | delivered | done` (start `open`).
- **context / objective / acceptanceCriteria[] / resources[] / output** — per task.
- **requestor** — `jdev1977` / beneficiary class until a named partner is secured.
- **verifiedNeed** — **`false`** while no committed partner/steward is secured (honest; the *gap* is
  real but the last-mile beneficiary is TO BE SECURED).
- **outputLicense** — `MIT` (code), `CC0-1.0` (CC0/PD-derived data/docs), or `CC-BY-4.0` /
  `CC-BY-SA-4.0` (where CC BY / CC BY-SA material is incorporated; segregated/labeled).

> **Standing guardrails on every data/extraction task:**
> (1) no source may be touched until its `sources/allowlist.yml` entry is `approved` by the license
> reviewer (PMC-OA resolved **per-article**); (2) any task proposing to ingest **COSMIC, OncoKB,
> DrugBank, any non-open DB, or any controlled-access / patient-level data (dbGaP, EGA, biobanks, EHR)**
> is **refused and flagged** — out of scope, full stop; (3) every therapeutic-target assertion carries
> the **"research evidence — not medical advice"** label + evidence level, and extraction is
> **assistive, flag-on-doubt, never-invent** (gaps stay gaps).

---

## Milestone M0 — Foundation & licensing spine

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ewsr1-fli1-knowledge-graph-ontology-001 | Define core schema (9 classes) aligned to Biolink + standard ontologies | design-spec | small | low | document | — | Maintainer + accuracy reviewer |
| ewsr1-fli1-knowledge-graph-license-001 | Source allow-list schema + licensing-gate policy (PMC-OA per-article; COSMIC/OncoKB/controlled-access hard-exclude) | design-spec | small | medium | document | — | License reviewer |
| ewsr1-fli1-knowledge-graph-license-002 | Analyze & record the 4 candidate sources (CIViC, Open Targets, Reactome, PMC-OA); ≥1 approved | research | medium | medium | document | ewsr1-fli1-knowledge-graph-license-001 | License reviewer |
| ewsr1-fli1-knowledge-graph-prov-001 | Ratify provenance mechanism + countable assertion unit + source-version manifest | design-spec | small | low | document | ewsr1-fli1-knowledge-graph-ontology-001 | Maintainer |
| ewsr1-fli1-knowledge-graph-iri-001 | Commit host-independent persistent IRI namespace (w3id.org) + CURIE policy | design-spec | small | low | document | ewsr1-fli1-knowledge-graph-ontology-001 | Maintainer |
| ewsr1-fli1-knowledge-graph-ci-001 | CI scaffold: SHACL + Biolink-conformance + provenance-completeness + not-advice-label lint | code | medium | low | pr | ewsr1-fli1-knowledge-graph-ontology-001, ewsr1-fli1-knowledge-graph-prov-001 | Maintainer |
| ewsr1-fli1-knowledge-graph-partner-001 | Steward + credentialed expert-reviewer outreach (status logged) | research | small | low | document | — | Maintainer |

**Acceptance criteria (key M0 tasks)**

- **ewsr1-fli1-knowledge-graph-ontology-001**
  - All core classes (Gene, Fusion, FusionBreakpoint, Disease, Pathway, Mechanism, TherapeuticAgent,
    EvidenceAssertion, SourceDocument) defined with properties, datatypes, and cardinality.
  - Each class/edge **mapped to the Biolink Model** and to HGNC/MONDO/SO/Reactome/ChEMBL/GO/ECO where
    an equivalent exists; gaps documented.
  - Fusion + FusionBreakpoint model the EWSR1-ETS family (FLI1, ERG, FEV, ETV1/4) and Type 1/2
    breakpoint subtypes (Sequence Ontology).
  - EvidenceAssertion reserves provenance + evidence-level + source-release + "not-advice" slots.
  - Conflicting/superseded-claim representation specified (no single forced "truth").
  - Published as a Biolink-conformant schema + SHACL stub + human-readable spec.
- **ewsr1-fli1-knowledge-graph-license-001**
  - `sources/allowlist.yml` schema defines: name, custodian, URL/endpoint, format, license/PD basis,
    rights analysis, **release/version**, and `status: approved|rejected|pending`.
  - Policy states COSMIC/OncoKB/DrugBank/any non-open DB and all controlled-access/patient-level data
    are categorically `rejected`.
  - PMC-OA **per-article** license-resolution requirement documented (CC BY / NC / ND / CC0 / PD);
    facts-not-verbatim-text rule stated; CC BY-SA (ChEMBL) segregation rule stated.
- **ewsr1-fli1-knowledge-graph-license-002**
  - The 4 candidate sources analyzed with recorded license/PD + release; ≥1 marked `approved`.
  - Reactome and Open Targets licenses confirmed for the *specific release* (not assumed).
  - Anti-laundering controls referenced (passage/record-level citation, attestation, spot-checks).
- **ewsr1-fli1-knowledge-graph-prov-001**
  - One provenance mechanism chosen (nanopublications vs named-graphs+PROV-O vs RDF-star), applied
    uniformly; the countable **"assertion" unit** defined so the 100%-provenance CI gate is checkable.
  - **Source-version manifest** format defined (CIViC snapshot, Open Targets release, Reactome version,
    PMC-OA snapshot) so any assertion is reproducible.
- **ewsr1-fli1-knowledge-graph-ci-001**
  - CI fails on any assertion lacking a provenance link or source-release stamp.
  - CI runs SHACL validation + **Biolink-conformance** on exported edges.
  - CI rejects data referencing a source not marked `approved`.
  - CI fails on any therapeutic-target assertion missing the "not medical advice" label + evidence level.

**M0 Definition of Done:** schema v0 published (Biolink-aligned); allow-list schema + policy merged
with the 4 sources analyzed and ≥1 approved (PMC-OA per-article policy + COSMIC/OncoKB/controlled-access
hard-exclude); provenance mechanism ratified **with the countable assertion unit + source-version
manifest defined**; **host-independent persistent IRI namespace committed**; CI provenance + SHACL +
Biolink + not-advice-label gates live; steward + expert-reviewer outreach initiated with status logged;
**a qualified License/ToS reviewer AND a credentialed biomedical/oncology accuracy reviewer named
(hard exit; if a seat is empty M0 cannot exit — escalate per the documented fallback in PLAN.md)**.
`pnpm build && pnpm test && pnpm lint` green.

---

## Milestone M1 — First sourced slice

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ewsr1-fli1-knowledge-graph-import-001 | Structured importer for CIViC/Open Targets (assistive, flag-on-doubt) | code | medium | medium | pr | ewsr1-fli1-knowledge-graph-ci-001, ewsr1-fli1-knowledge-graph-license-002 | Maintainer |
| ewsr1-fli1-knowledge-graph-data-001 | Import one approved source (CIViC EWSR1/FLI1 evidence) → assertions with provenance | data | large | medium | dataset | ewsr1-fli1-knowledge-graph-import-001 | Accuracy reviewer |
| ewsr1-fli1-knowledge-graph-pmc-001 | License-checked PMC-OA fact-extraction prototype (facts, not verbatim text) | data | large | medium | dataset | ewsr1-fli1-knowledge-graph-import-001 | License + accuracy reviewers |
| ewsr1-fli1-knowledge-graph-retraction-001 | Retraction/supersession screening gate (PubMed + Retraction Watch) | code | small | medium | pr | ewsr1-fli1-knowledge-graph-data-001 | Maintainer |
| ewsr1-fli1-knowledge-graph-qa-001 | Expert citation + grounding audit of M1 batch (stratified, ≥95%) | research | small | medium | document | ewsr1-fli1-knowledge-graph-data-001, ewsr1-fli1-knowledge-graph-pmc-001 | Accuracy reviewer |
| ewsr1-fli1-knowledge-graph-export-001 | KGX/Biolink + JSON-LD + Turtle + nanopub export tooling | code | medium | low | pr | ewsr1-fli1-knowledge-graph-data-001 | Maintainer |
| ewsr1-fli1-knowledge-graph-partner-002 | Engage ≥1 candidate steward + the expert reviewer | research | small | low | document | ewsr1-fli1-knowledge-graph-partner-001 | Maintainer |

**Acceptance criteria (key M1 tasks)**

- **ewsr1-fli1-knowledge-graph-import-001**
  - Maps CIViC / Open Targets records to typed assertions, preserving each source's native **evidence
    level** and **release version** into provenance.
  - Therapeutic-target assertions auto-carry the "research evidence — not medical advice" label.
  - Ambiguous fields are flagged for human review, never guessed.
- **ewsr1-fli1-knowledge-graph-data-001**
  - Records from one approved source mapped to Gene/Fusion/Disease/EvidenceAssertion/SourceDocument.
  - **100%** of new assertions carry a resolvable provenance link (source ID + license + release +
    method + evidence level).
  - Conflicting/superseded source statements retained with separate provenance + evidence levels.
  - Passes CI (SHACL + Biolink + provenance + not-advice-label + allow-list) before review.
- **ewsr1-fli1-knowledge-graph-pmc-001**
  - Only articles whose **per-article license is resolved and `approved`** are read.
  - Output is **structured facts with passage-level citations (PMCID + location)** — no verbatim
    redistribution of copyrighted text (esp. CC BY-NC / CC BY-NC-ND).
  - Extraction is assistive; uncertain extractions are flagged for human review, not asserted.
- **ewsr1-fli1-knowledge-graph-retraction-001**
  - Every cited PMID is screened against PubMed retraction notices + the Retraction Watch dataset.
  - Retracted/withdrawn sources are flagged and their assertions withheld or annotated.
- **ewsr1-fli1-knowledge-graph-qa-001**
  - A **stratified** sample (min size ≥200 or whole release; strata by source and extraction method) is
    checked against the cited open source; ≥95% verify.
  - The **auditor is independent of the extractor** of the sampled records (no self-grading).
  - Entity grounding sampled separately: **zero confirmed false ID mappings**.
  - Citations checked are passage/record-level; collection-level citations rejected.

**M1 entry precondition (hard gate):** the exact source releases for the M1 batch are named and recorded
in the allow-list as license-cleared (CIViC/Open Targets CC0 verified; any PMC-OA article license
resolved per-article) **before** any extraction begins.

**M1 Definition of Done:** end-to-end pipeline proven on one approved structured source + a
license-checked PMC-OA prototype; 100% provenance + source-release + evidence level; retraction
screening run; stratified expert citation audit (≥200, independent auditor) ≥95% and zero confirmed
false normalizations; KGX/Biolink + JSON-LD + Turtle exports produced under the persistent-IRI
namespace with a source-version manifest; ≥1 candidate steward + the expert reviewer engaged.

---

## Milestone M2 — Normalization, conflict & explorer

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ewsr1-fli1-knowledge-graph-norm-001 | Entity grounding/normalization to HGNC/MONDO/Reactome/ChEMBL/SO (precision-first) | data | medium | medium | dataset | ewsr1-fli1-knowledge-graph-data-001 | Accuracy reviewer |
| ewsr1-fli1-knowledge-graph-norm-002 | Human-reviewed conflict + duplicate resolution workflow | code | medium | medium | pr | ewsr1-fli1-knowledge-graph-norm-001 | Maintainer + accuracy reviewer |
| ewsr1-fli1-knowledge-graph-explorer-001 | Public explorer (no accounts/PII; shows source + evidence level + "not advice") | code | medium | low | pr | ewsr1-fli1-knowledge-graph-export-001 | Maintainer |
| ewsr1-fli1-knowledge-graph-docs-001 | Data dictionary + usage/citation + "not medical advice" guide | writing | small | low | document | ewsr1-fli1-knowledge-graph-explorer-001 | Maintainer |
| ewsr1-fli1-knowledge-graph-metrics-001 | Reuse-metrics tracking (downloads/queries) | maintenance | small | low | document | ewsr1-fli1-knowledge-graph-explorer-001 | Maintainer |

**Acceptance criteria (key M2 tasks)**

- **ewsr1-fli1-knowledge-graph-norm-001**
  - Entities grounded to standard IDs (gene→HGNC/Ensembl, disease→MONDO, pathway→Reactome,
    compound→ChEMBL, sequence feature→SO); ≥95% grounding coverage.
  - Uses an explicit **candidate-blocking** strategy (normalized symbol + entity type + context); the
    grounder is tuned for **precision over recall**.
  - **Zero confirmed false mappings** in the grounding audit sample.
- **ewsr1-fli1-knowledge-graph-norm-002**
  - No automatic merge/conflict-resolution ships without human confirmation; merges are reversible and
    preserve all source provenance + evidence levels.
  - Contradictory findings are retained side-by-side (conflict-faithful), not flattened.
- **ewsr1-fli1-knowledge-graph-explorer-001**
  - Browse a Fusion/Gene/Pathway and see its assertions, **each with source, evidence level, and the
    "not medical advice" label**.
  - Static / no-account; collects no visitor PII; links to raw KGX/JSON-LD/Turtle exports.

**M2 Definition of Done:** grounding to standard IDs at ≥95% coverage with zero confirmed false
mappings; conflict/duplicate workflow operational; explorer + documented exports live (source +
evidence + not-advice surfaced); reuse metrics tracked.

---

## Milestone M3 — Scale & partner adoption (shipped)

| ID | Title | Type | Size | Risk | Deliverable | Depends on | Reviewer |
| --- | --- | --- | --- | --- | --- | --- | --- |
| ewsr1-fli1-knowledge-graph-data-002 | Integrate ≥4 sources; scale to ≥2,000 sourced assertions | data | large | medium | dataset | ewsr1-fli1-knowledge-graph-norm-002, ewsr1-fli1-knowledge-graph-license-002 | Accuracy + license reviewers |
| ewsr1-fli1-knowledge-graph-partner-003 | Secure steward adoption + ≥1 documented citation/use | research | medium | low | document | ewsr1-fli1-knowledge-graph-partner-002 | Maintainer |
| ewsr1-fli1-knowledge-graph-sustain-001 | Sustainability: source-refresh cadence + reviewer rotation/throughput plan | writing | small | low | document | ewsr1-fli1-knowledge-graph-partner-003 | Maintainer |

**Acceptance criteria (key M3 tasks)**

- **ewsr1-fli1-knowledge-graph-data-002**
  - ≥4 total open sources integrated; ≥2,000 sourced assertions published.
  - Each source passed the license gate (per release) before extraction; 100% provenance maintained.
  - 100% of therapeutic assertions labeled + evidence-leveled; retraction screening current.
  - A fresh stratified citation audit still ≥95% verified; grounding still zero confirmed false mappings.
- **ewsr1-fli1-knowledge-graph-partner-003**
  - A named research/advocacy steward (or KG consortium) commits to adopt/host and cite the graph.
  - ≥1 concrete citation or downstream reuse (e.g. Monarch/Translator ingest) is documented.

**M3 Definition of Done (project "shipped"):** ≥2,000 sourced assertions across ≥4 open sources; 100%
provenance + source-release; 100% therapeutic assertions labeled/evidence-leveled; retraction screening
current; credentialed expert reviewer engaged; ≥1 partner has adopted/cited the graph; persistence +
source-refresh + reviewer-rotation sustainability plan in effect.

---

## Backlog / future (sized, unscheduled)

| ID | Title | Type | Size | Risk | Deliverable | Notes |
| --- | --- | --- | --- | --- | --- | --- |
| ewsr1-fli1-knowledge-graph-data-003 | Reactome pathway-perturbation enrichment for EWSR1-FLI1 targets | data | medium | medium | dataset | Verify Reactome release license |
| ewsr1-fli1-knowledge-graph-pmc-002 | Broaden PMC-OA literature extraction coverage (license-checked) | data | large | medium | dataset | Highest license-risk; per-article gate |
| ewsr1-fli1-knowledge-graph-sparql-001 | Optional hosted SPARQL endpoint | code | large | low | pr | Depends on steward hosting |
| ewsr1-fli1-knowledge-graph-nanopub-001 | Publish assertions to a nanopublication network/feed | code | medium | low | pr | Downstream interoperability |
| ewsr1-fli1-knowledge-graph-quality-001 | Automated anomaly/outlier flagging for review | code | medium | low | pr | Assistive QA, human-confirmed |
| ewsr1-fli1-knowledge-graph-i18n-001 | Multilingual entity labels via ontology cross-refs | data | small | low | dataset | CC0 labels where available |

---

## Generated task index

Every backlog row above is now decomposed into one schema-valid `tasks/<id>.json` (validated against
`packages/schema` taskSchema). **Mapping is 1:1 — no fan-out:** each row names a single concrete task
(no enumerated language/source/document dimension to expand), so each becomes exactly one task JSON.
Acceptance criteria are reused verbatim from this file where the milestone "Acceptance criteria"
blocks list them; for rows that lacked explicit per-task criteria (e.g. `iri-001`, `partner-001`,
`export-001`, `partner-002`, `docs-001`, `metrics-001`, `partner-003`, `sustain-001`, and the
backlog rows) checkable criteria were authored from each row's Title + Definition of Done. All tasks
are `lane: donated`, `status: open`, `verifiedNeed: false` (no committed steward yet — TO BE SECURED).

Standing guardrails are carried into every task `context`: open/PD/openly-licensed sources only;
COSMIC/OncoKB/DrugBank/any non-open DB and all controlled-access/patient-level data are refused and
out of scope; PMC-OA is resolved per-article (facts, not verbatim text); every therapeutic-target
assertion is labeled "research evidence — not medical advice". No row in this backlog requests refused
content (medical advice, eligibility, prognosis), so no task was skipped.

- **M0:** `ewsr1-fli1-knowledge-graph-ontology-001` (seed), `-license-001`, `-license-002`,
  `-prov-001`, `-iri-001`, `-ci-001`, `-partner-001`
- **M1:** `-import-001`, `-data-001`, `-pmc-001`, `-retraction-001`, `-qa-001`, `-export-001`,
  `-partner-002`
- **M2:** `-norm-001`, `-norm-002`, `-explorer-001`, `-docs-001`, `-metrics-001`
- **M3:** `-data-002`, `-partner-003`, `-sustain-001`
- **Backlog:** `-data-003`, `-pmc-002`, `-sparql-001`, `-nanopub-001`, `-quality-001`, `-i18n-001`

Total: 28 task files (1 pre-existing seed + 27 generated).

---

## Example task JSON

Schema-valid Task JSON for the first M0 task (`ewsr1-fli1-knowledge-graph-ontology-001`):

```json
{
  "id": "ewsr1-fli1-knowledge-graph-ontology-001",
  "title": "Define core schema (9 classes) aligned to Biolink + standard ontologies",
  "project": "ewsr1-fli1-knowledge-graph",
  "type": "design-spec",
  "lane": "donated",
  "priority": "high",
  "domain": ["cancer-research", "open-data", "bioinformatics", "knowledge-graph", "ewing-sarcoma"],
  "riskTier": "low",
  "urgent": false,
  "deliverable": "document",
  "tokenEstimate": "small",
  "status": "open",
  "context": "An open, linked knowledge graph of EWSR1-FLI1 / EWSR1-ETS fusion biology in Ewing sarcoma needs a shared, Biolink-aligned schema before any data is ingested. Sources are OPEN only: CIViC (CC0), Open Targets (CC0), Reactome (verify release license), and the license-checked PMC open-access subset. COSMIC, OncoKB, DrugBank, and any controlled-access or patient-level data (dbGaP, EGA, biobanks, EHR) are out of scope. All therapeutic-target content is research evidence and must be labeled 'not medical advice'. See PLAN.md.",
  "objective": "Define the nine core classes (Gene, Fusion, FusionBreakpoint, Disease, Pathway, Mechanism, TherapeuticAgent, EvidenceAssertion, SourceDocument) and their edges, mapping each to the Biolink Model and to HGNC/MONDO/SO/Reactome/ChEMBL/GO/ECO wherever an equivalent exists, modeling the EWSR1-ETS partner family and breakpoint subtypes, and representing conflicting/superseded findings rather than flattening them.",
  "acceptanceCriteria": [
    "All 9 classes defined with properties, datatypes, and cardinality.",
    "Each class/edge mapped to the Biolink Model and to HGNC/MONDO/SO/Reactome/ChEMBL/GO/ECO where one exists; gaps documented.",
    "Fusion and FusionBreakpoint model the EWSR1-ETS family (FLI1, ERG, FEV, ETV1/4) and Type 1/2 breakpoint subtypes via Sequence Ontology.",
    "EvidenceAssertion reserves provenance, evidence-level, source-release, and 'not medical advice' slots.",
    "Conflicting/superseded-assertion representation specified (no single forced 'truth').",
    "Published as a Biolink-conformant schema plus a SHACL stub and a human-readable spec."
  ],
  "resources": [
    "planning/projects/ewsr1-fli1-knowledge-graph/PLAN.md",
    "planning/ROADMAP.md",
    "https://biolink.github.io/biolink-model/",
    "https://civicdb.org/",
    "https://platform.opentargets.org/",
    "https://reactome.org/"
  ],
  "output": "A schema specification document (ontology/README.md) plus a Biolink-conformant schema and SHACL stub defining the 9 classes and their ontology mappings, committed via PR.",
  "requestor": "jdev1977",
  "verifiedNeed": false,
  "outputLicense": "CC0-1.0"
}
```
