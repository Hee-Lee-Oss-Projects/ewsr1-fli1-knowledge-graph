# PLAN — ewsr1-fli1-knowledge-graph

> Status: Draft · Version: 0.2.0 · Last updated: 2026-06-28 · Owner: TBD (maintainer) · Lane: donated

An open, linked, citable knowledge graph of **EWSR1-FLI1 / EWSR1-ETS fusion biology** in Ewing
sarcoma — genes, fusions and breakpoint subtypes, regulated target genes, mechanisms, pathways,
diseases, and research-stage therapeutic targets — where **every assertion carries machine-resolvable
provenance** to a specific OPEN source. Built **only** from open / public-domain / openly-licensed
data (CIViC, Open Targets, Reactome, and the PMC open-access subset). **No patient data. Not medical
advice.**

## Executive summary

Ewing sarcoma is a rare, aggressive bone-and-soft-tissue cancer of children, adolescents, and young
adults, driven in ~85% of cases by the **EWSR1-FLI1** fusion oncoprotein (and in most of the
remainder by other **EWSR1-ETS** fusions such as EWSR1-ERG). The biology — a pioneer transcription
factor that binds GGAA microsatellites and rewires gene regulation — is documented across thousands of
open-access papers and several openly-licensed databases, but that knowledge is **fragmented across
silos with incompatible identifiers and no shared provenance**. A researcher who wants "every
sourced assertion about what EWSR1-FLI1 regulates, through what mechanism, with what therapeutic
implication, and where each claim comes from" must reassemble it by hand.

This project builds an openly-licensed knowledge graph (RDF/JSON-LD + KGX/Biolink exports) of
EWSR1-ETS fusion biology, **aligned to the Biolink Model and standard biomedical ontologies**
(HGNC, MONDO/EFO, Sequence Ontology, Reactome, ChEMBL, GO, ECO) so it is interoperable and
reconcilable against the wider biomedical linked-data web (Monarch Initiative, Translator) from day
one. **Every assertion is a provenance-bearing unit** — subject, predicate, object, evidence level,
the exact open source consulted, and the source's release version — modeled on the CIViC/SEPIO
evidence pattern. A lightweight public explorer and standard exports make it usable by
non-programmers and machine-consumable by other open projects.

The single most important constraint is **licensing and provenance** (see
[Data, licensing & compliance](#data-licensing--compliance)). Two hard boundaries define the project:
(1) **Non-open oncology databases (COSMIC, OncoKB) and any controlled-access or individual-level
patient data (dbGaP, EGA, biobanks, clinical records) are OUT OF SCOPE and never acceptable** — they
are non-commercial/licensed or require authorized access + IRB; and (2) the **PMC open-access subset is
license-heterogeneous** — it contains CC BY, CC BY-NC, CC BY-NC-ND, CC0, and public-domain articles,
so each article's license is verified per-article and the graph extracts **structured facts, never
redistributes verbatim copyrighted text**. This is the headline gate; it is checked before any source
is touched.

Risk tier: **medium** — driven primarily by (a) per-source license compliance, (b) biomedical
extraction/normalization accuracy, and (c) the requirement that all therapeutic-target content be
framed as **research evidence, explicitly "not medical advice,"** and reviewed by a credentialed
biomedical/oncology expert. **No partner/steward is yet secured (`verifiedNeed: false`)** — the
research *gap* is real and demonstrable, but a committed last-mile beneficiary is **TO BE SECURED**.

## Problem & beneficiaries

**The problem.** EWSR1-ETS fusion biology is real, deep, and almost entirely public — but
*un-integrated*. Curated evidence lives in CIViC; target–disease associations in Open Targets;
pathway membership in Reactome; mechanistic detail in tens of thousands of PMC open-access papers.
These use **different identifiers** (gene symbol vs. HGNC vs. Ensembl vs. UniProt; disease names vs.
MONDO vs. NCIt), **different evidence models**, and carry **no shared, machine-resolvable provenance**.
There is no free, openly-licensed, queryable graph that connects the EWSR1-FLI1 fusion to the genes it
regulates, the mechanism (e.g. GGAA-microsatellite pioneer-factor activity), the pathways affected,
and the research-stage therapeutic hypotheses — *with a citation on every link.*

**Beneficiaries.**
- **Ewing/sarcoma researchers and bioinformaticians** who need a queryable, citable, source-linked
  starting point instead of re-curating the literature by hand.
- **Computational biologists / KG and AI/ML researchers** who need clean, normalized, Biolink-aligned
  training and reasoning substrate for a rare-cancer driver.
- **Patient-advocate "research primer" authors and science communicators** who translate the biology
  for families — a downstream, *non-patient-facing* audience that needs sourced, accurate facts.
- **Downstream open projects** (Monarch Initiative, Translator, other open biomedical KGs) that can
  reconcile against and reuse the graph.
- **The research commons** — an open, auditable map of a rare-cancer driver where commercial
  aggregators dominate.

**Verified need.** The *gap* (no open, provenance-bearing EWSR1-ETS KG) is real and demonstrable.
However, a **named partner organization or steward who will adopt, host, cite, and validate the
output is TO BE SECURED.** `verifiedNeed` is recorded **`false`** until that beneficiary signs on.
Per the quality bar ("delivered, not merged"), securing a steward **and** a credentialed expert
reviewer is a first-class M0/M1 objective and a precondition for declaring the project *shipped*.

**Partner org.** TO BE SECURED. Candidate stewards: a Ewing sarcoma research foundation or
patient-advocacy *research* program, an academic sarcoma/pediatric-oncology lab, a rare-disease data
commons, or an open biomedical KG consortium (Monarch/Translator) willing to ingest the graph.

## Goals and non-goals

**Goals**
- Publish an openly-licensed, queryable knowledge graph of EWSR1-ETS fusion biology with
  **provenance on every assertion** and a verifiable evidence level.
- Define a reusable schema **aligned to the Biolink Model** and standard ontologies, covering the
  core classes (Gene, Fusion, FusionBreakpoint, Disease, Pathway, Mechanism, TherapeuticAgent,
  EvidenceAssertion, SourceDocument).
- Seed the graph from open sources only — CIViC (CC0), Open Targets (CC0), Reactome, and the
  license-checked PMC open-access subset — recording each source's **release version** in provenance.
- Represent **conflicting / superseded preclinical findings faithfully** with their evidence levels,
  rather than flattening to a single "truth."
- Label **all therapeutic-target content as research evidence, "not medical advice,"** and distinguish
  preclinical from clinical evidence on every such assertion.
- Provide KGX/Biolink + RDF/Turtle + JSON-LD exports (and a nanopublication feed) and a simple public
  explorer — so Monarch/Translator and others can reuse it.
- Secure at least one research/advocacy partner that adopts and cites the graph, and a credentialed
  biomedical/oncology expert reviewer.

**Non-goals**
- **Not** a clinical decision tool, treatment recommender, trial-matcher, or patient-facing
  variant-interpretation service. It describes *biology and research evidence*, not care.
- **Not medical advice** of any kind. (If any deliverable ever becomes patient-facing, it is
  **escalated to risk-tier `high`** and gated behind oncologist + patient-advocate sign-off.)
- **Not** a scrape, mirror, or re-publication of **COSMIC, OncoKB, DrugBank,** or any non-open /
  non-commercial / licensed oncology database.
- **Not** a holder of **patient-level or controlled-access data** (dbGaP, EGA, biobanks, EHR,
  identifiable specimens). Aggregate / published / openly-licensed data only.
- **Not** a redistribution of verbatim copyrighted article text; it extracts structured *facts*.
- **Not** original wet-lab or computational research that *generates* new findings; it structures what
  open sources attest. **AI never invents or "fills in" unknown biology; gaps stay gaps.**
- **Not** a general-purpose cancer KG; scope is EWSR1-ETS fusion biology in Ewing sarcoma.

## Success metrics (outcomes)

Outcome-based and beneficiary-centric. Baselines are zero at project start unless noted.

| Metric | Baseline | Target (first 12 months) |
| --- | --- | --- |
| EvidenceAssertions published with ≥1 resolvable open-source citation | 0 | ≥ 2,000 |
| Share of published assertions carrying machine-resolvable provenance + source release version | n/a | 100% (hard gate; un-sourced assertions are never published) |
| Distinct open sources integrated | 0 | ≥ 4 (CIViC + Open Targets + Reactome + license-checked PMC-OA) |
| Entities normalized to a standard ontology ID (HGNC/MONDO/Reactome/ChEMBL/SO) | 0 | ≥ 95% of entities grounded |
| **False-normalization rate** (wrong gene/disease/drug ID in audit sample) | n/a | **zero confirmed false mappings** (precision over recall; a wrong ID silently corrupts the graph) |
| Biolink-conformance of exported edges (validated) | n/a | 100% of exported edges valid against the chosen Biolink version |
| Therapeutic-target assertions labeled "research evidence — not medical advice" + evidence level | n/a | 100% (hard gate) |
| Retraction/supersession screening of cited publications at each release | n/a | 100% of cited PMIDs screened; retracted sources flagged/withheld |
| Curation accuracy: stratified expert audit of sampled assertions verifying against the cited open source | n/a | ≥ 95% verified (see sampling frame) |
| Conflict-faithful representation (contested facts retaining ≥2 sourced variants) | n/a | 100% — no single-winner assertion ships without a sourced rationale |
| Research/advocacy partners adopting or citing the graph | 0 | ≥ 1 committed steward; ≥ 1 citation/use |
| Credentialed biomedical/oncology expert reviewer engaged | none | ≥ 1 named (hard M0 exit) |
| Reuse: external queries / downloads of exports | 0 | tracked from M2; growth trend reported quarterly |

**Citation-audit sampling frame (pins the accuracy metric).** The ≥95% target is meaningless without a
defined sample. The audit uses: a **minimum sample size** of 200 assertions per release (or the whole
release if smaller); **stratified sampling** across strata defined by *source* (CIViC / Open Targets /
Reactome / PMC-OA) and by *extraction method* (structured DB import vs. literature extraction), so no
source or method escapes review; and an **auditor independent of the extractor** of the sampled
records (no self-grading). The normalization-precision audit draws its own stratified sample of
proposed/confirmed entity groundings.

We explicitly **do not** treat raw assertion count, PRs merged, or commits as success. A large graph
with weak provenance, wrong identifiers, or unlabeled therapeutic claims is a **failure** under this
plan.

## Scope

**In scope**
- Schema/ontology aligned to Biolink + standard biomedical ontologies, and a source allow-list (with
  per-source and, for PMC-OA, per-article license/PD determination recorded).
- Structured import of EWSR1-ETS fusion biology from CIViC, Open Targets, and Reactome.
- License-checked structured **fact** extraction from the PMC open-access subset (not verbatim text).
- Entity grounding/normalization to standard IDs, duplicate/conflict resolution with human review.
- Provenance, license, evidence-level, and source-release metadata on every assertion.
- Retraction/supersession screening of cited literature.
- KGX/Biolink + RDF/Turtle + JSON-LD exports, a nanopublication feed, and a simple public explorer.
- Outreach for a steward and a credentialed expert reviewer.

**Out of scope (explicit)**
- **Any scraping, bulk download, API harvesting, or re-publication of COSMIC, OncoKB, DrugBank, or any
  non-open / non-commercial / licensed oncology database.** A hard refusal under Hee-Lee Oss guardrails.
- **Any controlled-access or individual-level patient data** (dbGaP, EGA, biobanks, EHR, identifiable
  specimens/images). Requires authorized access + IRB and is categorically excluded.
- Redistribution of verbatim copyrighted article text (incl. CC BY-NC / CC BY-NC-ND full text).
- **Any patient-facing clinical guidance, treatment recommendation, dosing, or trial matching.** Such
  content, if ever proposed, is escalated to `high` risk with oncologist + advocate sign-off.
- AI-generated "fill-in" of unknown biology; gaps stay gaps.
- Paid features, accounts, or for-profit packaging of the data.
- Curation of evidence by a contributor with an undisclosed financial tie to the drug/program in
  question (conflict-of-interest rule, below).

## Solution approach & architecture

A **data/content pipeline** project (with supporting code), not a hosted clinical service.

**Pipeline stages**
1. **Source intake & licensing gate** — each candidate source is logged in a machine-readable
   `sources/allowlist.yml` with: name, custodian, URL/endpoint, format, **license / PD determination**,
   rights basis, **release/version identifier**, and `status: approved | rejected | pending`. For
   PMC-OA, license is resolved **per-article** (CC BY / CC BY-NC / CC BY-NC-ND / CC0 / PD) before any
   text is read. Nothing is processed until `approved`.
2. **Schema/ontology layer** — a documented schema (`ontology/`) defining the core classes and their
   edges, **mapped to the Biolink Model** and to HGNC, MONDO/EFO, Sequence Ontology, Reactome, ChEMBL,
   GO, and the **ECO** (Evidence & Conclusion Ontology) / **SEPIO**-style evidence model. Published as
   a Biolink-conformant schema + SHACL shapes + human-readable spec.
3. **Extraction / import**
   - **Structured importers** for CIViC, Open Targets, and Reactome map their records to typed
     assertions, preserving each source's native evidence level and release version.
   - **Literature extraction** over the *license-checked* PMC-OA subset is **assistive** (LLM-aided):
     it emits *structured facts* (subject–predicate–object + evidence type) with a passage-level
     citation, **never verbatim redistribution**, and **flags ambiguous fields for human review —
     never guesses**.
4. **Normalization & provenance binding** — entities are grounded to standard IDs
   (gene→HGNC/Ensembl, disease→MONDO, pathway→Reactome, compound→ChEMBL, sequence feature→SO); **every
   assertion is wrapped with a provenance record** (source ID + license + source release version +
   extraction method + evidence level + confidence) using the mechanism fixed in M0.
5. **Conflict & dedup resolution** — candidate equivalences and contradictions are proposed
   automatically and **confirmed by a human reviewer**; grounding uses an explicit **candidate-blocking
   strategy** (e.g., normalized symbol + entity type + context) tuned for **precision over recall**.
   The bar is **zero confirmed false mappings** in the audit sample. Conflicting findings are retained
   side-by-side with evidence levels, not flattened.
6. **Retraction/supersession screening** — cited PMIDs are checked against PubMed retraction notices
   and the open Retraction Watch dataset; retracted/withdrawn sources are flagged and their assertions
   withheld or annotated. Run at every release.
7. **Validation** — SHACL shapes + **Biolink-conformance** check + provenance-completeness linter +
   "not-advice-label-present" check on therapeutic assertions (CI gates; no assertion without
   provenance, no therapeutic assertion without a label).
8. **Publish** — versioned KGX/Biolink TSV + RDF/Turtle + JSON-LD dumps + a nanopublication feed, a
   query surface, and a simple explorer UI, each stamped with a **source-version manifest**.

**Tech stack**
- Tooling/importers/extractors/validators: **TypeScript, ESM, pnpm** (per Hee-Lee Oss conventions).
- Graph serialization: **RDF (Turtle) + JSON-LD** with dereferenceable IRIs + **KGX/Biolink** TSV for
  downstream KG ingestion; **nanopublications** for assertion-level provenance interchange.
- Validation: **SHACL** shapes + Biolink-conformance + a custom provenance/label linter in CI.
- Normalization: identifiers.org / Biolink CURIE prefixes; scripted grounders with a human worklist.
- Explorer: a static-site explorer over the JSON-LD — no accounts, no PII; shows each assertion's
  source, evidence level, and "not medical advice" label.

**Data model (core classes)**
- **Gene** — EWSR1, the **EWSR1-ETS partner family** (FLI1, ERG, FEV, ETV1, ETV4), regulated target
  genes — aligned to HGNC / NCBI Gene / Ensembl (`biolink:Gene`).
- **Fusion** — the EWSR1-ETS fusion oncoprotein (EWSR1-FLI1, EWSR1-ERG, …) — modeled as a
  gene-product/sequence-feature with its two partner genes.
- **FusionBreakpoint** — breakpoint/junction subtype (e.g. EWSR1-FLI1 **Type 1 / Type 2** exon
  fusions) using Sequence Ontology terms — so scope is biologically precise, not just "EWSR1-FLI1."
- **Disease** — Ewing sarcoma / Ewing family tumors — aligned to MONDO / EFO / NCIt / Orphanet.
- **Pathway** — Reactome pathways the fusion perturbs (`biolink:Pathway`).
- **Mechanism** — molecular mechanism nodes (e.g. GGAA-microsatellite binding, pioneer-factor
  activity, transcriptional activation/repression).
- **TherapeuticAgent** — research-stage compounds/targets (e.g. aligned to ChEMBL) — **research
  evidence only; not advice.**
- **EvidenceAssertion** — the **provenance-bearing unit**: subject–predicate–object + evidence type
  (ECO) + evidence level + source + source release + extraction method + confidence. This *is* the
  countable "assertion."
- **SourceDocument** — the PMC article or database release, carrying its license/PD status + version.

**Key decisions (to ratify in M0)**
- **Provenance mechanism & assertion unit:** **nanopublications** vs **named graphs + PROV-O** vs
  **RDF-star** — pick one and apply uniformly. (Leaning: nanopublications, which are purpose-built for
  biomedical assertion + provenance + publication-info and interoperate with the open KG ecosystem;
  named-graphs+PROV-O evaluated as the fallback.) The chosen mechanism **defines the countable
  "assertion"** the 100%-provenance CI gate measures, so the gate is mechanically checkable, not
  aspirational.
- **IRI scheme & persistence:** stable, dereferenceable IRIs under a **host-independent persistent
  identifier** (w3id.org) as the canonical namespace, **decoupled from the (unsecured) steward**, with
  identifiers.org/Biolink CURIE prefixes for grounded entities. No IRI is minted under a host we do not
  control.
- **Conflict & evidence representation:** every contested/superseded finding retains all sourced
  variants with their evidence levels; no single "winner" without a sourced editorial rationale.
- **Source-version manifest:** every release records the exact CIViC snapshot date, Open Targets
  release, Reactome version, and PMC-OA snapshot date used, so any assertion is reproducible.

## Data, licensing & compliance

**This is the headline gate for the project. Read this section before doing any data work.**

### Hard boundary 1 — non-open oncology databases are out of scope
A knowledge graph **scraped, harvested, or bulk-copied from COSMIC, OncoKB, DrugBank, or any other
non-open / non-commercial / licensed oncology database** would violate those licenses and is therefore
**OUT OF SCOPE and never acceptable** under Hee-Lee Oss's license guardrails. This includes automated
harvesting, re-publishing their records/derived fields, or laundering such data through an
intermediary. We assume both contractual (license/ToS) and database-copyright protections apply and
**do not touch these systems** absent an explicit written open-release agreement.

### Hard boundary 2 — no patient-level or controlled-access data
**Controlled-access and individual-level patient data (dbGaP, EGA, biobanks, EHR/clinical records,
identifiable specimens or images) are categorically excluded.** They require authorized access + IRB
oversight, which is outside this project's scope. Only **aggregate / published / openly-licensed** data
is used. **No patient data, ever.**

### The PMC open-access nuance (the subtle gate)
The **PMC open-access subset is license-heterogeneous**: it contains **CC BY, CC BY-NC, CC BY-NC-ND,
CC0, and public-domain** articles. "In the OA subset" does **not** mean "freely redistributable for
derivatives." Therefore:
- **Per-article license is resolved and recorded before any text is read** (CC BY / CC BY-NC /
  CC BY-NC-ND / CC0 / PD), from the PMC OA license field.
- The graph extracts **structured facts** (which are not themselves copyrightable) with passage-level
  citations; it **does not redistribute verbatim full text**, especially from NC/ND articles.
- The **output dataset license is segregated** by source license (see Attribution below).

### Closing the citation-laundering hole
A contributor could copy a claim from a closed paper or a non-open database (COSMIC/OncoKB) and
back-fill a plausible open citation. Three controls make that detectable and unacceptable:
- **Passage/record-level citations are required** — an assertion must cite the specific PMCID +
  section/passage location actually consulted, or the specific DB record ID + release version — never a
  database or journal "as a whole." Collection-level citations are rejected at review.
- **Contributor attestation of open-only sourcing** — each contributed batch carries a signed
  attestation that the facts were read from the cited open source directly, not from COSMIC/OncoKB,
  paywalled papers, or any non-open compilation.
- **Spot-checks for non-open-DB-distinctive fields** — reviewers sample for tell-tale artifacts of
  COSMIC/OncoKB (their distinctive normalizations, derived/curated fields, or identifiers) that would
  not appear in the raw open source; a hit triggers rejection and a flag.

### Approved sources (open / openly-licensed only)
Every source must be entered in `sources/allowlist.yml` with a recorded license basis, **release
version**, and `approved` status before use:
- **CIViC** (Clinical Interpretation of Variants in Cancer) — **CC0**. Curated variant/evidence
  records; native evidence levels preserved into provenance.
- **Open Targets Platform** — **CC0** (verify per-release). Target–disease association evidence;
  note Open Targets itself aggregates upstream sources whose terms are respected.
- **Reactome** — pathway data; **verify the exact release's license (CC0 vs CC BY 4.0)** and apply the
  conservative attribution if CC BY.
- **PMC open-access subset** — **per-article** license (CC BY / CC BY-NC / CC BY-NC-ND / CC0 / PD);
  facts extracted, verbatim text not redistributed.
- **Standard ontologies for grounding** — HGNC, MONDO/EFO, Sequence Ontology, GO, **ChEMBL**
  (note: **CC BY-SA** — share-alike obligations; see contamination control), identifiers.org CURIEs.

**Caveats we will not gloss over:** an "open" record can be wrapped in a more restrictive container, an
aggregator's license can differ from its upstream sources, and a database's license can change between
releases — so license + version are verified for the *specific release* used, not assumed. Each
allow-list entry records this analysis.

### Provenance model
- **Every assertion** links to its source at **passage/record-level** granularity (PMCID + location,
  or DB record ID + release), and records the source's license, the source **release version**, the
  extraction method, the **evidence type/level (ECO / source-native)**, and a confidence value.
- Un-sourced assertions are **never published** — enforced as a CI gate over the countable assertion
  unit fixed by the provenance-mechanism decision.
- Conflicting / superseded findings are retained side by side with their evidence levels and provenance.
- **Retraction/supersession** status of each cited publication is screened at every release.

### Privacy / PII stance
- **No patient data of any kind.** No individual-level, controlled-access, or identifiable data.
- Subjects of the graph are **molecular/biological entities and published findings**, not people.
- No collection of contributor PII beyond standard open-source attribution the contributor chooses to
  provide.

### Attribution & output license
- **Code:** MIT.
- **Graph/content:** **CC0** where derived purely from CC0/PD sources (CIViC, Open Targets, CC0/PD
  articles); **CC BY 4.0** where CC BY material (e.g. CC BY articles, CC BY Reactome) is incorporated;
  **CC BY-SA-aware segregation** for any ChEMBL-derived statements (share-alike) — prefer CC0/CC BY
  sources and **segregate or label** CC BY-SA-derived statements to avoid contaminating a CC0 dataset.
  Facts from CC BY-NC / CC BY-NC-ND articles are used as **non-copyrightable facts with citation**, not
  as redistributed text. Mixed derivations are labeled at the dataset and, where feasible, the
  assertion level.

### Conflict-of-interest rule (for-profit guardrail)
No contributor may curate therapeutic-evidence assertions for a drug, target, or program in which they
hold an **undisclosed financial interest**. Disclosure is required; the maintainer/expert reviewer may
recuse a contributor. This protects against the graph being steered to benefit a for-profit entity
(Hee-Lee Oss criterion 3).

## Quality, review & risk gates

**Risk tier: medium.** Three review dimensions, all required before a deed is "done":

1. **License/ToS review (primary gate).** Before any source is processed, a reviewer confirms the
   `sources/allowlist.yml` entry: source is open/openly-licensed, the *specific release* is cleared,
   PMC-OA articles are license-resolved per-article, and it is not COSMIC/OncoKB/controlled-access. No
   extraction may start against a `pending`/`rejected` source. Any task proposing to touch a non-open
   DB or patient-level data is **refused and flagged** per Hee-Lee Oss guardrails.
2. **Biomedical-accuracy / citation review (expert).** A **credentialed biomedical/oncology reviewer**
   (cancer biologist or bioinformatician with relevant domain expertise) samples extracted assertions
   against their cited open sources; mis-grounded entities, mis-citations, mis-stated evidence levels,
   or invented facts block sign-off. Conflicting findings must be represented, not flattened.
3. **"Not medical advice" framing review.** Every therapeutic-target assertion must carry the
   research-evidence/"not medical advice" label and a preclinical-vs-clinical evidence distinction.
   **Any drift toward patient-facing guidance escalates the item to risk-tier `high`** (oncologist +
   patient-advocate sign-off required before merge).

**Definition of Shipped (project level).** A published, openly-licensed, queryable knowledge graph
with: provenance on **100%** of assertions; ≥4 approved open sources; passing SHACL +
Biolink-conformance + provenance-completeness + not-advice-label CI; **100% of therapeutic assertions
labeled and evidence-leveled**; retraction screening run; a **credentialed expert reviewer engaged**;
and at least one research/advocacy **partner that has adopted or cited it**. Per Hee-Lee Oss, *delivered ≠
merged* — the data must actually be in a beneficiary's hands.

**Per-deed Definition of Done.** Acceptance criteria met + CI green (schema/SHACL/Biolink/provenance/
label) + license review passed + expert accuracy/citation review passed + (for therapeutic content)
not-advice framing reviewed + output published under the declared license.

## Roadmap & milestones

**M0 — Foundation & licensing spine (cold-start).**
Goal: establish the rails so no data work can bypass the license/provenance/label gates.
Exit criteria: (a) schema v0 published covering the core classes, **mapped to Biolink + standard
ontologies**; (b) `sources/allowlist.yml` schema defined with the 4 candidate sources analyzed and ≥1
`approved` (incl. PMC-OA per-article license policy + COSMIC/OncoKB/controlled-access hard-exclude);
(c) provenance mechanism decision ratified, **including the countable "assertion" unit** and the
**source-version manifest** format; (d) CI provenance-completeness + SHACL + Biolink-conformance +
not-advice-label lints scaffolded; (e) steward + expert-reviewer outreach started (status logged);
(f) a **qualified License/ToS reviewer named AND a credentialed biomedical/oncology accuracy reviewer
named** (hard exit; documented fallback if a seat is empty — M0 cannot exit and escalation begins);
(g) a **host-independent persistent identifier (w3id.org) committed** as the canonical IRI namespace.

**M1 — First sourced slice (proof of pipeline).**
Goal: end-to-end one batch from approved structured sources (and a license-checked PMC-OA prototype)
into the graph with full provenance.
**Hard entry precondition:** the exact source releases for the M1 batch are named and recorded as
license-cleared (CIViC/Open Targets CC0 verified; any PMC-OA article license resolved) before
extraction starts.
Exit criteria: (a) ≥1 structured source (e.g. CIViC EWSR1/FLI1 evidence) imported into
Gene/Fusion/Disease/EvidenceAssertion/SourceDocument nodes, plus a small license-checked PMC-OA
extraction prototype; (b) 100% of new assertions carry provenance + source release + evidence level
and pass CI; (c) **retraction screening** run on cited PMIDs; (d) **stratified** expert citation-review
audit (min sample; by source and method; independent auditor) ≥95% verified, and zero confirmed false
normalizations in the grounding sample; (e) KGX/Biolink + JSON-LD + Turtle export produced under the
persistent-identifier namespace with a source-version manifest; (f) ≥1 candidate steward + the expert
reviewer engaged. Depends on M0.

**M2 — Normalization, conflict & explorer (usable surface).**
Goal: make the graph grounded, conflict-faithful, linked, and browsable.
Exit criteria: (a) entity grounding to standard IDs with ≥95% coverage and **zero confirmed false
mappings** in audit; (b) human-reviewed conflict/duplicate-resolution workflow operational;
(c) public explorer + documented exports live, showing each assertion's source, evidence level, and
"not medical advice" label; (d) reuse metrics tracked. Depends on M1.

**M3 — Scale & partner adoption (shipped).**
Goal: broaden sources and lock in real-world use.
Exit criteria: (a) ≥4 source collections integrated and ≥2,000 sourced assertions, 100% provenance
maintained, 100% therapeutic assertions labeled/evidence-leveled, retraction screening current;
(b) ≥1 partner has **adopted or cited** the graph (Definition of Shipped met); (c) expert-reviewer
engagement sustained; (d) sustainability/source-refresh/reviewer-rotation plan in effect. Depends on M2.

## Work breakdown

The itemized, schema-mapped backlog lives in [`TASKS.md`](./TASKS.md), organized by the milestones
above (M0–M3) plus a sized backlog. Each task maps to a Hee-Lee Oss Task JSON and carries a type, size, risk
tier, deliverable, dependencies, and reviewer. M0 deliberately front-loads the licensing, provenance,
Biolink-alignment, and not-advice guardrails before any bulk extraction begins.

## Governance, roles & stakeholders

- **Maintainer / Owner:** TBD — accountable for scope, the licensing gate, the not-advice framing, and
  releases.
- **License/ToS reviewer:** TBD — must approve every `sources/allowlist.yml` entry (incl. PMC-OA
  per-article licenses); has veto over any source. **Naming a qualified person is a hard M0 exit
  criterion.** **Documented fallback if the seat is empty:** no source advances past `pending`, no
  extraction begins, M0 cannot exit; the maintainer escalates to Hee-Lee Oss governance/board (and may engage
  qualified pro-bono counsel) before any data work proceeds.
- **Credentialed biomedical/oncology accuracy reviewer (rotation):** cancer biologist /
  bioinformatician with relevant expertise; performs citation + grounding + evidence-level review.
  **Naming ≥1 is a hard M0 exit criterion** (mandatory for medium risk). **TO BE SECURED.**
- **High-risk escalation reviewers:** oncologist + patient-advocate — engaged **only if** any
  deliverable becomes patient-facing (which would re-tier the item to `high`).
- **Steward (last-mile owner):** the research/advocacy partner or KG consortium that adopts, hosts
  long-term, and cites the graph. **TO BE SECURED** — required for "shipped."
- **Partner / requestor:** Ewing/sarcoma researchers, bioinformaticians, advocate-research authors,
  downstream KG projects (diffuse beneficiary class); a named representative org is TO BE SECURED.
- **Conflict-of-interest steward:** maintainer enforces the COI/disclosure rule on therapeutic curation.
- **Hee-Lee Oss governance/board:** arbiter for edge cases (e.g., a borderline source or a re-tiering
  decision) under the published conflict-of-interest/veto checklist.

## Dependencies & integrations

- **CIViC** (CC0), **Open Targets Platform** (CC0), **Reactome** (verify release license) — structured
  evidence sources.
- **PMC open-access subset** — license-checked per article (intake for literature extraction).
- **Standard ontologies/vocabularies:** Biolink Model, HGNC, MONDO/EFO, Sequence Ontology, GO, ChEMBL
  (CC BY-SA), ECO/SEPIO, identifiers.org CURIEs.
- **PubMed retraction notices + Retraction Watch open dataset** — retraction screening.
- **SHACL tooling, an RDF library, a KGX/Biolink toolkit, a nanopublication library** (TypeScript/ESM
  ecosystem where available).
- **Hee-Lee Oss pieces:** Task schema (`packages/schema`), CLI workspace/PR flow (`packages/cli`,
  `packages/core`), governance proposal/registry process. **Donated lane** — humans run their own
  agents. Any future metered extraction run uses `packages/runner` on an API key with a **hard
  per-task budget cap** (`funded` lane; `fundedBudgetUsd` required).
- **Monarch Initiative / Translator** (optional, ideal) — downstream KG consumers / candidate stewards.

## Risks & mitigations

| Risk | Likelihood | Impact | Mitigation | Owner |
| --- | --- | --- | --- | --- |
| Contributor imports COSMIC/OncoKB or other non-open DB (incl. laundered via a fake open citation) | Medium | Critical (license, project-ending) | Hard out-of-scope rule; allow-list gate blocks un-approved sources; license-reviewer veto; CI rejects non-`approved` sources; anti-laundering controls (passage/record-level citations, contributor attestation, spot-checks for COSMIC/OncoKB-distinctive fields); refusal + flag | License reviewer |
| PMC-OA article is CC BY-NC/ND and verbatim text is redistributed | Medium | High | Per-article license resolution before reading; extract **facts not verbatim text**; dataset-license segregation; CI label/license check | License reviewer |
| Inadvertent inclusion of patient-level / controlled-access data | Low | Critical (privacy guardrail) | No-patient-data scope rule; controlled-access categorically excluded; review checklist | License + accuracy reviewers |
| Therapeutic content read as medical advice | Medium | High | Mandatory "research evidence — not medical advice" label + preclinical/clinical distinction on every therapeutic assertion; framing review; **escalate to `high` (oncologist+advocate) if patient-facing** | Maintainer + expert reviewer |
| Mis-grounded entities / false ID normalization corrupts the graph | Medium | High | Precision-over-recall grounding; explicit candidate-blocking; human-confirmed, reversible mappings; **zero-confirmed-false-mapping** audit target | Accuracy reviewer |
| Inaccurate extraction / mis-citation / invented biology | Medium | High | Credentialed-expert citation-review sampling; provenance-completeness CI; **assistive (not autonomous) extraction; flag-on-doubt; gaps stay gaps** | Accuracy reviewer |
| Cited finding is retracted/superseded | Medium | Medium | Retraction screening (PubMed + Retraction Watch) at every release; evidence-level + recency on assertions; flag/withhold retracted sources | Maintainer |
| Conflicting preclinical findings flattened into one "truth" | Medium | Medium | Conflict-preserving model with evidence levels; reviewers reject single-winner assertions lacking sourced rationale; conflict-faithful metric | Accuracy reviewer |
| CC BY-SA (ChEMBL) statements contaminate a CC0 dataset | Medium | Medium | Prefer CC0/CC BY sources; segregate/label CC BY-SA-derived statements; dataset-level license honesty | Maintainer |
| No steward/partner secured → "delivered ≠ merged" not met | High | High | Treat partner + expert-reviewer outreach as M0/M1 deliverables; log status honestly; multiple candidate stewards pursued | Maintainer |
| Source license/version changes between releases | Medium | Medium | Per-release license + version recorded in allow-list and manifest; re-verify on each refresh | License reviewer |
| Reviewer capacity exhausted / burnout (license + expert review bottleneck) | Medium | High | Sampling-based review (not 100% re-check); rotation with response-time SLA; documented throughput ceiling throttling intake | Maintainer |
| Curation steered to benefit a for-profit (drug program) | Low | High | COI disclosure rule; recusal; public-benefit criterion enforced | Maintainer |
| IRI/host instability breaks dereferenceable links | Medium | Medium | Host-independent w3id.org namespace committed in M0; redirect strategy | Maintainer |
| Scope creep into a general cancer KG or clinical tool | Medium | Medium | Explicit non-goals; governance review of scope changes | Maintainer |

## Security & privacy

- **Threat surface:** small (data/content project, no accounts, no user PII). Main risks are
  *compliance* (non-open/controlled-access sourcing), *data integrity* (false/unsourced/mis-grounded
  assertions), and *mis-framing* (therapeutic content read as advice) — addressed by the license,
  accuracy, and not-advice gates above.
- **Secrets handling:** no API keys are required for the public open sources; any API tokens
  (e.g. a rate-limited endpoint) stay out of logs, receipts, and commits per Hee-Lee Oss rules. The donated
  lane never runs headless or authenticates an agent. A funded run uses `packages/runner` with a hard
  budget cap and the same secret-hygiene rules.
- **PII:** none — no patient or controlled-access data; subjects are molecular entities and published
  findings. Contributor attribution is opt-in and minimal.
- **Abuse/misuse prevention:** the graph is source-linked so every claim is auditable; the project
  refuses and flags any attempt to ingest non-open-DB or patient-level data, or to repackage the graph
  as clinical guidance. No surveillance or person-level profiling is supported.

## Sustainability & maintenance

- **After delivery,** the maintainer plus the secured steward own ongoing curation; the steward
  provides long-term hosting, while the **canonical persistent identifier (w3id.org) is owned by the
  project, not the host** — so identifiers survive a steward change and the graph is never orphaned.
- **Source-refresh cadence:** a documented re-sync schedule (CIViC, Open Targets release N+1, Reactome
  vN+1, PMC-OA snapshot), each re-verifying license + version and re-running retraction screening; each
  release stamped with a source-version manifest and a staleness report.
- **Reviewer sustainability:** review runs on **sampling, not exhaustive re-checking**; reviewers work
  a **rotation with a response-time SLA**; a **documented throughput ceiling** throttles new extraction
  intake when the review backlog exceeds it, so quality gates never silently degrade under load.
- **Outcome tracking:** quarterly report on sourced-assertion growth, provenance completeness (must
  stay 100%), grounding coverage + false-mapping audit, citation-audit pass rate, label coverage,
  retraction screening, partner adoptions, and reuse/downloads.
- **Contributions** continue via the donated lane with the same license + expert + not-advice gates.
- **Versioned releases** (with changelogs + manifest) keep downstream consumers (Monarch/Translator)
  stable.
- If no steward is secured, the project remains in a maintained-but-not-shipped state and the gap is
  reported honestly rather than declared done.

## Open questions

- Who is the committed steward / adopting partner, and who is the credentialed expert reviewer? (Both
  TO BE SECURED — they block "shipped" and M0 exit respectively.)
- Final provenance mechanism: **nanopublications** vs named-graphs+PROV-O vs RDF-star? (The countable
  assertion unit is fixed by whichever is chosen — decided in M0.)
- Which exact Biolink Model version to pin for conformance, and how to track Biolink upgrades?
- Reactome's exact per-release license (CC0 vs CC BY 4.0) for the release used — to be confirmed in
  the allow-list.
- Default dataset license: CC0-only with CC BY / CC BY-SA segregated, or a single CC BY 4.0 dataset?
- How deep should PMC-OA literature extraction go in v1 vs. relying on structured sources (CIViC/Open
  Targets) first? (Literature extraction is the highest-effort, highest-license-risk component.)

## References

- Project roadmap entry: `planning/ROADMAP.md` (Track 8a — Ewing's Sarcoma; `ewsr1-fli1-knowledge-graph`)
- Hee-Lee Oss work rules: `CLAUDE.md`
- Good Deed Definition & risk tiers: `docs/good-deed-definition.md`
- Task JSON schema: `packages/schema/src/schemas.ts`
- CIViC (CC0); Open Targets Platform (CC0); Reactome; PMC open-access subset (per-article license)
- Biolink Model; HGNC; MONDO/EFO; Sequence Ontology; Gene Ontology; ChEMBL (CC BY-SA); ECO; SEPIO;
  identifiers.org
- PubMed retraction notices; Retraction Watch open dataset
- W3C: RDF, JSON-LD, SHACL, PROV-O, RDF-star; nanopublications; KGX/Biolink exchange format

---

## Appendix A — Improvements applied

The following 25 specific improvements were identified against the first draft of this plan and have
been **applied** to the body above (and to `TASKS.md`). They are retained here so the work is visible.

1. **Two named hard boundaries**, not one: (a) non-open oncology DBs (COSMIC/OncoKB/DrugBank) and
   (b) controlled-access / patient-level data (dbGaP/EGA/biobanks/EHR) — both categorically out of
   scope, mirroring the GRS/PRS pattern, with anti-laundering controls. *(Scope, Data §Hard boundaries)*
2. **PMC-OA per-article license verification** pinned as the subtle headline gate (CC BY vs
   NC/ND vs CC0/PD); extract structured facts, never redistribute verbatim text. *(Data §PMC nuance)*
3. **Adopt the Biolink Model** + standard ontologies + identifiers.org CURIEs as the alignment
   vocabulary instead of inventing one — for interoperability with Monarch/Translator. *(Architecture)*
4. **Reuse CIViC/SEPIO + ECO evidence model** for evidence typing/leveling rather than ad hoc fields.
   *(Architecture, Data §Provenance)*
5. **Nanopublications** added as the leading candidate provenance mechanism (alongside
   named-graphs+PROV-O / RDF-star), with the countable "assertion unit" fixed in M0. *(Key decisions)*
6. **Evidence level + preclinical-vs-clinical distinction + mandatory "not medical advice" label** on
   every therapeutic assertion, enforced by CI and review. *(Metrics, Quality gates, Architecture)*
7. **Retraction/supersession screening** (PubMed + Retraction Watch) as a release gate and ongoing
   maintenance task. *(Architecture stage 6, Metrics, Sustainability, TASKS retraction-001)*
8. **Entity-normalization precision target (zero confirmed false mappings)** + explicit
   candidate-blocking/grounding strategy, precision over recall. *(Metrics, Architecture stage 5)*
9. **Credentialed biomedical/oncology expert reviewer required** (hard M0 exit) + explicit
   **escalation-to-`high`** (oncologist + advocate) if anything becomes patient-facing. *(Governance)*
10. **Host-independent persistent IRI namespace (w3id.org)** committed in M0, decoupled from the
    unsecured steward. *(Key decisions, M0, Sustainability)*
11. **Conflict-faithful representation** of contradictory/superseded preclinical findings with
    evidence levels (no flattening). *(Goals, Metrics, Architecture)*
12. **Source-version manifest** in every release (CIViC snapshot, Open Targets release, Reactome
    version, PMC-OA snapshot) for reproducibility. *(Key decisions, Architecture, Sustainability)*
13. **CC BY-SA contamination control** for ChEMBL-derived statements (segregate/label; prefer
    CC0/CC BY). *(Data §Attribution, Risks)*
14. **Stratified citation-audit sampling frame** (min N=200, strata by source + method, independent
    auditor) to make the ≥95% accuracy metric meaningful. *(Metrics)*
15. **SHACL + Biolink-conformance + provenance-completeness + not-advice-label CI gates.**
    *(Architecture stage 7, Quality gates, TASKS ci-001)*
16. **Precise beneficiaries** named (Ewing researchers, bioinformaticians, advocate-research authors,
    downstream KG consortia) with steward + expert reviewer **TO BE SECURED**. *(Problem & beneficiaries)*
17. **Assistive-extraction, flag-on-doubt, never-invent** rule for LLM literature extraction; ambiguous
    fields go to human review. *(Architecture, Non-goals, Risks)*
18. **Biologically precise scope**: EWSR1-ETS *family* (FLI1, ERG, FEV, ETV1/4) + breakpoint subtypes
    (Type 1/2) via Sequence Ontology — not just "EWSR1-FLI1." *(Data model: Gene, Fusion, FusionBreakpoint)*
19. **Explicit non-goal**: not a clinical decision tool / trial-matcher / patient variant-interpretation
    service. *(Goals & non-goals, Scope)*
20. **Passage/record-level provenance granularity** (PMCID+location, or DB record+release) — never
    "cite the database as a whole." *(Data §Provenance, anti-laundering)*
21. **Reviewer-sustainability controls** (sampling, rotation+SLA, throughput ceiling throttling intake).
    *(Sustainability, Risks)*
22. **Conflict-of-interest rule** (no curating evidence for a drug/program you're financially tied to) —
    the for-profit guardrail made concrete. *(Data §COI, Governance, Risks)*
23. **Downstream-interoperability deliverable**: KGX/Biolink TSV + JSON-LD/Turtle + nanopub feed so
    Monarch/Translator can ingest. *(Architecture, Goals, TASKS export-001)*
24. **Currency/freshness SLA**: documented source-refresh cadence + per-release staleness report +
    re-verification of license/version. *(Sustainability, Open questions)*
25. **Funded-lane note**: any metered extraction run uses `packages/runner` with a hard per-task budget
    cap; default lane is donated. *(Dependencies, Security, TASKS mapping)*

## Review sign-off

A completeness/correctness review of the revised plan + tasks was performed against the Hee-Lee Oss rules,
the good-deed definition, and the PLAN_SPEC structure:

- **Structure:** all 17 required H2 sections present and in order; metadata header present; Appendix A
  documents the 25 applied improvements.
- **Metrics measurable?** Yes — each has a baseline + target; the ≥95% accuracy metric is pinned to a
  stated stratified sampling frame; "100% provenance," "100% not-advice label," "zero false mappings,"
  and "Biolink-conformance" are mechanically checkable CI/audit gates (not vanity counts).
- **Gates enforceable?** Yes — license gate (allow-list `approved` precondition), provenance gate (CI),
  Biolink-conformance gate (CI), not-advice-label gate (CI), expert citation/grounding audit, and
  retraction screening are each tied to a milestone exit criterion and a Definition of Done.
- **Risks have owners + mitigations?** Yes — every row in the risk table names a mitigation and an
  owner; the two project-ending risks (non-open-DB import; patient-data inclusion) have hard
  out-of-scope rules + anti-laundering controls.
- **Guardrails present?** License/provenance (open/PD/CC only, per-release + per-article), privacy/PII
  (no patient/controlled-access data), for-profit (COI rule + non-goal), and expert review (credentialed
  reviewer hard M0 exit; "not medical advice" label; escalation-to-`high` if patient-facing) are all
  explicit.
- **Sequencing sound?** Yes — M0 front-loads licensing + provenance + Biolink + label rails and names
  both reviewers before any extraction; M1 proves the pipeline on one approved source + a license-checked
  PMC-OA prototype; M2 normalizes + makes it browsable; M3 scales + secures adoption. Dependencies are
  stated.
- **Tasks schema-valid?** The example Task JSON validates against `packages/schema/src/schemas.ts` (all
  required fields present, enums respected, `verifiedNeed: false`, real `outputLicense`); funded tasks
  (none scheduled) would add `fundedBudgetUsd`.
- **Fixes applied during review:** confirmed `verifiedNeed: false` throughout (no partner secured);
  confirmed the credentialed-expert-reviewer seat is a *hard M0 exit* (not just "nice to have");
  confirmed every therapeutic-evidence path carries the not-advice label gate; confirmed Reactome and
  Open Targets licenses are marked "verify per-release" rather than assumed.

**Headline gate:** the project lives or dies on **per-source/per-article license verification (open/PD/
CC only; COSMIC/OncoKB and all patient/controlled-access data categorically excluded) plus
provenance-on-every-assertion and a credentialed-expert + "not medical advice" review** — all enforced
before M0 can exit and before any deed is "done." **Human decisions still required:** secure a steward,
name the License/ToS reviewer **and** the credentialed biomedical/oncology reviewer, and ratify the
provenance mechanism + Biolink version.
