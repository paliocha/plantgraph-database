# PlantGraph SKILL.md Expansion — Design Spec

## Goal

Expand SKILL.md from a Cypher-only reference (~330 lines) into a comprehensive PlantGraph API handbook (~900-1000 lines) covering all 5 API categories, auth, enrichment, AI features, and deep Cypher reference. Single file, layered structure (quick reference up top, deep sections below).

## Architecture: Layered Single File (Approach B)

Quick reference section (~200 lines) handles 80% of queries. Deep reference sections follow for when Claude needs specifics. Claude can stop reading when it has enough context.

## Data Sources

- Current SKILL.md (battle-tested Cypher patterns, ID rules)
- Official docs at plantgraph.se/documentation/ (Graph API, Enrichment API, AI API, Auth, Data Sources)
- Live API probing results (schema introspection, undocumented endpoints, relationship counts, node/edge properties)

## Key Corrections from Live Probing

1. **ORTHOLOGOUS_TO is not Arabidopsis-anchored.** Current SKILL.md says "edges point FROM Arabidopsis outward." Live data shows cross-species edges in all directions (Brassica napus is dominant source). Direction is implementation detail. Undirected match advice stays correct, but reasoning changes.
2. **Auth exists.** Current SKILL.md says "No authentication required." True only for raw Cypher endpoint. REST endpoints need JWT bearer token. Two tiers: guest (60/hr), researcher (1000/hr).
3. **Graph is larger than stated.** Docs say 14.5M+ nodes, 47.9M+ relationships (vs "1.6M loci" in current skill — loci are a subset). 127 species confirmed (vs "60+").
4. **Publication.research_concepts has inconsistent type** — sometimes List[str], sometimes semicolon-delimited string. Must handle both.
5. **Enrichment p_threshold capped at 0.5** server-side (not documented). Minimum 2 genes enforced by Pydantic.

## Section-by-Section Design

### 1. Frontmatter & Auth Setup

**Frontmatter changes:**
- Expand `description` to include trigger keywords: enrichment, AlphaFold, STRING, protein structure, paper evidence, hypothesis generation, GNN, gene function prediction, research questions, chat
- Keep `name: plantgraph-database`

**Auth section (new, right after frontmatter):**
- Base URLs: `https://plantgraph.se/api` (production), note `/api/v1/` prefix for some endpoints
- JWT login: `POST /api/auth/login` with `{"username", "password"}` → `{"access_token", "token_type", "expires_in": 86400}`
- Token refresh: `POST /api/auth/refresh` with bearer header
- Required header for REST endpoints: `Authorization: Bearer <token>`
- Rate limits table: guest 60/hr (10/min burst), researcher 1000/hr (60/min burst)
- Note: raw Cypher endpoint `/api/v1/graph/query` currently works without auth

### 2. Quick Reference

**Master endpoints table** — all confirmed endpoints, grouped by category:
- Graph: gene lookup, search, neighbors, pathways, stats, network endpoints, raw Cypher, NL query
- Enrichment: per-gene, per-source, batch, GO enrichment, pathway enrichment, cache status
- AI: question generation (abstract, PMID, random), chat, hypothesis, GNN predict/links, Q&A discovery
- Paper Evidence: check-abstract, verify-claims (SSE), check (PDF), verify-claims-pdf (SSE)
- Search: /search/terms, /search/genes (undocumented but functional)
- Auth: login, refresh
- Status: health check

**ID format rules** — keep existing table verbatim:
- Arabidopsis: `l.name` (AT2G40150) or `l.id`
- Brachypodium: `l.id` (BRADI_1g16610v3)
- Rice: `l.id` (Os01g0111600, RAP format, not MSU LOC_Os*)
- Wheat: `l.id` (TraesCS1D02G002600)
- Maize: `l.id` (Zm00001eb078980)
- Barley: not in graph, use orthologue chain
- Phytozome conversion rules
- B. sylvaticum strategy

**Top 10 Cypher patterns** (curated from current 20):
1. Gene lookup by ID (Arabidopsis vs non-Arabidopsis)
2. Gene symbol search (exact + CONTAINS)
3. Fulltext search (cross-species)
4. GO annotations
5. Protein interactions
6. Co-expression partners
7. Regulatory network (incoming edges — who regulates gene X)
8. Orthologues (undirected match, specific species + all species)
9. Bulk query (multiple genes)
10. Schema introspection (labels, relationship types, species counts)

### 3. Graph API (REST)

Endpoint reference with one-line curl examples:

- `GET /api/graph/gene/{gene_id}` — gene details
- `GET /api/graph/search?q=&limit=&offset=&type=` — search (max 100)
- `GET /api/graph/gene/{gene_id}/neighbors?relationship_types=&direction=&limit=` — connected nodes
- `GET /api/graph/gene/{gene_id}/pathways` — pathway membership
- `GET /api/graph/gene/{gene_id}/stats` — counts summary
- `GET /api/graph/pathway/{pathway_id}` — pathway with genes/reactions/compounds
- `GET /api/graph/pathways?source=&q=&limit=` — list/search pathways
- `POST /api/graph/network/interactions` — `{"genes": [...], "min_confidence": 0.7, "include_neighbors": true}`
- `POST /api/graph/network/coexpression` — `{"genes": [...], "correlation_threshold": 0.8}`
- `POST /api/graph/network/regulatory` — `{"gene": "...", "include_targets": true, "include_regulators": true}`
- `GET /api/graph/stats` — graph-wide node/relationship counts by type
- `POST /api/graph/query` — raw Cypher `{"query": "...", "parameters": {}}`
- `POST /api/graph/query/natural` — NL → Cypher `{"question": "...", "limit": 10}` → interpretation, cypher, results, confidence
- `GET /api/v1/search/genes?q=&limit=&offset=` — multi-species gene search (undocumented, functional)
- `GET /api/v1/search/terms?q=` — term resolution (Arabidopsis symbols/IDs only)

### 4. Enrichment API

**Pattern:**
```
GET /api/enrichment/gene/{gene_id}                    # all sources (~30s)
GET /api/enrichment/gene/{gene_id}?sources=a,b,c      # selective (faster)
GET /api/enrichment/gene/{gene_id}/{source}            # single source
```

**42 sources by category** (one line each: source name, what it returns):

Protein & Structure:
- uniprot: protein function, PTMs, sequence, cross-refs
- alphafold: 3D structure, pLDDT confidence, model URLs
- interpro: domain architecture, functional sites
- panther: protein family/subfamily, cross-species orthologs
- pdbe: experimental structures (X-ray, NMR, cryo-EM)
- rcsb_pdb: structure data with quality metrics
- ebi_proteins: sequence features, variants, PTMs

Interactions:
- string: PPI with confidence scores (`score_threshold` param, default 0.4), evidence breakdown
- intact: experimentally validated interactions with methods
- biogrid: physical and genetic interactions

Pathways:
- kegg: pathways, modules, reactions (500+ Arabidopsis pathways)
- reactome: expert-curated with reaction mechanisms
- wikipathways: community-curated, 200+ plant species

Expression:
- expression_atlas: baseline/differential across 3000+ experiments (`experiment_type` param)
- bar_efp: tissue-specific expression visualization
- atted2: co-expression networks with correlation scores

Ontology:
- quickgo: GO terms with evidence codes and hierarchy
- chebi: chemical structures, SMILES/InChI, biological roles
- ols: 200+ EBI ontologies

Literature:
- europepmc: publications, 40M+ articles (`sort`: relevance/date/citations, `limit`)
- semantic_scholar: academic metadata, AI analysis, 200M+ papers
- crossref: bibliographic data, citation counts, 130M+ DOIs

Comparative:
- ensembl: orthologs, paralogs, gene trees (7 plant species)
- oma: orthology across 2100+ genomes
- gramene: cross-species mappings, 66 plant species
- plaza: gene families, synteny, 50+ plant genomes

Plant-specific:
- planttfdb: TF classification, DNA-binding domains (2000+ Ath TFs)
- suba: subcellular localization consensus + experimental evidence
- aranet: co-functional network (22K+ Ath genes)
- arapheno: phenotypic variation, QTL data, 1135 accessions
- thalemine: integrated gene info
- 1001genomes: SNPs/indels across 1135 accessions

Other:
- rnacentral: non-coding RNA (20M+ sequences)
- pubchem: molecular data, 100M+ compounds
- wikidata: linked data, gene-disease associations
- mygene: gene annotation aggregation

**Batch:**
```
POST /api/enrichment/batch
{"genes": ["AT1G01010", "AT1G01020"], "sources": ["uniprot", "string"], "parallel": true}
```

**Functional enrichment:**
- `POST /api/enrichment/analysis/go` — `{"genes": [...], "background": "genome", "ontology": "BP", "correction": "fdr"}`
- `POST /api/enrichment/analysis/pathway` — `{"genes": [...], "databases": ["kegg", "reactome"], "p_threshold": 0.05}`
- Legacy: `POST /api/v1/gene-enrichment/analyze` — Arabidopsis-only, min 2 genes, p_threshold max 0.5, categories: GO/PO/KEGG/MapMan

**Cache:** `GET /api/enrichment/cache/status/{gene_id}` — shows cached sources, age, expiry. 24h TTL.

### 5. AI API

**Question generation:**
- `POST /api/questions/generate-from-abstract` — `{"abstract": "...", "title": "...", "num_questions": 5, "use_rag": true, "use_graph": true}` → scored questions with graph_query_pattern, experimental_followups
- `POST /api/questions/generate-from-pmid` — `{"pmid": "...", "num_questions": 5}`
- `GET /api/questions/random-pmid` → `{pmid, title, abstract, year}`

**Chat:**
- `POST /api/chat/message` — `{"message": "...", "session_id": "..."}` → response, sources, suggested_actions
- `POST /api/chat/sessions` — create session
- `GET /api/chat/sessions` — list sessions
- `GET /api/chat/sessions/{id}/history` — conversation history
- `DELETE /api/chat/sessions/{id}` — delete session

**NL query & hypothesis:**
- `POST /api/ai/query` — `{"question": "...", "return_cypher": true}` → interpretation, cypher_query, results, confidence
- `POST /api/ai/hypothesis` — `{"genes": [...], "context": "...", "hypothesis_type": "..."}` → hypotheses with supporting_evidence, testable_predictions, confidence

**GNN predictions:**
- `GET /api/gnn/predict/{gene_id}` → predicted GO terms with probabilities
- `POST /api/gnn/predict/links` — `{"gene": "...", "relationship_type": "...", "threshold": 0.8, "limit": 10}` → predicted novel relationships

**Q&A Discovery loop:**
- `POST /api/qa-discovery/start` — `{"seed_question": "...", "max_iterations": 5, "depth": "deep"}` → session_id
- `GET /api/qa-discovery/{id}/status` → iterations_completed, novel_insights
- `GET /api/qa-discovery/{id}/results` → qa_pairs, discovered_genes, discovered_pathways, synthesis

### 6. Paper Evidence API

- `POST /api/v1/paper-evidence/check-abstract` — `{"abstract_text": "...", "paper_title": "..."}` → overall_score (0-100), score_tier, extracted entities, per-entity evidence breakdown (18 categories: GO, interactions, regulation, publications, expression, pathways, epigenetics, phenotypes, orthologs, protein_domains, etc.), data_sources_used. Processing: ~5-15s.
- `POST /api/v1/paper-evidence/verify-claims` (SSE) — `{"abstract_text": "...", "claims": [...]}` (claims optional, auto-generated if omitted). Streams: claim_extracted → entity_report → query_progress → claim_verified (verdict: supported/partially_supported/not_found, evidence narrative, raw Cypher results) → complete (overall_overlap_score). Processing: 30-70s.
- `POST /api/v1/paper-evidence/check` — multipart form `file` field, PDF only. Same response as check-abstract.
- `POST /api/v1/paper-evidence/verify-claims-pdf` (SSE) — multipart form PDF upload, same streaming response as verify-claims.

Note: These use `/api/v1/` prefix, not `/api/`.

### 7. Cypher Deep Reference

**Node labels (132)** — grouped by category:
- Gene-level: Locus, Transcript, Protein, Gene, SmallORF/sORF, CircRNA/CircularRNA, LncRNA/lncRNA
- Annotation: GO, PO, TO, BTO, PECO, ENVO, ProteinDomain, InterProDomain, MapManBin, Pathway, Enzyme, Reaction, MetabolicCompound
- Taxonomy: Species, NCBITaxon, GeneFamily, PhytozomeGeneFamily, ProteinFamily
- Regulation: TranscriptionFactorFamily, Promoter, BindingSite, BindingMotif, TFMotif, Motif, RegulatoryElement, ReMapPeak, PeakRegion, DHSRegion, ChromatinAccessibilityPeak, SignalTrack
- Expression: Tissue, CellType, ExpressionProfile, DiurnalProfile, DevelopmentalStage, SpatialRegion, SpatialZone, DiurnalCondition
- Experiment: Experiment, Study, Sample, GrowthCondition, Condition, StressCondition, StressExperiment, StressTimePoint, TimePoint, RnaSeqLibrary, MicroarrayPlatform, EBIAtlasExperiment, BARExperiment, GSE, GSM, MetaboLightsStudy
- Literature: Publication, Preprint, Author, Journal, MeSH
- Variants/GWAS: SNP, INDEL, GWASStudy, Trait, PlantTrait, AraphenoStudy, Germplasm
- Protein features: UniProt, ProteinComplex, Complex, ProteinStructure, ProteinFeature, ProteinLigand, Ligand, LigandBinding, LigandBindingSite, ProteomicsEvidence
- PTM sites: PhosphorylationSite, UbiquitinationSite, AcetylationSite, SUMOylationSite, SnitrosylationSite, SulfenylationSite, GlcNAcylationSite, MethylationSite, PTMSite
- Chemistry: ChEBICompound, KnapsackCompound, Metabolite, NaturalProduct, Phytochemical, PhytochemicalCompound, PhytohubCompound
- Methylation: MethylationContext, HistoneModification
- Pathogen: Pathogen, PathogenEffector, PathogenGene, PHIGene, PHIInteraction, PHIOrganism, PHIPOTerm, MicrobeInteraction
- Biosynthesis: BGC_Cluster, BGCluster, BiosynthesisCluster, BiosyntheticCluster (legacy dupes)
- Other: SubcellularCompartment, CellularCompartment, GeneXref, OntologyAnchor, SmallRNA, Hormone, Nutrient, Phenotype, ClimateAdaptation, Institution, PMNDatabase, TAIRLocus, TAIRObject, AppSettings, AuditLog, TestNode

**Relationship types (149)** — grouped, with edge counts for top 20:
- EXPRESSED_IN_CELL_TYPE: 22M
- ORTHOLOGOUS_TO: 18M
- ANNOTATED_WITH: 14M
- INTERACTS_WITH: 8M
- HAS_PROFILE: 7.6M
- EXPRESSED_IN: 7.4M
- BINDS_TO: 5.4M
- REGULATES: 4.4M
- COEXPRESSES_WITH: 3.8M
- DAPSEQ_BINDS_PROMOTER: 3.4M
- HAS_METHYLATION: 2.9M
- PARTICIPATES_IN: 2.7M
- HAS_MESH_TERM: 2.6M
- COFUNCTIONAL_WITH: 2.6M
- BINDS_PROMOTER: 2.4M
- (remaining types listed without counts)

Full relationship list grouped by category (same grouping as current SKILL.md but extended with newly discovered types).

**Relationship properties:**
- INTERACTS_WITH: `pubmed_ids`, `methods`, `source`, `evidence_codes`
- COEXPRESSES_WITH: `correlation_score` (always), `rank` (always), `dataset_source` (always), `source_entrez_id`, `target_entrez_id`, optionally `atted2_v13_lls`, `atted2_v13_rank`
- ORTHOLOGOUS_TO: `source` only (no score/confidence)
- ANNOTATED_WITH: `source`, `evidence` (GO evidence code: IDA, IMP, IEA, etc.)
- EXPRESSED_UNDER_STRESS: `log2fc` (indexed)
- DAPSEQ_BINDS_PROMOTER: `source` (indexed)
- NCBI_ANNOTATED_WITH: `evidence_code` (indexed)

**Node properties** (key entities):
- Locus (Arabidopsis): id, name, symbol, species, taxon_id, chromosome, start, end, strand, description, gene_type, is_tf, tf_family, is_arabidopsis, ncbi_gene_id, entrez_id, uniprot_id, pfam_ids (array), subcellular_location, peptideatlas_detected, ensembl_homolog_count, aranet_enriched, remap_peak_count, remap_tf_count, hundreds of embedded expression properties (atgen_*, circ_*, geod*_tpm, mtab*_tpm)
- Locus (non-Arabidopsis): id, species, taxon_id, description, description_source, source, is_arabidopsis (false), ortholog_ref_id, species-prefixed expression properties (bradi_mtab4401_*_tpm, etc.)
- GO: id, name, name_obo, namespace (molecular_function/biological_process/cellular_component), definition, synonyms, is_obsolete, enriched (boolean)
- Publication: pubmed_id, title, abstract, authors, year, journal, doi, pmc_id, citation_count, cited_by_count, is_open_access, full_text_mined, gene_count, research_concepts (WARNING: inconsistent type — sometimes List[str], sometimes semicolon-delimited string)
- Species: id (snake_case), name, common_name, taxon_id, source
- Pathway: id, name, kegg_name, kegg_category, kegg_organism, kegg_gene_count, kegg_compound_count, url

**Fulltext indexes (7):**
| Index | Label | Properties |
|---|---|---|
| gene_fulltext | Locus | symbol, name, description |
| go_fulltext | GO | name, definition |
| publication_fulltext | Publication | title, abstract |
| pathway_fulltext | Pathway | name, common_name |
| compound_fulltext | MetabolicCompound | name, common_name |
| enzyme_fulltext | Enzyme | common_name, id |
| reaction_fulltext | Reaction | common_name, id |

**Key indexes:** Locus.id (unique constraint), Locus.name, Locus.species, Locus.symbol, Locus.is_tf, Locus.tf_family, Locus.taxon_id, Locus.entrez_id, Locus.ncbi_gene_id, Locus.chromosome+start+end (composite), COEXPRESSES_WITH.correlation_score, EXPRESSED_UNDER_STRESS.log2fc

**Expanded query patterns** (beyond top 10):
- Filter GO by evidence code: `WHERE r.evidence IN ["IDA","IMP","IGI","IPI","IEP"]` (experimental only, exclude IEA)
- Filter co-expression by score: `WHERE r.correlation_score > 0.8`
- DAP-seq validated TF binding
- Histone marks
- Cell-type expression (single-cell)
- Nutrient/pathogen responses
- Publications
- Protein complex membership
- GWAS trait associations
- Multi-hop: non-Ath → Ath orthologue → interactions/annotations
- All annotations dump (all outgoing relationships)
- Spatial expression: `EXPRESSED_IN_REGION`, `LOCATED_IN_SPATIAL_REGION`
- Methylation: `HAS_METHYLATION`, `METHYLATED_IN_CONTEXT`
- Stress expression with LFC filter: `EXPRESSED_UNDER_STRESS` WHERE `r.log2fc > 1`

**Species list** — 127 species with locus counts (top 30 + note about full list). Confirmed: Pinus taeda (52K), Arabidopsis lyrata (52K), algae (Galdieria, Chara), mosses (Physcomitrium), liverworts (Marchantia), hornworts (Anthoceros). Not in graph: Hordeum vulgare, Brachypodium sylvaticum.

**Cypher best practices** — keep current list (NULL filtering, count{} not size(), EXISTS{}, elementId() not id(), coalesce(), LIMIT). Add: filter ANNOTATED_WITH by evidence code, filter COEXPRESSES_WITH by correlation_score.

**Common mistakes** — keep current list, add:
- `/api/` vs `/api/v1/` path confusion (paper-evidence uses v1)
- Expecting enrichment for non-Arabidopsis genes
- Not filtering ANNOTATED_WITH by evidence code (IEA floods results)
- Querying COEXPRESSES_WITH without score filter (3.8M edges)
- Assuming ORTHOLOGOUS_TO is Arabidopsis-anchored (it's cross-species)
- Publication.research_concepts type inconsistency

### 8. R/Python Integration

**R (httr2/jsonlite):**
```r
# Auth
plantgraph_login <- function(user, pass) {
  resp <- POST("https://plantgraph.se/api/auth/login",
    body = list(username = user, password = pass), encode = "json")
  content(resp)$access_token
}

# Cypher (works without token)
plantgraph_query <- function(cypher, token = NULL) {
  headers <- c("Content-Type" = "application/json")
  if (!is.null(token)) headers["Authorization"] <- paste("Bearer", token)
  resp <- POST("https://plantgraph.se/api/v1/graph/query",
    body = list(query = cypher), encode = "json",
    add_headers(.headers = headers))
  content(resp, simplifyVector = TRUE)$results
}

# REST gene lookup
plantgraph_gene <- function(gene_id, token) {
  resp <- GET(paste0("https://plantgraph.se/api/graph/gene/", gene_id),
    add_headers(Authorization = paste("Bearer", token)))
  content(resp, simplifyVector = TRUE)
}

# Enrichment
plantgraph_enrich <- function(gene_id, sources, token) {
  resp <- GET(paste0("https://plantgraph.se/api/enrichment/gene/", gene_id,
    "?sources=", paste(sources, collapse = ",")),
    add_headers(Authorization = paste("Bearer", token)))
  content(resp, simplifyVector = TRUE)
}
```

**Python (requests):**
```python
import requests

BASE = "https://plantgraph.se"

def login(user, pwd):
    r = requests.post(f"{BASE}/api/auth/login", json={"username": user, "password": pwd})
    return r.json()["access_token"]

def query(cypher, token=None):
    headers = {"Content-Type": "application/json"}
    if token: headers["Authorization"] = f"Bearer {token}"
    r = requests.post(f"{BASE}/api/v1/graph/query", json={"query": cypher}, headers=headers)
    return r.json()["results"]

def get_gene(gene_id, token):
    r = requests.get(f"{BASE}/api/graph/gene/{gene_id}", headers={"Authorization": f"Bearer {token}"})
    return r.json()

def enrich(gene_id, sources, token):
    r = requests.get(f"{BASE}/api/enrichment/gene/{gene_id}",
        params={"sources": ",".join(sources)},
        headers={"Authorization": f"Bearer {token}"})
    return r.json()
```

Rate limits: guest 60/hr (10/min burst), researcher 1000/hr (60/min burst). Enrichment cache: 24h TTL.

## Implementation Notes

- Preserve all battle-tested content from current SKILL.md (ID rules, Cypher patterns, common mistakes)
- Correct ORTHOLOGOUS_TO directionality description
- Update "No authentication required" → auth section
- Update "1.6M loci across 60+ species" → "14.5M+ nodes, 47.9M+ relationships, 127 species"
- All curl examples use `https://plantgraph.se` as base URL
- Keep Cypher best practices section (Neo4j 5+ patterns)
- No GDS/graph algorithms available (not installed on instance)
- `dbms.procedures()` not exposed (restricted instance)
