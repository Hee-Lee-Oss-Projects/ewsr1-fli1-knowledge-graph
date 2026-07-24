# Provenance Mechanism Specification
## ewsr1-fli1-knowledge-graph-prov-001

> **Document:** Provenance mechanism ratification + assertion-unit definition + source-version manifest specification  
> **Version:** 1.0  
> **Date:** 2026-07-24  
> **Status:** Ratified  
> **Lane:** Donated · **Risk Tier:** Low  

---

## Executive Summary

This specification ratifies **nanopublications** as the uniform provenance mechanism for the EWSR1-ETS knowledge graph, defines the countable "assertion" unit as a **nanopublication core graph** plus its provenance and evidence metadata, and specifies a **source-version manifest** format to ensure reproducibility of every assertion across release boundaries.

**Key decisions:**
- **Provenance mechanism:** Nanopublications (W3C standard, purpose-built for assertion+provenance, interoperable with open KG ecosystem)
- **Assertion unit:** A nanopublication's core triple (subject–predicate–object) + its associated provenance nanopublication (source, release version, method, evidence level)
- **CI gate:** 100%-provenance is mechanically checkable: every core nanopublication **must** have a valid provenance record linking to an approved source
- **Reproducibility:** Every release includes a timestamped source-version manifest recording the exact snapshot date or release version used for each source

---

## 1. Rationale: Choice of Nanopublications

Three provenance mechanisms were evaluated:

| Mechanism | Strengths | Weaknesses | Verdict |
|-----------|-----------|-----------|---------|
| **Nanopublications** | Purpose-built for biomedical assertion+provenance; lightweight unit of publication; interoperable with Linked Data and open KG ecosystems (Monarch, Translator); assertion == countable unit; W3C standardized | Requires a triplestore or specialized library; fewer tools than plain RDF | ✅ **CHOSEN** |
| **Named graphs + PROV-O** | Mature RDF standard; flexible reified statements; strong event/agent modeling | Overly complex for this use case; PROV-O entities add 3–5x the triple count; assertion unit ambiguous (which graph? which PROV-O entity?) | ⚠️ Fallback |
| **RDF-star (embedded triples)** | Inline provenance; avoids reification overhead; modern syntax | Lacks ecosystem maturity; fewer tools/libraries; assertion unit still needs clarification | ❌ Not chosen |

**Nanopublications win because they:**
1. **Are the countable unit themselves** — a core triple + provenance triple = one nanopublication = one (checkable) assertion
2. **Are built for biomedical data** — originate in biomedical KG projects (FAIR Linked Data); align with CIViC/SEPIO evidence model
3. **Are interoperable** — Linked Data; JSON-LD serializable; queryable in SPARQL; feed nanopublication networks
4. **Scale assertions independently** — each assertion is a self-contained publication, not embedded in a larger graph that must be validated in bulk
5. **Decouple from the graph's IRI host** — nanopublications carry persistent identifiers (w3id.org) and can be republished, versioned, and reused without losing provenance

---

## 2. Assertion Unit Definition

### 2.1 The Nanopublication as a Countable Assertion

An **assertion** in this graph is a single **nanopublication** consisting of:

1. **Head graph (assertion URI)** — identifies the nanopublication  
   - IRI: `https://w3id.org/ewsr1-fli1-kg/np/{uuid}` (host-independent persistent identifier)

2. **Assertion graph** (required) — the core claim  
   - Single RDF triple: `(subject predicate object)`
   - Example: `ewsr1-fli1:EWSR1-FLI1 biolink:transcription_regulation_of ewsr1-fli1:GABPA`
   - Subject/predicate/object grounded to standard ontologies (HGNC, MONDO, ChEMBL, etc.; via identifiers.org CURIEs)

3. **Provenance graph** (required; **checkable by CI**)  
   - Specifies the **source** (CIViC record ID, Open Targets association ID, PMCID+location, Reactome pathway ID)
   - Records the **source license** (CC0, CC BY, CC BY-NC, etc.; per-article for PMC-OA)
   - Records the **source release version** (CIViC snapshot date, Open Targets release #, Reactome version, PMC-OA snapshot date)
   - Records the **extraction method** (structured import vs. literature extraction vs. manual curation)
   - Records the **evidence type/level** (ECO term; CIViC native level; Open Targets score; Reactome confidence)
   - Records the **confidence** (1.0 for structured sources; lower for assistive extraction)
   - Carries an **"approved source"** statement: the source must be in `sources/allowlist.yml` with `status: approved`

4. **Publication info graph** (optional)  
   - Creation timestamp (ISO 8601)
   - Creator attribution (contributor name/URI)
   - Version / supersession history (if this assertion updates or retracts a prior one)

### 2.2 Nanopublication Serialization

Assertions are serialized in **N-Quads** (one assertion per file in `data/assertions/`) and as a **JSON-LD** bundle in exports:

```nquads
# Example nanopublication in N-Quads format
<https://w3id.org/ewsr1-fli1-kg/np/a1b2c3d4> <http://www.w3.org/1999/02/22-rdf-syntax-ns#type> <http://purl.org/nanopub/core#Nanopublication> .
<https://w3id.org/ewsr1-fli1-kg/np/a1b2c3d4> <http://purl.org/nanopub/core#hasAssertion> <https://w3id.org/ewsr1-fli1-kg/np/a1b2c3d4#assertion> .
<https://w3id.org/ewsr1-fli1-kg/np/a1b2c3d4> <http://purl.org/nanopub/core#hasProvenance> <https://w3id.org/ewsr1-fli1-kg/np/a1b2c3d4#provenance> .
<https://w3id.org/ewsr1-fli1-kg/np/a1b2c3d4> <http://purl.org/nanopub/core#hasPublicationInfo> <https://w3id.org/ewsr1-fli1-kg/np/a1b2c3d4#pubinfo> .

# Assertion (core triple)
<https://w3id.org/ewsr1-fli1-kg/np/a1b2c3d4#assertion> {
  <http://identifiers.org/hgnc/EWSR1> <http://purl.obolibrary.org/obo/biolink_transcription_regulation_of> <http://identifiers.org/hgnc/GABPA> .
}

# Provenance
<https://w3id.org/ewsr1-fli1-kg/np/a1b2c3d4#provenance> {
  <https://w3id.org/ewsr1-fli1-kg/np/a1b2c3d4#assertion> <http://purl.org/prov#wasDerivedFrom> <civicdb.org/evidence/123> .
  <civicdb.org/evidence/123> <http://purl.org/dc/terms/isPartOf> "CIViC snapshot 2026-06-15" .
  <civicdb.org/evidence/123> <http://purl.org/dc/terms/license> <http://creativecommons.org/publicdomain/zero/1.0/> .
  <https://w3id.org/ewsr1-fli1-kg/np/a1b2c3d4#assertion> <http://purl.obolibrary.org/obo/SEPIO_0000001> <http://purl.obolibrary.org/obo/ECO_0000007> .
  <civicdb.org/evidence/123> <http://purl.org/dc/terms/conformsTo> <http://purl.org/nanopub/core#approved> .
}

# Publication info
<https://w3id.org/ewsr1-fli1-kg/np/a1b2c3d4#pubinfo> {
  <https://w3id.org/ewsr1-fli1-kg/np/a1b2c3d4> <http://purl.org/dc/terms/created> "2026-07-01T14:32:00Z"^^<http://www.w3.org/2001/XMLSchema#dateTime> .
  <https://w3id.org/ewsr1-fli1-kg/np/a1b2c3d4> <http://purl.org/dc/terms/creator> <https://orcid.org/0000-0000-0000-0000> .
}
```

### 2.3 CI Gate: Provenance Completeness

Every **assertion** (nanopublication) is validated by CI:

1. **Does it have a provenance graph?** → Fail if missing.
2. **Does the provenance link to an approved source?** → Check `sources/allowlist.yml`; fail if source is `pending` or `rejected`.
3. **Does the provenance record the source release version?** → Fail if missing.
4. **Is the evidence type or level recorded?** → Fail if missing.
5. **For therapeutic-target assertions, is the "not medical advice" label present?** → Fail if missing.

If any check fails, the assertion is **withheld from export**; the CI run is **red**; no release can proceed.

### 2.4 Assertion Unit Examples

**Example 1 — Structured import (CIViC)**
```
Assertion: EWSR1-FLI1 → regulates → TP53 (TP53 stabilization in response to EWSR1-FLI1 binding)
Provenance:
  - Source: CIViC evidence record #2847 (CIViC Release 2026-06-15)
  - License: CC0
  - Evidence level: (CIViC native: "B — Clinical evidence")
  - Extraction method: structured import
  - Confidence: 1.0
  - Approved: yes
```

**Example 2 — Literature extraction (PMC open-access)**
```
Assertion: EWSR1-FLI1 → pioneer_transcription_factor_binding → GGAA_microsatellite
Provenance:
  - Source: PMCID 3456789, Figure 2B, section "Mechanisms of gene regulation"
  - License: CC BY (per-article verification)
  - Evidence level: ECO_0000007 (direct assay)
  - Extraction method: literature extraction (assistive)
  - Confidence: 0.85 (flagged for human review)
  - Approved: yes (article in PMC-OA approved list)
```

**Example 3 — Conflicting finding (preserved side-by-side)**
```
Assertion A: EWSR1-FLI1 → regulates → CCND1 (activation)
Provenance A:
  - Source: Open Targets disease-gene link; PMID: 12345678
  - Release: Open Targets Platform 2026Q2
  - Evidence level: Open Targets score 0.8
  - Confidence: 1.0

Assertion B: EWSR1-FLI1 → regulates → CCND1 (repression)
Provenance B:
  - Source: PMID: 87654321 (different experimental context)
  - Release: PMC-OA snapshot 2026-06-15
  - Evidence level: ECO_0000007
  - Confidence: 0.9
  - Both assertions published; their contradiction is **auditable via provenance**.
```

---

## 3. Source-Version Manifest Format

### 3.1 Purpose

Every release of the knowledge graph includes a **source-version manifest** that records the exact snapshot or release version of each data source used. This enables **full reproducibility**: any downstream consumer can regenerate the same assertions by using the same source versions.

### 3.2 Manifest Structure (YAML)

The manifest is stored at `data/sources/manifest.yml` and versioned with the release:

```yaml
# Source-Version Manifest for ewsr1-fli1-knowledge-graph
# Release: v0.1.0
# Generated: 2026-07-15T10:30:00Z
# Purpose: Records the exact version of each source used in this release for full reproducibility

sources:
  civic:
    source_name: "CIViC: Clinical Interpretation of Variants in Cancer"
    custodian: "Alex H. Wagner et al. / Washington University School of Medicine"
    url: "https://civicdb.org/"
    api_endpoint: "https://civicdb.org/api/graphql"
    
    # Snapshot identifier: the exact date the data was downloaded or the API was queried
    snapshot_date: "2026-06-15T23:59:59Z"
    snapshot_identifier: "civic-2026-06-15"
    
    # License as of this snapshot
    license: "CC0"
    license_url: "https://creativecommons.org/publicdomain/zero/1.0/"
    license_verified_date: "2026-06-15"
    
    # Records included
    records_included:
      - type: "evidence"
        count: 127
        filter: 'genes IN ("EWSR1", "FLI1", "ERG", "FEV", "ETV1", "ETV4")'
    
    # Extraction method
    extraction_method: "structured_import"
    extraction_tool: "ewsr1-fli1-kg-import-civic v1.0"
    
    # Checksum of the downloaded dataset (for integrity)
    checksum_sha256: "abc123def456abc123def456abc123def456abc123def456abc123def456abc1"
    
    # Notes
    notes: "CIViC evidence for EWSR1-ETS fusions, including breakpoint subtypes and Type 1/2 variants."

  open_targets:
    source_name: "Open Targets Platform"
    custodian: "Open Targets Collaboration (EMBL-EBI / Wellcome Trust Sanger)"
    url: "https://platform.opentargets.org/"
    api_endpoint: "https://platform-api.opentargets.org/v6/"
    
    # Release version (not snapshot date; Open Targets versioned by release)
    release_version: "24.06"
    release_date: "2026-06-21"
    release_notes_url: "https://platform.opentargets.org/releases/24.06"
    
    # License
    license: "CC0"
    license_url: "https://creativecommons.org/publicdomain/zero/1.0/"
    license_verified_date: "2026-06-21"
    
    # Records included
    records_included:
      - type: "target_disease_associations"
        count: 43
        filter: 'target_symbol IN ("EWSR1", "FLI1", "ERG", "FEV", "ETV1", "ETV4")'
    
    # Extraction method
    extraction_method: "structured_import"
    extraction_tool: "ewsr1-fli1-kg-import-opentargets v1.0"
    
    # Checksum
    checksum_sha256: "def456abc123def456abc123def456abc123def456abc123def456abc123def456"
    
    # Notes
    notes: "Target–disease associations; filtered for EWSR1-ETS family genes and Ewing sarcoma disease ontology nodes."

  reactome:
    source_name: "Reactome: Pathway Knowledgebase"
    custodian: "Reactome Project / EBI"
    url: "https://reactome.org/"
    api_endpoint: "https://reactome.org/ReactomeRESTfulAPI/RESTful/"
    
    # Release version
    release_version: "90"
    release_date: "2026-06-12"
    release_notes_url: "https://reactome.org/eLearning/news/89-release/"
    
    # License: Reactome can vary by release (CC0 vs CC BY 4.0); verify per-release
    license: "CC0"
    license_url: "https://creativecommons.org/publicdomain/zero/1.0/"
    license_verified_date: "2026-06-12"
    
    # Records included
    records_included:
      - type: "pathways"
        count: 18
        filter: 'pathways containing EWSR1-FLI1 or EWSR1-ERG or regulated by fusion gene partners'
    
    # Extraction method
    extraction_method: "structured_import"
    extraction_tool: "ewsr1-fli1-kg-import-reactome v1.0"
    
    # Checksum
    checksum_sha256: "ghi789jkl012ghi789jkl012ghi789jkl012ghi789jkl012ghi789jkl012ghi7"
    
    # Notes
    notes: "Pathways involving EWSR1-ETS partners and their known targets; pathway cross-references validated."

  pmc_open_access:
    source_name: "PubMed Central Open-Access Subset"
    custodian: "National Library of Medicine / NCBI"
    url: "https://www.ncbi.nlm.nih.gov/pmc/"
    
    # Snapshot: PMC-OA is a rolling collection; record the export date
    snapshot_date: "2026-06-15T12:00:00Z"
    snapshot_identifier: "pmc-oa-2026-06-15"
    
    # License: PMC-OA is per-article; the manifest records the approval status and aggregate breakdown
    license: "per-article (CC BY, CC BY-NC, CC BY-NC-ND, CC0, Public Domain)"
    license_summary:
      cc_by: 34
      cc_by_nc: 2
      cc0: 8
      public_domain: 1
      total_articles: 45
    license_verified_date: "2026-06-15"
    
    # Records included
    records_included:
      - type: "articles"
        count: 45
        filter: 'search terms: EWSR1-FLI1 OR EWSR1-ERG OR Ewing sarcoma; published 2000–2026'
    
    # Extraction method
    extraction_method: "assistive_literature_extraction"
    extraction_tool: "ewsr1-fli1-kg-extract-pmc v1.0"
    llm_model: "claude-3-sonnet-20240229"
    
    # Extraction policy
    extraction_policy:
      rule: "structured_facts_with_passage_citations"
      verbatim_redistribution: "prohibited"
      human_review: "all_extractions_flagged_for_review"
    
    # Checksums (one per article to ensure integrity)
    checksums_by_pmcid:
      PMC3456789: "abc123def456..."
      PMC3456790: "def456abc123..."
    
    # Notes
    notes: "45 PMC-OA articles on EWSR1-ETS fusion biology; each article's license verified before extraction; structured facts extracted with passage-level citations; no verbatim copyrighted text redistributed."

  standard_ontologies:
    description: "Standard ontologies and vocabularies used for entity grounding"
    sources:
      - name: "HGNC: HUGO Gene Nomenclature Committee"
        version: "HGNC release 2026-06-01"
        license: "CC BY 4.0"
        role: "gene_identifier_grounding"
      
      - name: "MONDO: Mondo Disease Ontology"
        version: "2026-06-15"
        license: "CC0"
        role: "disease_identifier_grounding"
      
      - name: "Sequence Ontology (SO)"
        version: "2024-01-30"
        license: "CC0"
        role: "sequence_feature_type_grounding"
      
      - name: "Reactome"
        version: "90"
        license: "CC0"
        role: "pathway_identifier_grounding"
      
      - name: "ChEMBL"
        version: "ChEMBL 34"
        license: "CC BY-SA 4.0"
        role: "compound_identifier_grounding"
        note: "CC BY-SA statements segregated in exports to avoid contaminating CC0 dataset"
      
      - name: "ECO: Evidence & Conclusion Ontology"
        version: "2026-02-15"
        license: "CC BY 4.0"
        role: "evidence_type_classification"
      
      - name: "Biolink Model"
        version: "4.2.0"
        license: "CC0"
        url: "https://biolink.github.io/biolink-model/"
        role: "semantic_mapping_standard"

# Validation and screening gates applied before release
validation:
  retraction_screening:
    method: "PubMed retraction notices + Retraction Watch dataset"
    date_screened: "2026-06-20"
    retracted_pmids_withheld: []
    flagged_pmids_annotated: []
    notes: "No retractions found; all cited publications remain in good standing."
  
  provenance_completeness_check:
    assertions_checked: 342
    assertions_with_provenance: 342
    coverage: "100%"
    notes: "Every assertion in this release carries a resolvable provenance record."
  
  biolink_conformance:
    edges_validated: 342
    edges_compliant: 342
    compliance_rate: "100%"
    biolink_version_used: "4.2.0"
    notes: "All exported edges conform to Biolink Model v4.2.0 YAML schema."
  
  not_medical_advice_label_check:
    therapeutic_assertions: 45
    labeled: 45
    coverage: "100%"
    notes: "Every therapeutic-target assertion carries the 'research evidence — not medical advice' label."

# Release metadata
release_metadata:
  release_id: "v0.1.0"
  release_date: "2026-07-15T10:30:00Z"
  previous_release: null
  assertion_count: 342
  graph_statistics:
    nodes: 156
    edges: 342
    unique_genes: 12
    unique_diseases: 4
    unique_pathways: 18
  generated_by: "ewsr1-fli1-kg-pipeline v0.1.0"
  signed_by: "maintainer@ewsr1-fli1-kg"
```

### 3.3 Machine-Readable Output

The manifest is also exported as **JSON-LD** for programmatic consumption:

```json
{
  "@context": {
    "ewsr1-kg": "https://w3id.org/ewsr1-fli1-kg/",
    "dcat": "http://www.w3.org/ns/dcat#",
    "dct": "http://purl.org/dc/terms/",
    "prov": "http://www.w3.org/ns/prov#"
  },
  "@id": "https://w3id.org/ewsr1-fli1-kg/releases/v0.1.0/manifest",
  "@type": "dcat:Catalog",
  "dct:title": "ewsr1-fli1-kg Source-Version Manifest v0.1.0",
  "dct:issued": "2026-07-15T10:30:00Z",
  "dcat:dataset": [
    {
      "@id": "https://w3id.org/ewsr1-fli1-kg/source/civic",
      "@type": "dcat:Dataset",
      "dct:title": "CIViC: Clinical Interpretation of Variants in Cancer",
      "dcat:accessURL": "https://civicdb.org/api/graphql",
      "dct:issued": "2026-06-15T23:59:59Z",
      "dct:license": "http://creativecommons.org/publicdomain/zero/1.0/",
      "prov:wasGeneratedAtTime": "2026-06-15T23:59:59Z",
      "ewsr1-kg:snapshotIdentifier": "civic-2026-06-15",
      "ewsr1-kg:recordCount": 127,
      "ewsr1-kg:checksumSha256": "abc123def456..."
    }
  ]
}
```

### 3.4 Reproducibility Guarantee

Given the source-version manifest, a downstream consumer can:

1. **Locate the exact snapshot/release** of each source  
2. **Download or access the same version** (via the URLs + release identifiers)  
3. **Regenerate the same assertions** (by running the same extraction tools against the same source versions)  
4. **Validate that the assertions are the same** (via checksums and nanopublication IDs)  

This **closes the reproducibility loop**: provenance + version manifest = full traceability and replicability.

---

## 4. Implementation Roadmap

### Phase 1: Commit this specification (M0)
- Specification document reviewed and merged (this file + acceptance criteria met)
- Decision communicated to the team
- Nanopublication library selected for the TypeScript/ESM tech stack

### Phase 2: Scaffold nanopublication tooling (M0)
- Create data directory structure (`data/assertions/`, `data/sources/`)
- Implement nanopublication serialization (N-Quads, JSON-LD)
- Implement CI linters for provenance completeness + source approval

### Phase 3: First nanopublication import (M1)
- Import CIViC evidence → nanopublications
- Generate source-version manifest
- Export KGX/Biolink/RDF using nanopublication data
- Run CI gates (100%-provenance gate should pass)

### Phase 4: Ongoing (M1–M3)
- Every new source integration follows the same pattern
- Manifest is updated with each release
- Assertions are published as nanopublications to a feed or repository

---

## 5. References & Standards

### Nanopublication Standard
- **Specification:** http://purl.org/nanopub/spec/
- **Java library:** https://github.com/knowledgepats/nanopub-java
- **RDF4J bindings:** https://github.com/semanticweb-java/rdf4j
- **Linked Data:** https://www.w3.org/standards/semanticweb/rdf/

### Biolink Model (Semantic Standard)
- **Specification:** https://biolink.github.io/biolink-model/
- **Predicates:** biolink:transcription_regulation_of, biolink:regulates, etc.
- **Version used in this project:** 4.2.0 (pinned in CI)

### Evidence & Ontologies
- **ECO (Evidence & Conclusion Ontology):** http://purl.obolibrary.org/obo/eco.owl
- **SEPIO (Scientific Evidence and Provenance Information Ontology):** http://purl.obolibrary.org/obo/SEPIO_0000001
- **CIViC evidence model:** https://civicdb.org/help/evidence/evidence-overview

### Persistent Identifiers
- **W3ID:** https://w3id.org/ (host-independent, redirect-based)
- **Identifiers.org CURIEs:** https://identifiers.org/ (standard biomedical entity namespaces)

### Serialization Formats
- **N-Quads (recommended for assertion storage):** https://www.w3.org/TR/n-quads/
- **JSON-LD (recommended for exports):** https://www.w3.org/TR/json-ld11/
- **Turtle (RDF readability):** https://www.w3.org/TR/turtle/
- **KGX/Biolink TSV (downstream KG ingestion):** https://kgx.readthedocs.io/

### Retraction & Integrity
- **PubMed retraction notices:** https://www.nlm.nih.gov/bsd/medline/cit_status.html
- **Retraction Watch dataset:** https://retractionwatch.com/retraction-watch-database/

---

## 6. Acceptance Criteria (from TASK.md ewsr1-fli1-knowledge-graph-prov-001)

- [x] **One provenance mechanism chosen and applied uniformly; the countable 'assertion' unit defined so the 100%-provenance CI gate is checkable.**
  - ✅ Nanopublications chosen (rationale in §1)
  - ✅ Assertion unit defined: nanopublication (core triple + provenance + evidence metadata) (§2)
  - ✅ CI gate defined: provenance completeness check (§2.3)

- [x] **Source-version manifest format defined (CIViC snapshot, Open Targets release, Reactome version, PMC-OA snapshot) so any assertion is reproducible.**
  - ✅ Manifest format specified (YAML + JSON-LD) (§3)
  - ✅ All four sources included with release identifiers (§3.2)
  - ✅ Checksums and reproducibility guarantees documented (§3.4)

---

## Appendix: Nanopublication Graph Structure (Technical Detail)

For reference, the W3C nanopublication structure (http://purl.org/nanopub/core#) defines:

```turtle
@prefix np: <http://purl.org/nanopub/core#> .
@prefix prov: <http://www.w3.org/ns/prov#> .

# A nanopublication consists of 4 named graphs:
nanopub:np1
  a np:Nanopublication ;
  np:hasHead nanopub:np1#head ;           # This nanopublication's URI
  np:hasAssertion nanopub:np1#assertion ; # The core claim
  np:hasProvenance nanopub:np1#provenance ; # Why we believe it
  np:hasPublicationInfo nanopub:np1#pubinfo . # When/who/version

nanopub:np1#assertion {
  # Core triple(s): the actual claim
  <gene A> <regulates> <gene B> .
}

nanopub:np1#provenance {
  # Provenance triples: the evidence and source
  nanopub:np1#assertion prov:wasDerivedFrom <source> .
  <source> dct:license <license-uri> .
  <source> dct:isPartOf "Source Release X" .
  # ... more provenance ...
}

nanopub:np1#pubinfo {
  # Publication metadata: when, who, version
  nanopub:np1 dct:created "2026-07-15T10:30:00Z" .
  nanopub:np1 dct:creator <contributor-uri> .
}
```

This structure is simple, composable, and directly queryable in SPARQL. The CI gate simply counts nanopublications with valid provenance graphs; if any lack one, the gate fails.
