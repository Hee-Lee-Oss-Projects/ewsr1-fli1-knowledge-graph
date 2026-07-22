# Competitive & Improvement Analysis — `ewsr1-fli1-knowledge-graph`

> Scope: an open, provenance-bearing knowledge graph (KG) of EWSR1-FLI1 / EWSR1-ETS fusion
> biology in Ewing sarcoma, synthesized **only** from open sources (CIViC, Open Targets, Reactome,
> PMC open-access), Biolink-aligned, no patient data. Analysis grounded with cited web sources.
> Author: analysis agent · Date: 2026-06-29

---

## 1. Correctness & completeness review of PLAN.md

The plan is unusually mature: it already names the hard licensing gates, picks Biolink + SEPIO/ECO,
adopts nanopublications as a candidate, sets precision-over-recall normalization with a zero-false-mapping
target, and front-loads the not-medical-advice framing. Appendix A shows 25 prior fixes were applied.
The findings below are the *residual* gaps after that hardening.

**A. Licensing / source-model issues**

1. **CIViC is itself a derived knowledgebase, not a primary source — a hidden citation-laundering vector
   the plan does not fully close.** CIViC content is CC0 ([CIViC NG paper](https://www.nature.com/articles/ng.3774);
   [CIViCdb 2022](https://academic.oup.com/nar/article/51/D1/D1230/6827107)), but each CIViC evidence
   item *cites a primary publication* that may be paywalled or non-open. If the KG ingests a CIViC
   assertion and re-cites that paywalled PMID as its "open source," the assertion's provenance points
   at a closed paper. The plan's passage-level citation rule needs an explicit sub-rule: when importing
   from an aggregator (CIViC/Open Targets), provenance must record **both** the aggregator record ID+release
   **and** the upstream citation, flagged by openness — not silently promote the upstream PMID to "open."

2. **Open Targets is an aggregator of >20 upstream sources with heterogeneous terms.** Open Targets
   *platform output* is CC0 1.0 ([Open Targets licence](https://platform-docs.opentargets.org/licence);
   [NAR 2025](https://academic.oup.com/nar/article/53/D1/D1467/7917960)), but it integrates evidence
   derived from sources whose own terms differ. The plan says "terms respected" but gives no mechanism:
   it should record the **Open Targets datasource** (e.g., `eva`, `cancer_gene_census` — which is
   COSMIC-derived!) per assertion and **exclude COSMIC/Cancer Gene Census-derived evidence channels**,
   or the project re-imports COSMIC by the back door through Open Targets. This is a concrete,
   project-ending gap the current text does not catch.

3. **ChEMBL CC BY-SA share-alike is treated as "segregate and label" — but share-alike is viral on
   *derivative datasets*, not just statements.** ChEMBL is CC BY-SA 3.0
   ([ChEMBL licensing](http://chembl.github.io/chembl-licensing/)). If TherapeuticAgent nodes are
   ChEMBL-derived and are *interlinked* with CC0 assertions in one graph, a downstream consumer may
   argue the integrated graph is a derivative and must be CC BY-SA — contaminating the CC0 promise.
   "Segregate or label" is under-specified: the plan needs a decision on whether ChEMBL IDs are used
   only as **external CURIE references** (arguably not a derivative) vs. whether ChEMBL *fields* (names,
   activities) are copied (clearly share-alike). Recommend: reference ChEMBL by CURIE only; never copy
   ChEMBL field values into the dataset.

4. **Reactome is genuinely dual-licensed and the plan's "verify per-release" is correct but needs to be
   sharper.** Reactome *data/annotation* is CC0; *pathway illustrations, diagrams, code, and database
   dumps* are CC BY 4.0 ([Reactome license](https://reactome.org/license/)). So importing pathway
   *membership facts* is CC0-clean, but reusing a Reactome **diagram** is CC BY. The plan should state
   it imports facts (CC0 path), not diagrams.

5. **OncoKB exclusion is correct and the plan should note the AI-training clause explicitly.** OncoKB is
   free for academic *web* use but requires a license for programmatic/commercial use, **and explicitly
   forbids using OncoKB to train AI/ML models** ([OncoKB licensing FAQ](https://faq.oncokb.org/licensing)).
   This strengthens the hard-exclude and is worth citing in the allow-list rationale, because a
   contributor might naively assume "free for academics" means usable.

**B. Ontology / modeling issues**

6. **Variant/fusion representation under Biolink is genuinely hard and under-specified.** Biolink's
   `Gene`/`Pathway` mapping is clean, but a **gene fusion** and a **breakpoint subtype (Type 1/Type 2)**
   do not have a first-class Biolink class. The plan invokes Sequence Ontology for breakpoints but does
   not say how a fusion maps to Biolink (`SequenceVariant`? `MolecularEntity`? a custom subclass?), nor
   how it will represent the *transcript-level* exon-junction precisely (GENCODE/Ensembl transcript IDs,
   HGVS, or the VICC/GA4GH **Variation Representation Specification (VRS)** for fusions). Without VRS or
   an explicit fusion model, "Type 1 vs Type 2" risks being a free-text label, undermining the
   "biologically precise" claim. **Recommend evaluating GA4GH VRS / VICC categorical variation as the
   fusion model**, which also eases reconciliation with CIViC (a VICC member).

7. **"Mechanism" is a custom class with no named ontology backing.** GGAA-microsatellite binding /
   pioneer-factor activity are real but the plan grounds Mechanism only loosely. GO molecular function /
   biological process, and the **Relation Ontology (RO)** for the predicate (e.g., `directly positively
   regulates`), should be specified, or "Mechanism" becomes an un-normalized free-text bucket — exactly
   the silo problem the project criticizes.

8. **EFO vs MONDO is listed as "MONDO/EFO" but they are not interchangeable.** Pick a canonical disease
   ID space (MONDO is the cross-walk hub; EFO is Open Targets' native) and map the other to it, or
   disease nodes will duplicate.

**C. Provenance / evidence-model issues**

9. **The "countable assertion unit" decision is correctly flagged as M0-blocking, but the three
   candidates have very different downstream costs that the plan doesn't weigh.** Nanopublications give
   per-assertion provenance and ecosystem fit (>10M life-science nanopubs exist;
   [nanopublications registry](https://kghub.org/kg-registry/resource/nanopublications/nanopublications.html)),
   **but Monarch/Translator ingest via KGX/Biolink, not nanopubs** ([Monarch KGX](https://monarch-initiative.github.io/monarch-documentation/Repositories/kgx/)).
   So the project likely needs **both** a nanopub feed *and* a KGX export with provenance squeezed into
   Biolink association slots (`primary_knowledge_source`, `publications`, `has_evidence`). The plan lists
   both exports but doesn't acknowledge that nanopub-native provenance is *richer* than what KGX edges
   carry, so a lossy mapping is unavoidable — that loss should be documented.

10. **Evidence-level harmonization across sources is asserted but not designed.** CIViC has its own
    evidence levels (A–E), Open Targets has association *scores* (0–1, not levels), Reactome has none,
    PMC extraction has none natively. Mapping these onto one ECO/SEPIO scale is a real modeling task
    with judgment calls; "preserve native level" and "harmonize to ECO" are in tension. The plan should
    say native levels are preserved **and** a derived, clearly-labeled harmonized tier is added — not
    conflated.

11. **Contradiction/conflict detection has a metric ("≥2 sourced variants retained") but no detection
    method.** Two assertions conflict only if they are about the *same* normalized subject-predicate-object
    with opposite polarity/effect. That requires the normalization to already be perfect — circular with
    the zero-false-mapping goal. Needs an explicit conflict-candidate generation step (same S+P+O,
    differing effect/direction/evidence) feeding the human reviewer.

**D. LLM-extraction risk (under-treated relative to its importance)**

12. **Hallucination / fabricated-citation risk is named ("never invent; flag on doubt") but lacks
    technical controls.** The single biggest failure mode for LLM literature extraction is a *plausible
    but unsupported* relation tied to a real PMCID — which passes a "has a citation" CI gate while being
    false. The plan's CI checks provenance *presence*, not provenance *faithfulness*. Needed:
    (a) **span-grounding** — the model must return the exact quoted passage offsets supporting the
    relation, verified to exist verbatim in the source text; (b) an automated **entailment / NLI check**
    (the claim must be entailed by the cited span) before it reaches a human; (c) a **per-extraction
    confidence** that gates into the audit sample. Without span-offset verification the 95% audit is the
    *only* defense and it samples just 200/release.

13. **No baseline against existing extractors.** INDRA already assembles mechanistic statements from
    text-mining + databases with a Bayesian **belief score** and de-duplication
    ([INDRA, Mol Sys Biol 2023](https://link.springer.com/article/10.15252/msb.202211325)). The plan
    reinvents relation extraction without saying why not to **reuse INDRA's reader outputs / belief
    model** as a prior. This is both a correctness gap (don't rebuild a worse wheel) and a competitive
    one (Section 2/5).

**E. Metrics / outcome issues**

14. **"≥2,000 assertions" risks being a vanity count the plan elsewhere disavows.** The plan rightly
    says raw counts aren't success, then sets a raw count target. Better: a *coverage* metric (e.g.,
    "≥X% of EWSR1-FLI1 direct target genes attested in CIViC/Open Targets are represented with
    provenance") tied to a defined reference set, so 2,000 trivially-true edges can't game it.

15. **No metric for the actual beneficiary outcome.** Per Hee-Lee Oss "delivered ≠ merged," the success metric
    that matters is *a researcher answered a question they couldn't before*. There's a partner-adoption
    metric but no **task-based usability evaluation** (e.g., N researchers complete N sourced-lookup
    tasks against the explorer). Recommend adding one.

16. **`verifiedNeed: false` is the dominant project risk and the plan is honest about it, but the
    roadmap still lets M1–M2 (heavy extraction build) proceed before a steward exists.** That risks
    building a technically perfect graph nobody adopts. Consider gating M2 scale-up on at least a
    *letter-of-intent* steward, not just "outreach started."

**Smaller gaps:** no mention of (a) **identifiers.org/Bioregistry** resolver dependency risk; (b) how
**superseded** (not retracted) findings are detected — Retraction Watch covers retractions, not quiet
obsolescence; (c) versioned **SSSOM mapping files** for the entity groundings (so mappings are auditable
and reversible, matching the "reversible mappings" risk mitigation); (d) a **SHACL-on-nanopublications**
validation story (SHACL over named graphs is non-trivial).

---

## 2. Competitive landscape

No open, provenance-first KG dedicated to EWSR1-ETS fusion biology appears to exist (a targeted search
for one returned only primary literature, not a database;
[search evidence](https://pmc.ncbi.nlm.nih.gov/articles/PMC7226175/)). The competition is therefore
**general biomedical KGs and cancer-variant knowledgebases** that *touch* Ewing sarcoma rather than
specialize in it.

**Source knowledgebases (also the project's inputs)**

- **CIViC** — crowd-curated clinical interpretation of cancer variants; **CC0**, uses a SEPIO-style
  evidence model. *Strengths:* gold-standard expert curation, fully open, evidence-leveled, VICC member.
  *Weakness for our scope:* organized around clinically actionable *variants*, thin on EWSR1-FLI1
  *mechanistic regulatory* biology (target genes, GGAA microsatellites); not a graph.
  [Nat Genet](https://www.nature.com/articles/ng.3774) · [CIViCdb 2022](https://academic.oup.com/nar/article/51/D1/D1230/6827107)
- **Open Targets Platform** — target–disease association KG over >20 sources, ~7.8M associations, GraphQL
  API; **CC0 1.0**. *Strengths:* huge, integrated, scored, truly open output, already a KG.
  *Weaknesses:* association *scores* not mechanistic provenance; aggregates COSMIC-derived channels
  (license trap); breadth-over-depth — Ewing is one disease among thousands, no fusion-breakpoint
  granularity. [Licence](https://platform-docs.opentargets.org/licence) · [NAR 2025](https://academic.oup.com/nar/article/53/D1/D1467/7917960)
- **OncoKB** — MSK precision-oncology knowledgebase, FDA-recognized. *Strengths:* authoritative
  actionability tiers. *Weaknesses / why excluded:* **not open** — academic web use free, but
  programmatic/commercial use needs a license and **AI/ML training is prohibited**
  ([licensing FAQ](https://faq.oncokb.org/licensing)). Categorically out of scope; also a *differentiator*
  (we are reusable; they are not).
- **COSMIC** — Catalogue of Somatic Mutations in Cancer (Sanger). *Strengths:* vast somatic mutation
  catalogue incl. fusions/Cancer Gene Census. *Weaknesses / why excluded:* **dual license — free to
  academics, commercial license otherwise**; not redistributable
  ([COSMIC/Sanger](https://www.sanger.ac.uk/group/cosmic-catalogue-of-somatic-mutations-in-cancer/);
  [Wikipedia](https://en.wikipedia.org/wiki/COSMIC_cancer_database)). Hard-excluded, including via
  Open Targets' Cancer Gene Census channel.
- **ClinVar** — NIH/NCBI variant–phenotype archive, **fully open/public domain**. *Strength:* open,
  authoritative germline/somatic interpretations. *Weakness:* clinical-significance focus, not
  fusion-mechanism biology; could be an *additional* open source rather than a competitor.

**General biomedical knowledge graphs**

- **Monarch Initiative** — the closest *methodological* peer and an ideal steward/consumer. Biolink-Model
  KG (~1.12M nodes / 8.9M edges), KGX exchange, **authors of SEPIO**, strong provenance discipline.
  *Strengths:* exactly the standards this project adopts; cross-species phenotype depth.
  *Weakness for our scope:* breadth-first, not Ewing/fusion-deep; mechanism of a single oncogenic fusion
  is below its granularity. [Monarch 2024](https://academic.oup.com/nar/article/52/D1/D938/7449493) ·
  [Biolink](https://arxiv.org/pdf/2203.13906) · [KGX](https://monarch-initiative.github.io/monarch-documentation/Repositories/kgx/)
- **Hetionet** — integrative hetnet (47k nodes / 2.25M edges from 29 resources) for drug repurposing;
  open, in Neo4j. *Strengths:* elegant metapath reasoning, fully open, reusable. *Weaknesses:* frozen
  v1.0 (2017), node/edge-type level not assertion-level provenance, no per-edge citation/evidence-level,
  no Ewing depth. [eLife 2017](https://elifesciences.org/articles/26726) · [GitHub](https://github.com/hetio/hetionet)
- **PrimeKG** — precision-medicine KG, 17,080 diseases / ~4M relationships from 20 resources, with
  drug indication/contraindication/off-label edges + text descriptions. *Strengths:* ML-ready, multimodal,
  disease-centric. *Weaknesses:* aggregated, **provenance is at the source-integration level, not a
  citable per-assertion record**; a rare fusion's regulatory mechanism is not its focus.
  [Sci Data 2023](https://www.nature.com/articles/s41597-023-01960-3) · [GitHub](https://github.com/mims-harvard/PrimeKG)
- **BioCypher** — not a KG but a **framework** for building Biolink/ontology-grounded KGs from adapters
  + schema config; outputs property-graph/RDF. *Strength:* could *build* this project faster than bespoke
  TypeScript; FAIR, reproducible, ontology-grounded. *Weakness:* Python ecosystem (vs. plan's TS/ESM
  mandate) and it doesn't solve the *content/curation*. Best treated as a **build-vs-buy** option, not a
  rival. [arXiv](https://arxiv.org/abs/2212.13543) · [GitHub](https://github.com/biocypher/biocypher)
- **INDRA / INDRA Database** — automated assembly of molecular mechanism *statements* from text-mining +
  curated DBs, with de-duplication and a **Bayesian belief score** and full provenance to each evidence.
  *Strengths:* directly solves mechanistic relation extraction at scale **with provenance and belief** —
  the hardest part of this project. *Weaknesses:* belief score ≠ curated evidence level; reader noise;
  not Ewing-curated; not Biolink-native. **Strongest functional overlap with the PMC-extraction lane.**
  [Mol Sys Biol 2023](https://link.springer.com/article/10.15252/msb.202211325)
- **Reactome** — curated pathways; dual CC0 (data) / CC BY (diagrams/dumps); an input, not a rival.
  [License](https://reactome.org/license/)
- **SIGNOR** — manually curated **causal** signaling interactions (effect + mechanism), **CC BY**, ~23k+
  relationships, 2025 v4.0. *Strengths:* causal directionality + mechanism, open, could be an *additional*
  open source for mechanism edges. *Weakness:* signaling-centric, not transcriptional-target-centric for
  a TF fusion. [SIGNOR 4.0](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12807704/)
- **Pharos / TCRD (IDG)** — druggable-genome target resource over 80+ sources, Target Development Levels;
  open. *Strengths:* target-centric, druggability annotation, open. *Weakness:* proteome-wide, not
  fusion-biology; aggregated provenance. [Pharos 2023](https://academic.oup.com/nar/article/51/D1/D1405/6851109)
- **DisGeNET** — gene–disease associations; **no longer fully open — moved to a freemium model in 2020,
  commercial tiers via MedBioInformatics**. *Strength:* large G–D coverage. *Weakness / caution:* license
  shift means it is **not a safe open source** for this project; cite as a cautionary tale of an "open"
  resource that closed. [Wikipedia](https://en.wikipedia.org/wiki/DisGeNET) · [reuse note](https://kghub.org/kg-registry/resource/disgenet/disgenet.html)
- **KEGG** — pathway maps; **academic free / commercial license required for bulk download** — not an
  open source for redistribution; excluded.

**Adjacent standards/infra:** GA4GH/VICC **VRS** (variant representation — relevant to fusion modeling),
**nanopublications** network (provenance interchange;
[registry](https://kghub.org/kg-registry/resource/nanopublications/nanopublications.html)), KG-Hub /
KGX (Biolink exchange).

**Net read:** every general KG is **broad and shallow** on Ewing and treats provenance at the
*source-integration* level; every deep cancer-variant KB is either **not graph-shaped** (CIViC) or
**not open** (OncoKB/COSMIC/DisGeNET/KEGG). The intersection "open + graph + assertion-level provenance +
EWSR1-ETS-deep" is **empty** — that is the wedge.

---

## 3. Gaps we can fill

1. **Rare-cancer depth where everyone else is broad.** Open Targets/Monarch/PrimeKG/Hetionet model
   thousands of diseases shallowly; none model EWSR1-FLI1 *target-gene regulation, GGAA-microsatellite
   pioneer mechanism, breakpoint subtypes, and research-stage targets* as a connected, queryable
   sub-graph. A small, deep, correct graph for one driver is genuinely novel.
2. **Assertion-level, citable provenance instead of source-level provenance.** Hetionet/PrimeKG tell you
   *which database* an edge came from; this project tells you *which passage of which open paper or which
   DB record+release* — and retains conflicting variants. That is the FAIR/nanopub promise actually
   delivered for one domain.
3. **An openly-licensed alternative to closed cancer KBs.** OncoKB, COSMIC, DisGeNET, KEGG all gate
   reuse and/or AI training. A CC0/CC BY graph that downstream projects (and ML pipelines) may freely
   reuse fills a real reusability gap for this driver.
4. **A clean fusion + breakpoint model** (if VRS/SO-based) that the broad KGs lack — fusions are usually
   collapsed to a gene or a free-text label elsewhere.
5. **Conflict-faithful mechanistic preclinical biology.** Most KGs flatten to one edge; preclinical Ewing
   biology is full of contested findings (e.g., direct vs. indirect targets) that deserve side-by-side
   representation with evidence levels.
6. **Ready-to-ingest interoperability for the open KG commons** — KGX/Biolink + nanopub feed so
   Monarch/Translator can absorb a rare-disease module they currently lack the depth to build.

---

## 4. Differentiators to win

1. **"A citation on every link" — assertion-level provenance + source release version as a hard,
   CI-enforced gate.** Not aspirational FAIR; mechanically checked. No major competitor enforces this.
2. **Open-only, AI-reusable license posture** as a deliberate contrast to OncoKB/COSMIC/DisGeNET/KEGG —
   explicitly usable for model training (the thing OncoKB forbids).
3. **Domain depth + correctness over scale** — precision-over-recall, zero-confirmed-false-mappings,
   independent expert audit. The graph competes on *trust*, not node count.
4. **Conflict-preserving evidence model** (SEPIO/ECO) — shows the science as contested where it is,
   a differentiator vs. single-truth KGs.
5. **Standards-native from day one** (Biolink + KGX + nanopublications + MONDO/HGNC/SO/ChEMBL CURIEs +
   w3id persistent IRIs) so it slots into Monarch/Translator rather than being a dead-end silo.
6. **Governance as a feature** — anti-laundering controls, COI rule, not-medical-advice gate, named
   license + accuracy reviewers. For a beneficiary deciding whether to *trust and adopt*, this is the moat.
7. **A reusable template** (Section 7) — the first instance of a generalizable "open, provenance-first
   fusion-oncogene KG" pattern.

---

## 5. Claude API leverage

**Where Claude is a strong fit (assistive, human-gated):**

1. **Relation extraction from PMC-OA literature into structured S–P–O candidates** — Claude reads a
   license-cleared open-access passage and emits typed candidate assertions (subject gene → predicate
   `regulates`/`directly positively regulates` → object gene/pathway, evidence type) **with the exact
   supporting quoted span and character offsets**. This is the core multiplier and the highest-value use.
2. **Entity normalization / grounding suggestions** — propose HGNC/Ensembl/MONDO/ChEBI/Reactome/SO
   candidate IDs for a mention, *with alternatives and a confidence*, feeding the candidate-blocking +
   human-confirm workflow (never auto-commit). Good at disambiguating "FLI1 the gene" vs. "FLI1 in
   EWSR1-FLI1 the fusion."
3. **Claim/evidence structuring into the SEPIO/ECO + nanopublication shape** — turn a messy sentence into
   a well-formed EvidenceAssertion (evidence type, preclinical-vs-clinical, polarity) ready for review;
   draft the not-medical-advice framing for therapeutic items.
4. **Contradiction / conflict candidate detection** — given two normalized assertions about the same
   S–P–O, classify whether they agree, conflict, or one supersedes the other, and surface the pair to a
   human (an NLI/entailment-style task Claude does well).
5. **License triage assistance** — parse a PMC article's license field/front-matter and *propose*
   CC BY / NC / ND / CC0 / PD, flagging ambiguity — as a first pass before the human License reviewer.
6. **Faithfulness pre-check** — before a candidate reaches the audit queue, have Claude verify the
   extracted claim is *entailed by* the quoted span (catch its own/another reader's hallucinations).
   Prompt-cache the schema + ontology context; structured tool-use/JSON-schema output for typed records.

**Where Claude must NOT decide (hard human/curator gates):**

- **No asserted biological fact ships on Claude's say-so.** Every assertion needs resolvable provenance
  to an open source **and** credentialed-expert citation review. Claude proposes; the curator disposes.
- **Never invent or "fill in" biology.** Gaps stay gaps; "flag on doubt, never guess" is binding on the
  model. A relation with no verbatim supporting span is rejected, not softened.
- **Claude does not make final entity groundings** — a wrong ID silently corrupts the graph; mappings are
  human-confirmed and reversible (SSSOM), targeting zero confirmed false mappings.
- **Claude does not make license determinations** — its triage is advisory; the named License/ToS
  reviewer makes the binding call, especially CC BY-NC/ND and aggregator-channel edge cases.
- **Claude does not assign final evidence levels or set the not-medical-advice posture** — those are
  expert-reviewer / governance decisions; Claude may draft, a human signs off.
- **No patient-facing or clinical inference, ever** — out of scope regardless of model capability.

This split keeps Claude on the *throughput* problem (reading, structuring, proposing at scale) and humans
on the *truth and rights* problems (is it real, is it ours to use, is it safe) — matching the plan's
"assistive, not autonomous" rule.

---

## 6. Ten concrete optimizations

1. **Reuse INDRA reader output + belief scores as a prior** for the PMC-extraction lane instead of
   building relation extraction from scratch; treat INDRA statements as candidate assertions to be
   re-grounded to Biolink and human-reviewed. Faster, and benchmarkable.
2. **Add a span-offset + verbatim-quote requirement and an automated entailment check** to the extraction
   CI gate, so "has a citation" becomes "has a *verified-supporting* citation" — the single biggest
   defense against fabricated-but-cited claims.
3. **Record aggregator provenance two-tier** (aggregator record+release AND upstream citation, each
   openness-flagged) and **exclude Open Targets' COSMIC/Cancer Gene Census-derived evidence channels** to
   plug the back-door COSMIC re-import.
4. **Adopt GA4GH/VICC VRS (or VICC categorical variation) for the fusion/breakpoint model** so Type 1/2
   are machine-precise and reconcilable with CIViC — not free text.
5. **Use SSSOM mapping files** for every entity grounding, versioned and reviewer-signed, making the
   "reversible, human-confirmed mappings" mitigation real and auditable.
6. **Replace the raw "2,000 assertions" target with a coverage metric** against a defined reference set
   of EWSR1-FLI1 target genes/mechanisms, plus a small **task-based usability eval** with real researchers.
7. **Reference ChEMBL by CURIE only; never copy ChEMBL field values** — sidesteps CC BY-SA share-alike
   contamination of the CC0 core cleanly.
8. **Evaluate BioCypher as the build substrate** (adapters + Biolink schema config + KGX output) before
   committing to bespoke TypeScript; if the TS/ESM mandate holds, at least mirror BioCypher's
   adapter/schema separation pattern.
9. **Ship a public SPARQL/GraphQL endpoint + a worked "answer this question" demo notebook** alongside the
   static explorer, so adoption (the `verifiedNeed` risk) has a concrete on-ramp for the steward.
10. **Add supersession detection beyond Retraction Watch** — track newer assertions about the same S–P–O
    and an article-recency signal, so quietly-obsoleted (not formally retracted) findings are flagged for
    review.

---

## 7. Parallel & perpendicular spin-offs

**Direct ties named in the roadmap:**

- **`ewing-open-data-catalog`** — the allow-list + per-source/per-article license determinations + source
  -version manifests are reusable as a standalone, citable *catalog of open Ewing data sources*. Ship the
  `sources/allowlist.yml` machinery as its own deliverable; it's valuable even before the graph is full.
- **`ewing-biomarker-evidence-cards`** — render each EvidenceAssertion (with provenance, evidence level,
  not-advice label) as a human-readable "evidence card" for advocate-research authors and science
  communicators — a non-patient-facing translation layer over the same graph.
- **`ewing-drug-target-evidence`** — a focused view over the TherapeuticAgent + Mechanism subgraph
  (research-stage targets, preclinical-vs-clinical, conflict-preserved) for the drug-target audience;
  reuses the same provenance/label gates.
- **`oncogene-knowledge-graph`** — promote the schema (Gene/Fusion/FusionBreakpoint/Mechanism/Pathway/
  Disease/EvidenceAssertion/SourceDocument) to a **generalized fusion-oncogene KG template** instantiable
  for other fusion-driven cancers (synovial sarcoma SS18-SSX, alveolar rhabdomyosarcoma PAX3-FOXO1,
  DSRCT EWSR1-WT1, fusion-positive leukemias). The pipeline is the asset; Ewing is instance #1.

**Multiplier / perpendicular ideas:**

- **An MCP server for researchers** exposing the graph as tools (`find_assertions(subject, predicate)`,
  `get_provenance(assertion_id)`, `list_conflicts(entity)`, `export_kgx(subgraph)`) so any
  Claude/agent user can query the provenance-bearing KG in natural language — directly leverages the
  JSON-LD/SPARQL surface and is a low-cost, high-visibility adoption driver.
- **A reusable "provenance-first extraction" library** (span-grounding + entailment-check + nanopub
  emission) usable by any Hee-Lee Oss data/content deed — the anti-hallucination harness generalized.
- **A license-triage microservice** (PMC-OA per-article license resolver + aggregator-channel openness
  classifier) reusable across all open-data Hee-Lee Oss projects.
- **A "rare-cancer KG starter kit"** — template repo + BioCypher/KGX config + CI gates (SHACL, Biolink,
  provenance, not-advice) that a sarcoma/rare-disease lab can fork to bootstrap their own open KG.
- **A nanopublication contribution channel** letting domain experts publish single, citable assertions
  that flow into the graph — turning the graph into community infrastructure (CIViC's crowdsourcing model,
  applied to mechanism).

---

## 8. Open questions for the maintainer

1. **Steward first or build first?** `verifiedNeed: false` is the dominant risk. Will you gate M2 scale-up
   on at least a letter-of-intent steward (e.g., Monarch/Translator, a Ewing foundation research program),
   rather than only "outreach started"?
2. **Fusion model:** will you adopt GA4GH/VICC **VRS** for fusions/breakpoints, or a custom SO-based model?
   (Determines reconciliation with CIViC and precision of the Type 1/2 claim.)
3. **Build substrate:** bespoke TypeScript/ESM (per Hee-Lee Oss convention) vs. **BioCypher** (Python, faster,
   Biolink-native) — is the TS mandate worth the rebuild cost for a data/content project?
4. **Reuse INDRA** for mechanistic extraction, or build relation extraction in-house? (Big effort/risk
   lever; INDRA already has provenance + belief.)
5. **Provenance interchange:** nanopublications as the canonical assertion unit means a *lossy* mapping to
   KGX/Biolink for Monarch/Translator ingest — is that loss acceptable, or should KGX be canonical and
   nanopubs a derived feed?
6. **Aggregator channels:** will you exclude Open Targets' COSMIC/Cancer Gene Census-derived evidence
   channels explicitly, and record upstream-vs-aggregator openness per assertion?
7. **Evidence-level harmonization:** preserve native levels only, or also publish a derived ECO-harmonized
   tier (clearly labeled)? How do Open Targets *scores* (0–1) map onto CIViC A–E?
8. **Default dataset license:** CC0 core with CC BY / CC BY-SA segregated, or one CC BY 4.0 dataset for
   simplicity? (Affects ChEMBL handling and downstream reuse friction.)
9. **Credentialed reviewer pipeline:** who, and at what throughput? The 200-assertion stratified audit per
   release plus conflict review is the real rate-limiter at scale.
10. **Beneficiary-outcome metric:** beyond "≥1 partner cited it," what *task* will a researcher complete
    against the graph that proves delivery (not merge)?
