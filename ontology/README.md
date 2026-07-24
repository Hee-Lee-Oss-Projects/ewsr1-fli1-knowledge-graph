# EWSR1-FLI1 Knowledge Graph Schema Specification

**Version:** 1.1.0  
**Date:** 2026-07-23  
**License:** CC0-1.0  
**Aligned to:** Biolink Model v1.8.3  

---

## Overview

This schema defines the core classes and relationships for the EWSR1-FLI1 / EWSR1-ETS knowledge graph of Ewing sarcoma fusion biology. Every class and edge is grounded in the Biolink Model and mapped to authoritative biomedical ontologies: HGNC (genes), MONDO (diseases), Sequence Ontology (molecular features), Reactome (pathways), ChEMBL (compounds), GO (functions), and ECO (evidence types).

The schema encodes the following principles:
- **Provenance-bearing assertions:** Every claim carries source, evidence level, release version, and "not medical advice" label (where applicable).
- **Conflict-faithful representation:** Contradictory or superseded findings retain all sourced variants; no single "truth" is imposed.
- **Precision over recall:** Entity normalization favors false-negative gaps over false-positive mappings; all normalization is human-verified.
- **Open-source only:** All data sourced from CIViC (CC0), Open Targets (CC0), Reactome, and license-checked PMC open-access articles.

---

## Class Definitions

### 1. **Gene**

Represents a genomic entity: the EWSR1 locus, its ETS-family fusion partners (FLI1, ERG, FEV, ETV1, ETV4), and regulated target genes.

**Biolink mapping:** `biolink:Gene`  
**Standard ontology mappings:**
- Primary: HGNC ID (e.g., `HGNC:3238` for EWSR1)
- Secondary: NCBI Gene, Ensembl Gene ID, UniProt Accession
- Functional: GO term (via `biolink:has_biological_function`)

**Properties:**

| Property | Type | Cardinality | Description |
|----------|------|-------------|-------------|
| `id` | IRI (CURIE) | 1 | Unique identifier, resolvable via Biolink prefix (e.g., `hgnc:HGNC:3238`) |
| `name` | string | 1 | Gene symbol (e.g., "EWSR1", "FLI1") |
| `description` | string | 0..1 | Full gene name and brief function |
| `xref` | IRI array | 0..* | Cross-references to NCBI, Ensembl, UniProt, HGNC |
| `in_taxon` | IRI | 1 | Organism (Homo sapiens; `NCBITaxon:9606`) |
| `symbol` | string | 1 | Official gene symbol per HGNC |
| `genomic_location` | string | 0..1 | Chromosomal location (e.g., "22q12.2") |

**Example:**
```
id: hgnc:HGNC:3238
name: "EWSR1"
symbol: "EWSR1"
description: "Ewing sarcoma breakpoint region 1, a pioneer transcription factor fused with ETS partners in Ewing sarcoma"
xref:
  - ncbigene:2130
  - ensembl:ENSG00000182463
  - uniprot:Q01844
in_taxon: NCBITaxon:9606
genomic_location: "22q12.2"
```

---

#### 1a. The EWSR1-ETS Partner Gene Family

EWSR1 (22q12.2) recombines with five known ETS-family transcription-factor genes in Ewing sarcoma. Each partner is modeled as an ordinary `Gene` instance (Section 1); the family is closed over these five members plus EWSR1 itself, and `Fusion.three_prime_partner_gene` (Section 2) is constrained to this set.

| Partner symbol | Cytogenetic band | NCBI Gene (Entrez) ID | Approx. share of EWSR1-ETS fusions | HGNC ID |
|---|---|---|---|---|
| **FLI1** | 11q24.3 | `ncbigene:2313` | ~85% | *verify against HGNC REST lookup — see [Known Gaps](#known-gaps-and-limitations)* |
| **ERG** | 21q22.2 | `ncbigene:2078` | ~10% | *verify against HGNC REST lookup* |
| **ETV1** | 7p21.2 | `ncbigene:2115` | rare | *verify against HGNC REST lookup* |
| **ETV4** | 17q21.31 | `ncbigene:2118` | rare | *verify against HGNC REST lookup* |
| **FEV** | 2q35 | `ncbigene:54738` | rare | *verify against HGNC REST lookup* |

Each partner gene follows the same `Gene` template as the EWSR1 example above (symbol, HGNC/NCBI/Ensembl xrefs, `in_taxon`). Numeric HGNC identifiers are intentionally left as a verification action rather than hard-coded here — per this schema's own entity-normalization policy (Data Quality & Validation, below), identifiers must be resolved and human-verified against the live registry before ingestion rather than asserted from memory.

**Example (ERG partner gene):**
```
id: ncbigene:2078
name: "ERG"
symbol: "ERG"
description: "ETS-family transcription factor; EWSR1's second most common Ewing sarcoma fusion partner after FLI1"
xref:
  - ensembl:ENSG00000157554
in_taxon: NCBITaxon:9606
genomic_location: "21q22.2"
```

---

### 2. **Fusion**

Represents a somatic fusion protein resulting from a breakpoint between two genes.

**Biolink mapping:** `biolink:Protein` (with `biolink:has_gene_template` to source genes)  
**Standard ontology mappings:**
- Sequence Ontology: `SO:0000159` (read-through transcript), `SO:0001631` (upstream_open_reading_frame), or `SO:0001026` (missense_variant) context
- ChEMBL: fusion-specific identifiers where available
- Literature: CIViC Variant Fusion Type

**Properties:**

| Property | Type | Cardinality | Description |
|----------|------|-------------|-------------|
| `id` | IRI (CURIE) | 1 | Unique fusion identifier (e.g., `ewsr1-fli1-kg:fusion_EWSR1_FLI1_type1`) |
| `name` | string | 1 | Fusion name (e.g., "EWSR1-FLI1", "EWSR1-ERG") |
| `five_prime_partner_gene` | IRI | 1 | 5' breakpoint gene (EWSR1; reference to `Gene.id`) |
| `three_prime_partner_gene` | IRI | 1 | 3' breakpoint gene (ETS family member; reference to `Gene.id`) |
| `fusion_breakpoint` | IRI array | 1..* | Ordered list of FusionBreakpoint entities defining the exon junctions and subtypes |
| `description` | string | 0..1 | Brief characterization |
| `is_protein_altering` | boolean | 1 | True for all in-frame fusions; false for out-of-frame (rarely present in Ewing) |
| `derived_from` | IRI array | 0..* | Reference to SourceDocument(s) attesting the fusion |

**Example:**
```
id: ewsr1-fli1-kg:fusion_EWSR1_FLI1
name: "EWSR1-FLI1"
five_prime_partner_gene: hgnc:HGNC:3238
three_prime_partner_gene: hgnc:HGNC:3236
fusion_breakpoint:
  - ewsr1-fli1-kg:breakpoint_EWSR1_FLI1_type1
  - ewsr1-fli1-kg:breakpoint_EWSR1_FLI1_type2
description: "The canonical EWSR1-FLI1 fusion oncoprotein, a pioneer transcription factor present in ~85% of Ewing sarcoma"
is_protein_altering: true
derived_from:
  - civicdb:evidence/id/12345
```

**The five EWSR1-ETS fusions modeled by this schema:**

| Fusion | `id` pattern | 3' partner (`Gene.id`) | Notes |
|---|---|---|---|
| EWSR1-FLI1 | `ewsr1-fli1-kg:fusion_EWSR1_FLI1` | `ncbigene:2313` | Canonical; Type 1/Type 2 breakpoints (see FusionBreakpoint) |
| EWSR1-ERG | `ewsr1-fli1-kg:fusion_EWSR1_ERG` | `ncbigene:2078` | Second most common; analogous Type 1/Type 2 breakpoint nomenclature reported independently in the ERG literature |
| EWSR1-ETV1 | `ewsr1-fli1-kg:fusion_EWSR1_ETV1` | `ncbigene:2115` | Rare; single case-series-level breakpoint characterization |
| EWSR1-ETV4 | `ewsr1-fli1-kg:fusion_EWSR1_ETV4` | `ncbigene:2118` | Rare; single case-series-level breakpoint characterization |
| EWSR1-FEV | `ewsr1-fli1-kg:fusion_EWSR1_FEV` | `ncbigene:54738` | Rarest reported partner |

Every row reuses the identical `Fusion` and `FusionBreakpoint` templates — only `three_prime_partner_gene`, `fusion_breakpoint`, and the literature-sourced `derived_from` citations differ. No partner-specific subclassing is needed because the schema's cardinalities and datatypes are already partner-agnostic.

**Example (EWSR1-ERG):**
```
id: ewsr1-fli1-kg:fusion_EWSR1_ERG
name: "EWSR1-ERG"
five_prime_partner_gene: hgnc:HGNC:3238
three_prime_partner_gene: ncbigene:2078
fusion_breakpoint:
  - ewsr1-fli1-kg:breakpoint_EWSR1_ERG_type1
description: "The second most common EWSR1-ETS fusion in Ewing sarcoma, present in a minority of cases lacking EWSR1-FLI1"
is_protein_altering: true
derived_from:
  - civicdb:evidence/id/67890
```

---

### 3. **FusionBreakpoint**

Specifies the molecular junction of a fusion at the exon level, including Sequence Ontology classification and partner identity. Enables precise biological modeling of Type 1, Type 2, and rarer junction subtypes.

**Biolink mapping:** `biolink:SequenceFeature` (or `SO:Sequence_feature`)  
**Standard ontology mappings:**
- Sequence Ontology: `SO:0001625` (chromosome_breakpoint), `SO:0001631` (junction_sequence), or specific fusion subtype terms
- Example: EWSR1 exon 7 / FLI1 exon 4 (Type 1); EWSR1 exon 7 / FLI1 exon 6 (Type 2)

**Properties:**

| Property | Type | Cardinality | Description |
|----------|------|-------------|-------------|
| `id` | IRI (CURIE) | 1 | Unique breakpoint identifier (e.g., `ewsr1-fli1-kg:breakpoint_EWSR1_FLI1_type1`) |
| `junction_type` | IRI | 1 | SO term identifying the molecular junction class (e.g., `SO:0001625` for chromosome_breakpoint) |
| `breakpoint_subtype` | string | 1 | Clinical/literature designation (e.g., "Type 1", "Type 2") |
| `five_prime_exon` | integer | 1 | 5' partner exon number (EWSR1 exon 7 for most EWSR1-FLI1) |
| `five_prime_exon_coordinate` | string | 0..1 | Genomic coordinate of 5' exon boundary (e.g., "22:29134393") |
| `three_prime_exon` | integer | 1 | 3' partner exon number (FLI1 exon 4 for Type 1, exon 6 for Type 2) |
| `three_prime_exon_coordinate` | string | 0..1 | Genomic coordinate of 3' exon boundary |
| `reading_frame` | string | 1 | "in_frame" or "out_of_frame" |
| `description` | string | 0..1 | Functional annotation (e.g., "Canonical Type 1 junction; ~40% of EWSR1-FLI1 cases") |
| `frequency_in_ewing` | string | 0..1 | Observed prevalence (e.g., "~40%", "rare") |

**Example:**
```
id: ewsr1-fli1-kg:breakpoint_EWSR1_FLI1_type1
junction_type: SO:0001625
breakpoint_subtype: "Type 1"
five_prime_exon: 7
five_prime_exon_coordinate: "22:29134393"
three_prime_exon: 4
three_prime_exon_coordinate: "11:128714071"
reading_frame: "in_frame"
description: "EWSR1 exon 7 / FLI1 exon 4 junction; canonical Type 1 fusion, ~40% of EWSR1-FLI1 Ewing cases"
frequency_in_ewing: "~40%"
```

**Breakpoint nomenclature is partner-specific, not universal.** "Type 1" / "Type 2" is the historical EWSR1-FLI1 designation (Zucman et al., 1993, exon-junction based). EWSR1-ERG breakpoints are independently classified with their own Type 1/Type 2 labels in the ERG literature, using different exon boundaries. The `breakpoint_subtype` field is intentionally a free-text string (not a closed enum) to accommodate this per-partner nomenclature and any rarer/atypical junctions reported for ETV1, ETV4, or FEV fusions, where breakpoint typing has not yet been standardized. See [Known Gaps and Limitations](#known-gaps-and-limitations) for the absence of a dedicated Sequence Ontology term distinguishing these subtypes.

**Example (EWSR1-ERG, Type 1):**
```
id: ewsr1-fli1-kg:breakpoint_EWSR1_ERG_type1
junction_type: SO:0001625
breakpoint_subtype: "Type 1"
five_prime_exon: 7
three_prime_exon: 6
reading_frame: "in_frame"
description: "EWSR1 exon 7 / ERG exon 6 junction; most frequently reported EWSR1-ERG breakpoint"
```

---

### 4. **Disease**

Represents Ewing sarcoma and related disease entities.

**Biolink mapping:** `biolink:Disease`  
**Standard ontology mappings:**
- MONDO: `MONDO:0008121` (Ewing sarcoma)
- EFO (Experimental Factor Ontology): EFO_0002938
- NCIT (NCI Thesaurus): C9145
- Orphanet: ORPHA_2421

**Properties:**

| Property | Type | Cardinality | Description |
|----------|------|-------------|-------------|
| `id` | IRI (CURIE) | 1 | Unique disease identifier (e.g., `mondo:MONDO:0008121`) |
| `name` | string | 1 | Disease name (e.g., "Ewing sarcoma") |
| `description` | string | 0..1 | Clinical and epidemiological summary |
| `xref` | IRI array | 0..* | NCIT, EFO, Orphanet, MeSH identifiers |
| `in_taxon` | IRI | 1 | Affected species (Homo sapiens; NCBITaxon:9606) |
| `disease_onset_qualifier` | string | 0..1 | Age of onset (e.g., "childhood to young adulthood") |
| `has_material_basis_in` | IRI | 0..1 | Molecular driver (reference to Fusion.id for Ewing-EWSR1-FLI1) |

**Example:**
```
id: mondo:MONDO:0008121
name: "Ewing sarcoma"
description: "A malignant bone and soft-tissue tumor of children and young adults, driven in ~85% of cases by EWSR1-ETS fusion oncoprotein"
xref:
  - ncit:C9145
  - efo:EFO_0002938
  - orphanet:ORPHA_2421
in_taxon: NCBITaxon:9606
disease_onset_qualifier: "childhood to young adulthood"
has_material_basis_in: ewsr1-fli1-kg:fusion_EWSR1_FLI1
```

---

### 5. **Pathway**

Represents canonical biological pathways (e.g., Reactome) that are affected by the fusion.

**Biolink mapping:** `biolink:Pathway`  
**Standard ontology mappings:**
- Reactome: R-HSA pathway identifiers (e.g., R-HSA-1400328 for "Transcriptional Regulation by TP53")
- GO Biological Process: GO:0006355 (regulation of transcription)

**Properties:**

| Property | Type | Cardinality | Description |
|----------|------|-------------|-------------|
| `id` | IRI (CURIE) | 1 | Reactome or GO pathway identifier (e.g., `reactome:R-HSA-1400328`) |
| `name` | string | 1 | Pathway name |
| `description` | string | 0..1 | Summary of the pathway and its biological role |
| `xref` | IRI array | 0..* | KEGG, Wikipathways, or other pathway database IDs |
| `has_part` | IRI array | 0..* | Gene products or protein complexes in the pathway |
| `in_taxon` | IRI | 1 | Species (Homo sapiens; NCBITaxon:9606) |
| `evidence_level` | string | 0..1 | Experimental evidence supporting the link (e.g., "experimental", "computational") |

**Example:**
```
id: reactome:R-HSA-1400328
name: "Transcriptional Regulation by TP53"
description: "Reactome pathway describing p53-mediated transcriptional control; EWSR1-FLI1 interferes with TP53 function and normal p53 target-gene regulation"
xref:
  - go:GO:0006355
  - kegg:hsa04115
in_taxon: NCBITaxon:9606
evidence_level: "experimental"
```

---

### 6. **Mechanism**

Represents molecular or cellular mechanisms by which the fusion affects biology (e.g., GGAA-microsatellite binding, pioneer-factor activity, transcriptional regulation).

**Biolink mapping:** `biolink:BiologicalProcess` (or `biolink:MolecularActivity`)  
**Standard ontology mappings:**
- GO Biological Process: GO:0006355 (regulation of transcription)
- GO Molecular Function: GO:0003700 (DNA-binding transcription factor activity)
- ECO: Evidence codes for the mechanism evidence type

**Properties:**

| Property | Type | Cardinality | Description |
|----------|------|-------------|-------------|
| `id` | IRI (CURIE) | 1 | Unique mechanism identifier (e.g., `ewsr1-fli1-kg:mechanism_pioneer_factor`) |
| `name` | string | 1 | Mechanism name (e.g., "GGAA-microsatellite pioneer-factor binding") |
| `description` | string | 0..1 | Detailed explanation of the molecular mechanism |
| `has_input` | IRI array | 0..* | Inputs (e.g., Fusion, Gene) that participate in the mechanism |
| `has_output` | IRI array | 0..* | Outputs (e.g., altered Gene expression, Pathway perturbation) |
| `mechanism_type` | IRI | 1 | GO or SO term (e.g., `GO:0006355` for transcriptional regulation) |
| `evidence_types` | IRI array | 1..* | ECO terms for supporting evidence (e.g., `ECO:0000269` for experimental evidence) |

**Example:**
```
id: ewsr1-fli1-kg:mechanism_pioneer_factor
name: "GGAA-microsatellite pioneer-factor binding"
description: "EWSR1-FLI1 acts as an ETS-domain transcription factor that recognizes and binds GGAA-microsatellite motifs in enhancer regions, opening chromatin and enabling transcriptional activation at otherwise-silent loci"
has_input:
  - ewsr1-fli1-kg:fusion_EWSR1_FLI1
has_output:
  - ewsr1-fli1-kg:mechanism_transcriptional_activation
mechanism_type: GO:0006355
evidence_types:
  - ECO:0000269
  - ECO:0000314
```

---

### 7. **TherapeuticAgent**

Represents research-stage compounds, targets, or modalities with potential therapeutic relevance to EWSR1-FLI1 biology. **Labeled as research evidence only; not medical advice.**

**Biolink mapping:** `biolink:SmallMolecule` or `biolink:Protein` (depending on agent type)  
**Standard ontology mappings:**
- ChEMBL: ChEMBL ID (e.g., CHEMBL:CHEMBL59) for compounds
- DrugCentral (open data only; no DrugBank)
- GO Molecular Function for target proteins

**Properties:**

| Property | Type | Cardinality | Description |
|----------|------|-------------|-------------|
| `id` | IRI (CURIE) | 1 | ChEMBL or protein ID (e.g., `chembl:CHEMBL59`) |
| `name` | string | 1 | Common name or IUPAC (e.g., "Temozolomide") |
| `agent_type` | string | 1 | "small_molecule", "antibody", "kinase_inhibitor", "protein_target", or other modality |
| `description` | string | 0..1 | Mechanism of action and rationale for EWSR1-FLI1 context |
| `xref` | IRI array | 0..* | DrugCentral (open), PubChem, or other open identifiers |
| `targets` | IRI array | 0..* | Gene/protein targets (references to Gene.id or biolink:Protein) |
| `development_stage` | string | 1 | "preclinical", "IND", "Phase 1", "Phase 2", "Phase 3", "approved" |
| `is_research_evidence_only` | boolean | 1 | Always true; flagged in assertions and visualizations |

**Example:**
```
id: chembl:CHEMBL59
name: "Temozolomide"
agent_type: "small_molecule"
description: "Alkylating agent under preclinical and clinical investigation in Ewing sarcoma; may sensitize EWSR1-FLI1-driven cells to DNA damage"
xref:
  - pubchem:5391
  - drugcentral:DB00853
targets: []
development_stage: "approved"
is_research_evidence_only: true
```

---

### 8. **EvidenceAssertion**

The provenance-bearing unit of the knowledge graph. Encodes subject–predicate–object triples with full evidence lineage, source attribution, evidence level, and regulatory metadata.

**Biolink mapping:** `biolink:InformationResource` or an RDF graph pattern (subject, predicate, object) with attached properties  
**Standard ontology mappings:**
- ECO (Evidence & Conclusion Ontology): evidence type classification
- SEPIO: evidence-provenance model (custodian adopted for nanopublications)
- Provenance ontology (PROV-O): `prov:wasDerivedFrom`, `prov:wasAttributedTo`

**Properties:**

| Property | Type | Cardinality | Description |
|----------|------|-------------|-------------|
| `id` | IRI (CURIE) | 1 | Unique assertion identifier (e.g., `nanopub:nanopub_12345`) |
| `subject` | IRI | 1 | Subject entity (Gene, Fusion, Disease, etc.) |
| `predicate` | IRI | 1 | Biolink or custom relation (e.g., `biolink:regulates`, `biolink:associated_with_increased_risk`) |
| `object` | IRI | 1 | Object entity (Gene, Pathway, Mechanism, TherapeuticAgent, etc.) |
| `assertion_type` | string | 1 | "regulatory", "association", "mechanistic", "therapeutic", "contraindication" |
| `evidence_type` | IRI array | 1..* | ECO terms (e.g., `ECO:0000269` for experimental evidence, `ECO:0000033` for author statement) |
| `evidence_level` | string | 1 | "experimental", "computational", "literature", "expert_assertion" |
| `confidence_score` | float | 0..1 | Confidence (0.0–1.0) assigned by extraction pipeline or curators (optional) |
| `source_document` | IRI | 1 | Reference to SourceDocument (which paper, database, release) |
| `source_release_version` | string | 1 | Exact release/version of source (e.g., "CIViC 2026-07-23", "PMID:12345678 / PMC9876543") |
| `extraction_method` | string | 1 | "manual_curation", "structured_import", "literature_extraction" |
| `provenance_record` | object | 1 | Machine-readable provenance (source ID, license, date extracted) |
| `not_medical_advice` | boolean | 0..1 | Set to true if object is TherapeuticAgent or if assertion implies clinical action; must be present in derived output |
| `conflict_note` | string | 0..1 | If assertion contradicts another, cite the competing evidence (e.g., "Contradicts Assertion #456 (Reactome May 2025)") |
| `superseded_by` | IRI array | 0..* | If assertion is retracted/withdrawn, cite the superceding assertion(s) |
| `assertion_date` | dateTime | 1 | When assertion was extracted/curated (ISO 8601) |
| `curator_notes` | string | 0..1 | Free-text human curation commentary |

**Example (regulatory):**
```
id: nanopub:nanopub_EWSR1_FLI1_regulates_IGF1
subject: ewsr1-fli1-kg:fusion_EWSR1_FLI1
predicate: biolink:regulates
object: hgnc:HGNC:5459
assertion_type: "regulatory"
evidence_type:
  - ECO:0000269
evidence_level: "experimental"
confidence_score: 0.9
source_document: civicdb:evidence/id/12345
source_release_version: "CIViC 2026-07-01"
extraction_method: "structured_import"
provenance_record:
  source_id: "civicdb:evidence/id/12345"
  license: "CC0"
  extracted_date: "2026-07-23T10:30:00Z"
not_medical_advice: false
assertion_date: "2026-07-23T10:30:00Z"
```

**Example (therapeutic, with not-medical-advice flag):**
```
id: nanopub:nanopub_EWSR1_FLI1_therapeutic_CDK4_inhibitor
subject: ewsr1-fli1-kg:fusion_EWSR1_FLI1
predicate: biolink:potentially_treated_by
object: chembl:CHEMBL428
assertion_type: "therapeutic"
evidence_type:
  - ECO:0000033
evidence_level: "literature"
confidence_score: 0.6
source_document: pmid:12345678
source_release_version: "PMID:12345678 / PMC9876543 (CC BY)"
extraction_method: "literature_extraction"
provenance_record:
  source_id: "PMID:12345678"
  license: "CC BY"
  extracted_date: "2026-07-23"
not_medical_advice: true
conflict_note: "Preclinical evidence only; clinical efficacy not established. See also Assertion #789 (negative study, 2024)."
assertion_date: "2026-07-23T11:00:00Z"
curator_notes: "Mark as low-confidence pending additional validation. CDK4/6i sensitivity is context-dependent."
```

---

### 9. **SourceDocument**

Represents the original source: a published article, database release, or bulk-import event.

**Biolink mapping:** `biolink:InformationResource`  
**Standard ontology mappings:**
- DCMI (Dublin Core Metadata Initiative) terms for publication metadata
- License ontologies: CC-BY, CC-BY-NC, CC0, PD, etc.

**Properties:**

| Property | Type | Cardinality | Description |
|----------|------|-------------|-------------|
| `id` | IRI (CURIE) | 1 | Persistent document identifier (e.g., `pmid:12345678` or `civicdb:release/2026-07-01`) |
| `title` | string | 1 | Document title or database release name |
| `authors` | string array | 0..* | Author names (for publications) |
| `publication_date` | date | 0..1 | Original publication or release date (ISO 8601) |
| `url` | IRI | 0..1 | URL to the source (stable, persistent) |
| `document_type` | string | 1 | "journal_article", "database_release", "preprint", "technical_report" |
| `license` | string | 1 | "CC0", "CC-BY", "CC-BY-NC", "CC-BY-ND", "CC-BY-NC-ND", "PD", "proprietary_no_license" (for out-of-scope sources) |
| `license_verified_date` | date | 0..1 | Date license status was verified (for PMC-OA articles) |
| `rights_statement` | string | 0..1 | Machine-readable assertion of rights (e.g., "This work is in the public domain.") |
| `source_identifier` | string | 1 | Canonical identifier from the source (e.g., PMID, PubMed Central ID, CIViC release) |
| `source_release_version` | string | 1 | Version or release tag of the database/archive at ingestion time |
| `abstract` | string | 0..1 | Publication abstract (for journals) or release notes (for databases) |
| `mesh_terms` | string array | 0..* | MeSH indexing (for journal articles) |
| `pubmed_id` | string | 0..1 | PMID (if a PubMed-indexed article) |
| `pmc_id` | string | 0..1 | PubMed Central ID (if PMC-OA) |
| `doi` | string | 0..1 | Digital Object Identifier |

**Example (journal article):**
```
id: pmid:12345678
title: "EWSR1-FLI1-driven pioneer factor binding at GGAA-microsatellites enhances oncogenic transcription"
authors:
  - "Riggi N"
  - "Suvà ML"
  - "others"
publication_date: "2014-06-15"
url: "https://pubmed.ncbi.nlm.nih.gov/12345678"
document_type: "journal_article"
license: "CC-BY"
license_verified_date: "2026-07-15"
rights_statement: "This article is distributed under the terms of the Creative Commons Attribution 4.0 International License."
source_identifier: "12345678"
source_release_version: "PubMed/PMC 2026-07-01"
abstract: "We demonstrate that EWSR1-FLI1..."
mesh_terms:
  - "Ewing Sarcoma"
  - "Fusion Proteins, Oncogenic"
  - "Transcription Factors"
pubmed_id: "12345678"
pmc_id: "PMC9876543"
doi: "10.1234/example.2014.123456"
```

**Example (database release):**
```
id: civicdb:release/2026-07-01
title: "CIViC Release 2026-07-01"
publication_date: "2026-07-01"
url: "https://civicdb.org"
document_type: "database_release"
license: "CC0"
rights_statement: "All CIViC content is available under the CC0 public domain."
source_identifier: "civicdb"
source_release_version: "2026-07-01"
abstract: "CIViC is a freely-available resource for cancer researchers to interpret the clinical significance of cancer variants."
```

---

## Edge (Relationship) Definitions

All edges conform to Biolink predicates. The following table lists the primary edges in use:

| Source Class | Predicate | Target Class | Biolink Mapping | Cardinality | Notes |
|---------------|-----------|---------------|-----------------|-------------|-------|
| Fusion | `regulates` | Gene | `biolink:regulates` | 0..* | EWSR1-FLI1 activates/represses target genes |
| Fusion | `has_biological_sequence` | FusionBreakpoint | `biolink:has_biological_sequence` | 1..* | Fusion consists of one or more breakpoint junctions |
| Fusion | `associated_with_increased_risk` | Disease | `biolink:associated_with_increased_risk` | 1..* | Fusion drives disease |
| Fusion | `involved_in` | Pathway | `biolink:involved_in` | 0..* | Fusion perturbs canonical pathways |
| Fusion | `has_mechanism` | Mechanism | (custom) | 0..* | Molecular/cellular mechanism link |
| Gene | `involved_in` | Pathway | `biolink:involved_in` | 0..* | Gene participates in pathway |
| Gene | `has_biological_function` | Mechanism | (custom via GO) | 0..* | Gene function in context of EWSR1-FLI1 |
| Disease | `has_material_basis_in` | Fusion | `biolink:has_material_basis_in` | 1..* | Disease caused by fusion |
| Mechanism | `regulates` | Gene | `biolink:regulates` | 0..* | Mechanism drives gene expression |
| Mechanism | `part_of` | Pathway | `biolink:part_of` | 0..* | Mechanism is a step in broader pathway |
| TherapeuticAgent | `treats` | Disease | `biolink:treats` | 0..* | **Research evidence only; not medical advice** |
| TherapeuticAgent | `targets` | Gene | `biolink:targets` | 0..* | Agent binds or inhibits protein product |
| EvidenceAssertion | `subject` | (any) | (RDF property) | 1 | RDF-triple-like encoding |
| EvidenceAssertion | `predicate` | (any) | (RDF property) | 1 | — |
| EvidenceAssertion | `object` | (any) | (RDF property) | 1 | — |

---

## Conflict and Supersession Representation

When two assertions contradict (e.g., one study reports gene X is activated by EWSR1-FLI1, another reports it is repressed), **both assertions are published side-by-side with full evidence lineage.** The assertion model achieves this via:

1. **Independent assertion IDs:** Each assertion receives a unique, permanent IRI.
2. **Evidence type and level:** Each carries its evidence type (`ECO` term) and level (experimental, computational, literature, expert).
3. **Source version pinning:** Each asserts the exact source release version it came from.
4. **Conflict note:** The `conflict_note` field explicitly flags contradictions, citing the competing assertion ID and source.
5. **No "winner" enforcement:** The graph never enforces a single "true" assertion; instead, consumers can query the full assertion set and apply their own filtering or confidence weighting.

**Example:**
- Assertion A: "EWSR1-FLI1 activates FOO" (experimental, CIViC 2024)
- Assertion B: "EWSR1-FLI1 represses FOO" (literature, PMID:98765432)
- Both are published; Assertion A's curator notes cite Assertion B's ID and note the contradiction; downstream tools can weight by evidence level.

---

## Regulatory Labels and Declarations

### "Not Medical Advice" Flagging

Every EvidenceAssertion related to therapeutic agents or clinical intervention **must carry `not_medical_advice: true`** and prominently display this in any human-facing output. This is a hard CI gate: assertions touching therapeutic targets or disease treatment implications are withheld from publication if the label is absent.

### Research-Evidence-Only Boundary

- TherapeuticAgent.is_research_evidence_only is always true.
- Assertions about therapeutic agents carry conflict notes, evidence levels, and supersession pointers to ensure consumers see the full uncertainty landscape.
- No assertion implies clinical utility or medical recommendation.

---

## Ontology Mappings Reference

| Concept | Biolink Class | Standard Ontology | Identifier Format | Example |
|---------|---------------|-------------------|-------------------|---------|
| Gene | `biolink:Gene` | HGNC, NCBI Gene, Ensembl | `hgnc:HGNC:3238` | EWSR1 |
| Protein/Gene Product | `biolink:Protein` | UniProt, ChEMBL | `uniprot:Q01844` | EWSR1 protein |
| Fusion (structural variant) | `biolink:Protein` + `biolink:SequenceFeature` | SO (Sequence Ontology) | Custom or CIViC ID | EWSR1-FLI1 |
| Sequence Feature (breakpoint) | `biolink:SequenceFeature` | SO | `SO:0001625` | Chromosome breakpoint |
| Disease | `biolink:Disease` | MONDO, EFO, NCIT, Orphanet | `mondo:MONDO:0008121` | Ewing sarcoma |
| Pathway | `biolink:Pathway` | Reactome, KEGG, Wikipathways | `reactome:R-HSA-1400328` | Transcriptional regulation |
| Biological Process | `biolink:BiologicalProcess` | GO (Gene Ontology) | `go:GO:0006355` | Transcription regulation |
| Molecular Function | `biolink:MolecularActivity` | GO | `go:GO:0003700` | DNA-binding TF activity |
| Small Molecule / Drug | `biolink:SmallMolecule` | ChEMBL, PubChem, DrugCentral | `chembl:CHEMBL59` | Temozolomide |
| Evidence Type | (property) | ECO (Evidence & Conclusion Ontology) | `ECO:0000269` | Experimental evidence |
| Source / Publication | `biolink:InformationResource` | DCMI + PubMed + License ontologies | `pmid:12345678` | PubMed article |

---

## Known Gaps and Limitations

No public ontology fully covers gene-fusion knowledge graphs; the following mapping gaps are known and intentionally documented rather than papered over with an invented "exact" term:

| Gap | Where it occurs | How this schema handles it |
|---|---|---|
| Biolink Model has no dedicated `GeneFusion` or `FusionProtein` class | `Fusion` class | Modeled as `biolink:Protein` + `biolink:has_gene_template`, the closest existing Biolink pattern; flagged as an approximation, not an exact match |
| Sequence Ontology has no term distinguishing fusion "Type 1" vs. "Type 2" breakpoint subtypes | `FusionBreakpoint.breakpoint_subtype` | Generic `SO:0001625` (chromosome_breakpoint) used for `junction_type`; the partner-specific Type 1/2 label is carried as a free-text string, not a controlled SO term (see FusionBreakpoint section above) |
| Biolink Model has no dedicated `Mechanism` class | `Mechanism` | Modeled as `biolink:BiologicalProcess`; molecular-function-flavored mechanisms may equally fit `biolink:MolecularActivity` — curators should pick the closer fit per instance and note the choice in `curator_notes` |
| Exact numeric HGNC identifiers for the ERG, ETV1, ETV4, and FEV partner genes are not hard-coded in this spec | Gene (EWSR1-ETS Partner Gene Family) | Left as an explicit verification action against the live HGNC REST API before ingestion, consistent with this schema's normalization policy (unverified entities are excluded from publication, below) |
| ChEMBL/DrugCentral have no identifier for a "target-only" protein with no associated compound | `TherapeuticAgent.targets` | Falls back to `Gene.id` (e.g., an HGNC/NCBI Gene reference) with no independent small-molecule ontology term; `agent_type: "protein_target"` signals this case |
| Reactome does not have a pathway entry for every fusion-specific perturbation described in primary literature | `Pathway` | Mechanism-level GO terms (`Mechanism.mechanism_type`) are used to capture the finding when no Reactome pathway ID exists yet; the pathway edge is simply omitted rather than mapped to an unrelated pathway |
| MONDO/EFO/NCIT/Orphanet occasionally disagree on subtype granularity for Ewing sarcoma variants (e.g., extraosseous vs. osseous) | `Disease` | All applicable xrefs are recorded (multi-valued `xref`); no single "correct" disease ID is forced when sources disagree |

---

## Data Quality & Validation

### Cardinality and Mandatory Fields

- **Mandatory** (cardinality 1): id, name, symbol (Gene); id, name, five_prime_partner_gene, three_prime_partner_gene (Fusion); subject, predicate, object, evidence_level, source_document (EvidenceAssertion).
- **Required conditionally:** `not_medical_advice` is mandatory if object is a TherapeuticAgent or if assertion implies treatment/clinical outcome.
- **Optional** (cardinality 0..1): most descriptive fields.
- **Multi-valued** (cardinality 0..*): xref, has_part, evidence_types, targets, authors, mesh_terms.

### Grounding and Entity Normalization

All Gene, Disease, Pathway, and TherapeuticAgent instances **must** resolve to at least one standard ontology identifier:
- Gene → HGNC ID (or NCBI/Ensembl)
- Disease → MONDO ID
- Pathway → Reactome ID
- TherapeuticAgent → ChEMBL ID (or open alternative)
- Sequence feature → SO term

Unmappable entities are **excluded** from publication; normalization precision is prioritized over completeness.

### Provenance Completeness

Every EvidenceAssertion **must** carry:
- source_document (IRI)
- source_release_version (string with date/version)
- provenance_record (object with source_id, license, extracted_date)

This is a hard CI gate. Un-sourced assertions are never published.

### License Compliance

- All sources must be CC0, CC-BY, CC-BY-NC (with explicit license preservation), or PD.
- PMC-OA articles are per-article license-checked before extraction.
- Assertions from proprietary sources (COSMIC, OncoKB, controlled-access) are categorically rejected.

---

## Export Formats

The knowledge graph is serialized in three complementary formats, all carrying the full schema and provenance:

1. **KGX/Biolink TSV** (tab-separated values): nodes and edges conform to Biolink Model.
2. **RDF/Turtle:** W3C-standard linked-data format with dereferenceable IRIs and nanopublications.
3. **JSON-LD:** Hierarchical JSON representation with @context linking to Biolink and standard ontologies.

All three formats are validated against SHACL shapes (see `ontology/ewsr1-fli1-knowledge-graph-schema.shacl`).

---

## Version History

| Version | Date | Change |
|---------|------|--------|
| 1.0.0 | 2026-07-23 | Initial schema specification aligned to Biolink v1.8.3 and standard biomedical ontologies |
| 1.1.0 | 2026-07-23 | Modeled the full EWSR1-ETS partner family (FLI1, ERG, ETV1, ETV4, FEV) with worked Fusion/FusionBreakpoint examples for a second partner (ERG); added the Known Gaps and Limitations section documenting unmapped ontology terms |

---

## References & Normative Documents

- **Biolink Model:** https://biolink.github.io/biolink-model/ (v1.8.3)
- **HGNC:** https://www.genenames.org/
- **MONDO:** https://github.com/monarch-initiative/mondo
- **Sequence Ontology:** http://www.sequenceontology.org/
- **Reactome:** https://reactome.org/
- **ChEMBL:** https://www.ebi.ac.uk/chembl/
- **Gene Ontology:** http://geneontology.org/
- **ECO (Evidence & Conclusion Ontology):** http://purl.obolibrary.org/obo/eco
- **CIViC:** https://civicdb.org/ (CC0)
- **Open Targets Platform:** https://platform.opentargets.org/ (CC0)
- **DCMI (Dublin Core Metadata Initiative):** https://dublincore.org/

---

## Contact & Attribution

**Schema authors:** Hee-Lee Oss Project Contributors  
**License:** CC0-1.0 (public domain)  
**Date:** 2026-07-23  

For questions, issues, or contributions, please refer to the project repository and CONTRIBUTING guidelines.
