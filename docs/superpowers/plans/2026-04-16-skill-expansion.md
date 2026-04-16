# SKILL.md Expansion Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Expand SKILL.md from a Cypher-only reference (~330 lines) into a comprehensive PlantGraph API handbook (~900-1000 lines) covering auth, REST APIs, enrichment, AI features, paper evidence, and deep Cypher reference.

**Architecture:** Single file, layered structure (Approach B). Quick reference section up top handles 80% of queries. Deep reference sections follow. Claude stops reading when it has enough context.

**Tech Stack:** Markdown only. No build, no tests. Validation is manual curl against live API.

**Spec:** `docs/superpowers/specs/2026-04-16-skill-expansion-design.md`

---

### Task 1: Frontmatter & Auth Setup

**Files:**
- Modify: `SKILL.md:1-8` (frontmatter) and insert new auth section after line 8

- [ ] **Step 1: Rewrite frontmatter**

Replace lines 1-4 of SKILL.md with expanded frontmatter. Keep `name: plantgraph-database`. Expand `description` to include trigger keywords for all API categories:

```yaml
---
name: plantgraph-database
description: >-
  Use when querying PlantGraph (plantgraph.se) — a Neo4j knowledge graph and REST API platform
  for plant gene annotations, interactions, co-expression, orthologues, GO terms, pathways,
  protein structures, and enrichment data. Covers Arabidopsis, rice, Brachypodium, wheat, maize,
  poplar, and 120+ other plant species (14.5M+ nodes, 47.9M+ relationships). Also covers:
  real-time enrichment from 42 sources (UniProt, STRING, AlphaFold, KEGG, InterPro, Expression Atlas),
  AI-powered question generation and hypothesis generation, GNN function predictions, paper evidence
  verification, and natural language graph queries. Use when user mentions PlantGraph, or needs
  cross-species gene annotation, protein interactions, pathway enrichment, or literature evidence
  via a plant biology knowledge graph.
---
```

- [ ] **Step 2: Replace intro paragraph**

Replace the current intro (`Public API at plantgraph.se...`) with updated stats and add the auth section:

```markdown
# PlantGraph Knowledge Graph & API Platform

Public API at `plantgraph.se` (UPSC, Umea Plant Science Centre). 14.5M+ nodes, 47.9M+ relationships, 127 species, 132 node types, 149 relationship types. Integrates 140+ data sources with 42 real-time enrichment services.

## Authentication

**Base URL:** `https://plantgraph.se`

Two path prefixes exist: `/api/` (REST endpoints) and `/api/v1/` (Cypher query, search, paper evidence). Raw Cypher at `/api/v1/graph/query` works without auth. REST endpoints require JWT.

### JWT Login

```bash
curl -s -X POST https://plantgraph.se/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "YOUR_USER", "password": "YOUR_PASS"}'
# Returns: {"access_token": "eyJ...", "token_type": "bearer", "expires_in": 86400}
```

Use token in subsequent requests:
```bash
curl -s https://plantgraph.se/api/graph/gene/AT1G01010 \
  -H "Authorization: Bearer YOUR_TOKEN"
```

Refresh before expiry: `POST /api/auth/refresh` with current bearer header.

### Rate Limits

| Tier | Limit | Burst |
|---|---|---|
| Guest | 60/hour | 10/minute |
| Researcher | 1000/hour | 60/minute |

Exceeding limits returns `429 Too Many Requests`.
```

- [ ] **Step 3: Commit**

```bash
git add SKILL.md
git commit -m "skill: update frontmatter and add auth section"
```

---

### Task 2: Quick Reference — Endpoints Table

**Files:**
- Modify: `SKILL.md` — replace current "API Endpoints" table (lines 12-18 of current file) with comprehensive master table

- [ ] **Step 1: Replace endpoints table**

Replace the current 4-endpoint table and its curl/enrichment examples (everything from `## API Endpoints` through the enrichment format example, lines 12-33) with:

```markdown
## API Endpoints

### Path Prefixes

| Prefix | Auth | Notes |
|---|---|---|
| `/api/v1/` | Optional | Cypher query, search, paper evidence |
| `/api/` | Required (JWT) | REST graph, enrichment, AI, chat |

### Graph

| Endpoint | Method | Purpose |
|---|---|---|
| `/api/graph/gene/{gene_id}` | GET | Gene details (symbol, GO terms, aliases) |
| `/api/graph/search?q=&limit=&offset=` | GET | Search genes (max 100) |
| `/api/graph/gene/{gene_id}/neighbors` | GET | Connected nodes (filter by rel type, direction) |
| `/api/graph/gene/{gene_id}/pathways` | GET | Pathway membership |
| `/api/graph/gene/{gene_id}/stats` | GET | Counts (interactions, pathways, publications, GO, orthologs) |
| `/api/graph/pathway/{pathway_id}` | GET | Pathway with genes, reactions, compounds |
| `/api/graph/pathways?source=&q=` | GET | List/search pathways |
| `/api/graph/network/interactions` | POST | Interaction network for gene list |
| `/api/graph/network/coexpression` | POST | Co-expression network |
| `/api/graph/network/regulatory` | POST | Regulatory network (targets + regulators) |
| `/api/graph/stats` | GET | Graph-wide node/relationship counts |
| `/api/graph/query` | POST | Raw Cypher |
| `/api/graph/query/natural` | POST | Natural language -> Cypher -> results |
| `/api/v1/graph/query` | POST | Raw Cypher (no auth needed) |

### Search

| Endpoint | Method | Purpose |
|---|---|---|
| `/api/v1/search/terms?q=` | GET | Term resolution (Arabidopsis symbols/IDs only) |
| `/api/v1/search/genes?q=&limit=&offset=` | GET | Multi-species gene search (paginated) |

### Enrichment

| Endpoint | Method | Purpose |
|---|---|---|
| `/api/enrichment/gene/{gene_id}` | GET | All 42 sources (~30s) |
| `/api/enrichment/gene/{gene_id}?sources=a,b` | GET | Selective sources (faster) |
| `/api/enrichment/gene/{gene_id}/{source}` | GET | Single source |
| `/api/enrichment/batch` | POST | Multiple genes + sources in parallel |
| `/api/enrichment/analysis/go` | POST | GO enrichment analysis |
| `/api/enrichment/analysis/pathway` | POST | Pathway enrichment (KEGG, Reactome) |
| `/api/enrichment/cache/status/{gene_id}` | GET | Cache age and expiry per source |
| `/api/v1/gene-enrichment/analyze` | POST | Legacy GO enrichment (Ath only, min 2 genes) |

### AI

| Endpoint | Method | Purpose |
|---|---|---|
| `/api/questions/generate-from-abstract` | POST | Research questions from abstract text |
| `/api/questions/generate-from-pmid` | POST | Research questions from PubMed ID |
| `/api/questions/random-pmid` | GET | Random paper for exploration |
| `/api/chat/message` | POST | Chat message with sources |
| `/api/chat/sessions` | POST/GET | Create/list chat sessions |
| `/api/chat/sessions/{id}/history` | GET | Conversation history |
| `/api/ai/query` | POST | NL question -> Cypher -> results + confidence |
| `/api/ai/hypothesis` | POST | Hypothesis generation from gene list |
| `/api/gnn/predict/{gene_id}` | GET | GNN function predictions (GO + probabilities) |
| `/api/gnn/predict/links` | POST | Predict novel gene relationships |
| `/api/qa-discovery/start` | POST | Autonomous Q&A discovery loop |
| `/api/qa-discovery/{id}/status` | GET | Discovery loop progress |
| `/api/qa-discovery/{id}/results` | GET | Discovery results + synthesis |

### Paper Evidence

| Endpoint | Method | Purpose |
|---|---|---|
| `/api/v1/paper-evidence/check-abstract` | POST | Score abstract against graph (0-100) |
| `/api/v1/paper-evidence/verify-claims` | POST | Verify claims via Cypher (SSE stream) |
| `/api/v1/paper-evidence/check` | POST | PDF upload, same as check-abstract |
| `/api/v1/paper-evidence/verify-claims-pdf` | POST | PDF upload, streaming claim verification |

### Auth & Status

| Endpoint | Method | Purpose |
|---|---|---|
| `/api/auth/login` | POST | JWT login |
| `/api/auth/refresh` | POST | Token refresh |
| `/api/v1/status` | GET | Health check |
```

- [ ] **Step 2: Commit**

```bash
git add SKILL.md
git commit -m "skill: replace endpoint table with comprehensive API reference"
```

---

### Task 3: Quick Reference — ID Rules & Top 10 Cypher

**Files:**
- Modify: `SKILL.md` — keep existing ID format section largely intact, curate Cypher patterns to top 10

- [ ] **Step 1: Keep ID format section**

Keep the existing "ID Format by Species" section (lines 37-62 of current SKILL.md) exactly as-is, including the Phytozome conversion table and B. sylvaticum strategy. These are battle-tested and correct.

- [ ] **Step 2: Keep Key Node Labels and Relationship Types sections**

Keep the existing "Key Node Labels" and "Key Relationship Types" sections (lines 64-89) as-is for the quick reference. The deep reference (Task 7) will expand these.

- [ ] **Step 3: Keep Lookup Methods section**

Keep "Lookup Methods" section (lines 91-127) as-is — these are the ID lookup patterns that belong in quick reference.

- [ ] **Step 4: Curate Common Query Patterns to top 10**

Keep these patterns from current "Common Query Patterns" (lines 129-255):
1. Get GO annotations (line 131)
2. Get protein interactions (line 137)
3. Get co-expression partners (line 143)
4. Get regulatory network — incoming edges (line 149)
5. Get orthologues — specific species (line 202) — **FIX the intro text**: change "ORTHOLOGOUS_TO edges point FROM Arabidopsis outward" to "ORTHOLOGOUS_TO direction is an implementation detail, not Arabidopsis-anchored. Always use undirected match for cross-species lookups."
6. Get orthologues — all species with counts (line 208)
7. Find Arabidopsis orthologue FROM non-Ath gene — undirected (line 215)
8. Multi-hop: Brachypodium -> Arabidopsis -> interactions (line 226)
9. Bulk query: multiple genes (line 242)
10. Schema introspection (line 250)

Move remaining patterns (DAP-seq, GWAS, histone, cell-type, nutrient/pathogen, publications, complex membership, all-annotations) to the deep Cypher reference in Task 7.

- [ ] **Step 5: Commit**

```bash
git add SKILL.md
git commit -m "skill: curate quick reference Cypher patterns, fix ORTHOLOGOUS_TO docs"
```

---

### Task 4: Graph API (REST) Section

**Files:**
- Modify: `SKILL.md` — insert new section after the quick reference Cypher patterns

- [ ] **Step 1: Write Graph API section**

Insert after the quick reference patterns:

```markdown
## Graph API (REST)

All REST endpoints require JWT auth header. Base: `https://plantgraph.se/api/graph`.

### Gene lookup
```bash
curl -s https://plantgraph.se/api/graph/gene/AT1G01010 \
  -H "Authorization: Bearer $TOKEN"
# Returns: id, symbol, name, description, chromosome, start, end, strand, gene_type, aliases, go_terms
```

### Search genes
```bash
curl -s "https://plantgraph.se/api/graph/search?q=photosynthesis&limit=10" \
  -H "Authorization: Bearer $TOKEN"
# Returns: results[], total, limit, offset. Max limit: 100.
```

### Gene neighbors
```bash
curl -s "https://plantgraph.se/api/graph/gene/AT1G01010/neighbors?relationship_types=INTERACTS_WITH,COEXPRESSES_WITH&direction=both&limit=50" \
  -H "Authorization: Bearer $TOKEN"
```

### Gene pathways
```bash
curl -s https://plantgraph.se/api/graph/gene/AT1G01010/pathways \
  -H "Authorization: Bearer $TOKEN"
# Returns: pathways[] with id, name, source, gene_count
```

### Gene stats
```bash
curl -s https://plantgraph.se/api/graph/gene/AT1G01010/stats \
  -H "Authorization: Bearer $TOKEN"
# Returns: interaction_count, pathway_count, publication_count, go_term_count, ortholog_count
```

### Pathway details
```bash
curl -s https://plantgraph.se/api/graph/pathway/ath00901 \
  -H "Authorization: Bearer $TOKEN"
# Returns: id, name, description, source, genes[], reactions, compounds
```

### List/search pathways
```bash
curl -s "https://plantgraph.se/api/graph/pathways?source=KEGG&q=phenylpropanoid&limit=20" \
  -H "Authorization: Bearer $TOKEN"
```

### Interaction network
```bash
curl -s -X POST https://plantgraph.se/api/graph/network/interactions \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"genes":["AT1G01010","AT2G40150"],"include_neighbors":true,"min_confidence":0.7}'
# Returns: nodes[], edges[] with source, target, confidence, evidence
```

### Co-expression network
```bash
curl -s -X POST https://plantgraph.se/api/graph/network/coexpression \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"genes":["AT1G01010","AT2G40150"],"correlation_threshold":0.8,"limit":50}'
```

### Regulatory network
```bash
curl -s -X POST https://plantgraph.se/api/graph/network/regulatory \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"gene":"AT2G40150","include_targets":true,"include_regulators":true}'
```

### Graph statistics
```bash
curl -s https://plantgraph.se/api/graph/stats \
  -H "Authorization: Bearer $TOKEN"
# Returns: node counts by type, relationship counts by type
```

### Natural language query
```bash
curl -s -X POST https://plantgraph.se/api/graph/query/natural \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"question":"What genes interact with FLC?","limit":20}'
# Returns: interpretation, cypher (generated), results[], confidence
```

### Raw Cypher (no auth needed)
```bash
curl -s -X POST https://plantgraph.se/api/v1/graph/query \
  -H "Content-Type: application/json" \
  -d '{"query":"MATCH (l:Locus {name: \"AT2G40150\"}) RETURN l.name, l.description LIMIT 1"}'
```

### Multi-species gene search
```bash
curl -s "https://plantgraph.se/api/v1/search/genes?q=WRKY&limit=20&offset=0"
# Returns: genes[], count, total, has_more. Works cross-species. No auth needed.
```
```

- [ ] **Step 2: Commit**

```bash
git add SKILL.md
git commit -m "skill: add Graph API REST endpoints section"
```

---

### Task 5: Enrichment API Section

**Files:**
- Modify: `SKILL.md` — insert after Graph API section

- [ ] **Step 1: Write Enrichment API section**

```markdown
## Enrichment API

Real-time data from 42 external databases. All endpoints require JWT auth. Results cached 24 hours.

### Query patterns
```bash
# All sources (slow, ~30s)
curl -s https://plantgraph.se/api/enrichment/gene/AT1G01010 \
  -H "Authorization: Bearer $TOKEN"

# Selective sources (faster)
curl -s "https://plantgraph.se/api/enrichment/gene/AT1G01010?sources=uniprot,string,alphafold" \
  -H "Authorization: Bearer $TOKEN"

# Single source
curl -s https://plantgraph.se/api/enrichment/gene/AT1G01010/string \
  -H "Authorization: Bearer $TOKEN"
```

### Available Sources (42)

**Protein & Structure:**
`uniprot` (function, PTMs, sequence), `alphafold` (3D structure, pLDDT confidence), `interpro` (domain architecture), `panther` (protein families, cross-species orthologs), `pdbe` (experimental structures), `rcsb_pdb` (structure quality metrics), `ebi_proteins` (sequence features, variants)

**Interactions:**
`string` (PPI confidence scores, `score_threshold` param default 0.4), `intact` (experimentally validated, detection methods), `biogrid` (physical + genetic interactions)

**Pathways:**
`kegg` (500+ Ath pathways, modules, reactions), `reactome` (expert-curated, reaction mechanisms), `wikipathways` (community-curated, 200+ species)

**Expression:**
`expression_atlas` (baseline/differential, 3000+ experiments, `experiment_type` param), `bar_efp` (tissue-specific visualization), `atted2` (co-expression networks, correlation scores)

**Ontology:**
`quickgo` (GO terms, evidence codes, hierarchy), `chebi` (chemical structures, SMILES/InChI), `ols` (200+ EBI ontologies)

**Literature:**
`europepmc` (40M+ articles, `sort`: relevance/date/citations), `semantic_scholar` (200M+ papers, AI analysis), `crossref` (130M+ DOIs, citation counts)

**Comparative genomics:**
`ensembl` (orthologs, paralogs, gene trees), `oma` (orthology, 2100+ genomes), `gramene` (66 plant species), `plaza` (gene families, synteny, 50+ genomes)

**Plant-specific:**
`planttfdb` (TF classification, 2000+ Ath TFs), `suba` (subcellular localization consensus), `aranet` (co-functional network, 22K+ genes), `arapheno` (phenotypic variation, 1135 accessions), `thalemine` (integrated gene info), `1001genomes` (SNPs/indels, 1135 accessions)

**Other:**
`rnacentral` (ncRNA, 20M+ sequences), `pubchem` (100M+ compounds), `wikidata` (linked data), `mygene` (annotation aggregation)

### Batch enrichment
```bash
curl -s -X POST https://plantgraph.se/api/enrichment/batch \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"genes":["AT1G01010","AT1G01020","AT1G01030"],"sources":["uniprot","string"],"parallel":true}'
```

### GO enrichment analysis
```bash
curl -s -X POST https://plantgraph.se/api/enrichment/analysis/go \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"genes":["AT1G01010","AT1G01020","AT1G01030"],"background":"genome","ontology":"BP","correction":"fdr"}'
# Returns: enriched_terms[] with go_id, name, p_value, adjusted_p_value, fold_enrichment, genes
```

### Pathway enrichment
```bash
curl -s -X POST https://plantgraph.se/api/enrichment/analysis/pathway \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"genes":["AT1G01010","AT1G01020"],"databases":["kegg","reactome"],"p_threshold":0.05}'
```

### Legacy enrichment (Arabidopsis only)
```bash
curl -s -X POST https://plantgraph.se/api/v1/gene-enrichment/analyze \
  -H "Content-Type: application/json" \
  -d '{"gene_ids":["AT1G01010","AT1G01020"],"categories":["GO"],"p_threshold":0.05}'
# No auth needed. Min 2 genes. p_threshold max 0.5. Categories: GO, PO, KEGG, MapMan.
# Background size: 38,171 (Arabidopsis genome). Non-Ath IDs rejected as invalid.
```

### Cache status
```bash
curl -s https://plantgraph.se/api/enrichment/cache/status/AT1G01010 \
  -H "Authorization: Bearer $TOKEN"
# Returns: per-source cache status, age_hours, expiration. TTL: 24 hours.
```
```

- [ ] **Step 2: Commit**

```bash
git add SKILL.md
git commit -m "skill: add Enrichment API section with 42 sources"
```

---

### Task 6: AI API Section

**Files:**
- Modify: `SKILL.md` — insert after Enrichment API section

- [ ] **Step 1: Write AI API section**

```markdown
## AI API

AI-powered features for research question generation, hypothesis testing, and graph-aware chat. All endpoints require JWT auth.

### Generate research questions from abstract
```bash
curl -s -X POST https://plantgraph.se/api/questions/generate-from-abstract \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"abstract":"FLC represses flowering by...","title":"FLC regulation","num_questions":5,"use_rag":true,"use_graph":true}'
# Returns: questions[] with question, rationale, graph_query_pattern, experimental_followups, total_score
```

### Generate questions from PubMed ID
```bash
curl -s -X POST https://plantgraph.se/api/questions/generate-from-pmid \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"pmid":"12345678","num_questions":5}'
```

### Random paper for exploration
```bash
curl -s https://plantgraph.se/api/questions/random-pmid \
  -H "Authorization: Bearer $TOKEN"
# Returns: pmid, title, abstract, year
```

### Chat
```bash
# Send message (creates session if session_id omitted)
curl -s -X POST https://plantgraph.se/api/chat/message \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"message":"What is known about FLC regulation?","session_id":"optional-id"}'
# Returns: response, session_id, sources[], suggested_actions

# List sessions
curl -s https://plantgraph.se/api/chat/sessions -H "Authorization: Bearer $TOKEN"

# Get history
curl -s https://plantgraph.se/api/chat/sessions/SESSION_ID/history -H "Authorization: Bearer $TOKEN"
```

### Natural language graph query
```bash
curl -s -X POST https://plantgraph.se/api/ai/query \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"question":"What transcription factors regulate FLC?","return_cypher":true}'
# Returns: interpretation, cypher_query, results[], confidence, suggestions
```

### Hypothesis generation
```bash
curl -s -X POST https://plantgraph.se/api/ai/hypothesis \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"genes":["AT5G10140","AT1G69120"],"context":"flowering time regulation","hypothesis_type":"mechanism"}'
# Returns: hypotheses[] with statement, supporting_evidence, testable_predictions, confidence
```

### GNN function predictions
```bash
# Predict gene functions
curl -s https://plantgraph.se/api/gnn/predict/AT1G01010 \
  -H "Authorization: Bearer $TOKEN"
# Returns: predictions[] with go_term, name, probability, evidence

# Predict novel relationships
curl -s -X POST https://plantgraph.se/api/gnn/predict/links \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"gene":"AT1G01010","relationship_type":"INTERACTS_WITH","threshold":0.8,"limit":10}'
```

### Q&A Discovery loop
```bash
# Start autonomous exploration
curl -s -X POST https://plantgraph.se/api/qa-discovery/start \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"seed_question":"What regulates vernalization in Arabidopsis?","max_iterations":5,"depth":"deep"}'
# Returns: session_id

# Check progress
curl -s https://plantgraph.se/api/qa-discovery/SESSION_ID/status \
  -H "Authorization: Bearer $TOKEN"
# Returns: status, iterations_completed, questions_answered, knowledge_graph_nodes, novel_insights

# Get results
curl -s https://plantgraph.se/api/qa-discovery/SESSION_ID/results \
  -H "Authorization: Bearer $TOKEN"
# Returns: qa_pairs[], discovered_genes, discovered_pathways, synthesis
```
```

- [ ] **Step 2: Commit**

```bash
git add SKILL.md
git commit -m "skill: add AI API section (questions, chat, hypothesis, GNN, discovery)"
```

---

### Task 7: Paper Evidence API Section

**Files:**
- Modify: `SKILL.md` — insert after AI API section

- [ ] **Step 1: Write Paper Evidence section**

```markdown
## Paper Evidence API

Verify research claims against the knowledge graph. These endpoints use `/api/v1/` prefix. No auth currently required.

### Check abstract
```bash
curl -s -X POST https://plantgraph.se/api/v1/paper-evidence/check-abstract \
  -H "Content-Type: application/json" \
  -d '{"abstract_text":"FLC is a MADS-box transcription factor that represses flowering.","paper_title":"FLC regulation"}'
# Returns: overall_score (0-100), score_tier, total_entities_extracted, total_entities_in_graph,
#   entities[] (per-entity: genes, go_terms, interactions, regulation, publications, expression,
#   pathways, epigenetics, phenotypes, orthologs, protein_domains, localization, tf_families,
#   stress_data, diurnal, metabolism, genetic_variation, total_connections),
#   evidence_summary, data_sources_used (18 sources), processing_time_ms
```

### Verify claims (streaming SSE)
```bash
curl -s -X POST https://plantgraph.se/api/v1/paper-evidence/verify-claims \
  -H "Content-Type: application/json" \
  -d '{"abstract_text":"FLC represses flowering by binding to SOC1."}'
# Claims auto-generated if omitted. Supply explicit claims:
# -d '{"abstract_text":"...","claims":["FLC represses flowering","FLC binds SOC1 promoter"]}'
#
# SSE event stream:
#   type:claim_extracted  -> claim_id, claim_text, claim_type, entities, confidence
#   type:entity_report    -> full check-abstract-style report
#   type:claim_verified   -> verdict (supported|partially_supported|not_found),
#                            plantgraph_evidence (narrative), evidence_data (raw Cypher results)
#   type:complete         -> overall_overlap_score, all claims, time_saved_estimate
# Processing: 30-70s
```

### PDF upload
```bash
# Check PDF
curl -s -X POST https://plantgraph.se/api/v1/paper-evidence/check \
  -F "file=@paper.pdf"
# Same response as check-abstract. Extracts text from PDF. Only PDF format accepted.

# Verify claims from PDF (streaming SSE)
curl -s -X POST https://plantgraph.se/api/v1/paper-evidence/verify-claims-pdf \
  -F "file=@paper.pdf"
# Same SSE stream as verify-claims.
```
```

- [ ] **Step 2: Commit**

```bash
git add SKILL.md
git commit -m "skill: add Paper Evidence API section"
```

---

### Task 8: Cypher Deep Reference

**Files:**
- Modify: `SKILL.md` — restructure and expand existing Cypher content below the API sections

- [ ] **Step 1: Write expanded node labels section**

Replace the current "Key Node Labels" quick-reference copy with a comprehensive deep reference. The quick reference (Task 3) keeps the abbreviated version; this section is the full inventory:

```markdown
## Cypher Deep Reference

### Node Labels (132)

**Gene-level:** `Locus`, `Transcript`, `Protein`, `Gene`, `SmallORF`/`sORF`, `CircRNA`/`CircularRNA`, `LncRNA`/`lncRNA`, `SmallRNA`

**Annotation:** `GO`, `PO`, `TO`, `BTO`, `PECO`, `ENVO`, `ProteinDomain`, `InterProDomain`, `MapManBin`, `Pathway`, `Enzyme`, `Reaction`, `MetabolicCompound`

**Taxonomy:** `Species`, `NCBITaxon`, `GeneFamily`, `PhytozomeGeneFamily`, `ProteinFamily`

**Regulation:** `TranscriptionFactorFamily`, `Promoter`, `BindingSite`, `BindingMotif`, `TFMotif`, `Motif`, `RegulatoryElement`, `ReMapPeak`, `PeakRegion`, `DHSRegion`, `ChromatinAccessibilityPeak`, `SignalTrack`

**Expression:** `Tissue`, `CellType`, `ExpressionProfile` (7.3M nodes), `DiurnalProfile`, `DevelopmentalStage`, `SpatialRegion`, `SpatialZone`, `DiurnalCondition`

**Experiment:** `Experiment`, `Study`, `Sample`, `GrowthCondition`, `Condition`, `StressCondition`, `StressExperiment`, `StressTimePoint`, `TimePoint`, `RnaSeqLibrary`, `MicroarrayPlatform`, `EBIAtlasExperiment`, `BARExperiment`, `GSE`, `GSM`, `MetaboLightsStudy`

**Literature:** `Publication` (290K), `Preprint`, `Author` (105K), `Journal`, `MeSH`

**Variants/GWAS:** `SNP` (1.3M), `INDEL`, `GWASStudy`, `Trait`, `PlantTrait`, `AraphenoStudy`, `Germplasm`

**Protein features:** `UniProt`, `ProteinComplex`, `Complex`, `ProteinStructure`, `ProteinFeature`, `ProteinLigand`, `Ligand`, `LigandBinding`, `LigandBindingSite`, `ProteomicsEvidence`

**PTM sites:** `PhosphorylationSite` (190K), `UbiquitinationSite`, `AcetylationSite`, `SUMOylationSite`, `SnitrosylationSite`, `SulfenylationSite`, `GlcNAcylationSite`, `MethylationSite` (2.9M), `PTMSite`

**Chemistry:** `ChEBICompound`, `KnapsackCompound`, `Metabolite`, `NaturalProduct`, `Phytochemical`, `PhytochemicalCompound`, `PhytohubCompound`

**Epigenetics:** `MethylationContext`, `HistoneModification` (154K)

**Pathogen:** `Pathogen`, `PathogenEffector`, `PathogenGene`, `PHIGene`, `PHIInteraction`, `PHIOrganism`, `PHIPOTerm`, `MicrobeInteraction`

**Biosynthesis:** `BGC_Cluster`, `BGCluster`, `BiosynthesisCluster`, `BiosyntheticCluster` (legacy duplicates)

**Other:** `SubcellularCompartment`, `CellularCompartment`, `GeneXref`, `OntologyAnchor`, `Hormone`, `Nutrient`, `Phenotype`, `ClimateAdaptation`, `Institution`, `PMNDatabase`, `TAIRLocus`, `TAIRObject`
```

- [ ] **Step 2: Write relationship types with counts**

```markdown
### Relationship Types (149) — Top 20 by Count

| Relationship | Count | Notes |
|---|---|---|
| EXPRESSED_IN_CELL_TYPE | 22.1M | Single-cell expression |
| ORTHOLOGOUS_TO | 18.0M | Cross-species, undirected match recommended |
| ANNOTATED_WITH | 14.2M | GO terms, filter by evidence code |
| INTERACTS_WITH | 8.2M | PPI |
| HAS_PROFILE | 7.7M | Expression profiles |
| EXPRESSED_IN | 7.4M | Tissue/organ expression |
| BINDS_TO | 5.4M | TF binding |
| REGULATES | 4.4M | Regulatory edges |
| COEXPRESSES_WITH | 3.8M | Filter by correlation_score |
| DAPSEQ_BINDS_PROMOTER | 3.4M | Gold-standard TF binding |
| HAS_METHYLATION | 2.9M | DNA methylation |
| PARTICIPATES_IN | 2.7M | Pathway membership |
| HAS_MESH_TERM | 2.6M | Publication MeSH |
| COFUNCTIONAL_WITH | 2.6M | AraNet |
| BINDS_PROMOTER | 2.4M | Promoter binding |
| METHYLATED_IN_CONTEXT | 1.6M | CpG/CHG/CHH context |
| ANNOTATED_WITH_PO | 1.6M | Plant Ontology |
| HAS_MAPMAN_BIN | 1.6M | MapMan functional bins |
| HAS_DOMAIN | 1.5M | Protein domains |
| IN_TAXON | 1.5M | Taxonomic assignment |

**LIMIT aggressively** on high-count relationships. `INTERACTS_WITH` and `COEXPRESSES_WITH` can return 100K+ rows for well-connected genes.

**Full relationship list by category:**

**Core biology:** `INTERACTS_WITH`, `COEXPRESSES_WITH`, `COFUNCTIONAL_WITH`, `ORTHOLOGOUS_TO`, `PARALOGOUS_TO`

**Annotation:** `ANNOTATED_WITH`, `ANNOTATED_WITH_PO`, `NCBI_ANNOTATED_WITH`, `HAS_DOMAIN`, `HAS_INTERPRO_DOMAIN`, `HAS_PROTEIN_DOMAIN`, `HAS_MAPMAN_BIN`, `DOMAIN_GO_ANNOTATION`

**Regulation:** `ACTIVATES`, `REPRESSES`, `REGULATES`, `BINDS_TO`, `BINDS_PROMOTER`, `DAPSEQ_BINDS_PROMOTER`, `TARGETS`, `HAS_BINDING_MOTIF`, `HAS_BINDING_SITE`, `HAS_MOTIF`, `HAS_JASPAR_MOTIF`, `HAS_REPRESENTATIVE_MOTIF`, `BINDS_AT`

**Membership:** `MEMBER_OF`, `MEMBER_OF_TF_FAMILY`, `MEMBER_OF_GENE_FAMILY`, `MEMBER_OF_COMPLEX`, `MEMBER_OF_CLUSTER`, `PART_OF_COMPLEX`, `PART_OF_CLUSTER`, `PARTICIPATES_IN`, `BELONGS_TO_PATHWAY`, `HAS_SUBPATHWAY`, `IN_CLUSTER`, `PARENT_BIN`

**Expression:** `EXPRESSED_IN`, `EXPRESSED_IN_TISSUE`, `EXPRESSED_IN_CELL_TYPE`, `EXPRESSED_IN_REGION`, `EXPRESSED_IN_ZONE`, `EXPRESSED_UNDER_STRESS`, `EXPRESSED_UNDER_CONDITION`, `EXPRESSED_AT_TIMEPOINT`, `HAS_PROFILE`, `HAS_DIURNAL_PROFILE`, `HAS_RHYTHMIC_MEMBERS`, `REPRESENTS_STAGE`

**Spatial:** `LOCATED_IN_SPATIAL_REGION`, `HAS_SPATIAL_LOCATION`, `LOCATED_IN`, `LOCALIZED_TO`, `LOCALIZES_TO`

**Responses:** `RESPONDS_TO_HORMONE`, `RESPONDS_TO_NUTRIENT`, `RESPONDS_TO_PATHOGEN`, `AFFECTED_BY_CONDITION`, `UNDER_CONDITION`, `HAS_CONDITION`

**Life history:** `INVOLVED_IN_FLOWERING`, `CITED_IN_FLOWERING`, `INVOLVED_IN_ADAPTATION`, `INVOLVED_IN_SYMBIOSIS`

**GWAS/traits:** `ASSOCIATED_WITH_TRAIT`, `ASSOCIATED_WITH`, `IS_A_TRAIT`, `MEASURES_TRAIT`, `GWAS_OF`, `HAS_PHENOTYPE`

**Epigenetics:** `HAS_HISTONE_MARK`, `HAS_METHYLATION`, `METHYLATED_IN_CONTEXT`, `HAS_METHYLATION_SITE`, `HAS_DHS_REGION`, `IN_ACCESSIBLE_REGION`

**PTMs:** `HAS_PHOSPHORYLATION_SITE`, `HAS_PHOSPHOSITE`, `HAS_UBIQUITINATION_SITE`, `HAS_ACETYLATION_SITE`, `HAS_SUMOYLATION_SITE`, `HAS_SNITROSYLATION_SITE`, `HAS_SULFENYLATION_SITE`, `HAS_GLCNACYLATION_SITE`, `HAS_MYRISTOYLATION`, `HAS_PTM_SITE`

**Protein:** `MAPS_TO`, `ENCODES`, `HAS_FEATURE`, `HAS_STRUCTURE`, `BINDS_LIGAND`, `HAS_LIGAND_BINDING`, `HAS_MS_EVIDENCE`, `HAS_PROTEOMICS_EVIDENCE`, `TRANSLATED_FROM`, `EXPRESSED_AS`, `HAS_TRANSCRIPT`, `ENCODES_ENZYME`, `ENCODES_LNCRNA`, `ENCODES_SORF`, `HAS_SMALL_ORF`

**Pathogen:** `HAS_PATHOGEN`, `TARGETS_HOST_PROTEIN`, `VIRULENCE_GENE_OF`, `HAS_HOST`, `FOUND_IN_ORGANISM`, `INVOLVES_GENE`

**Metabolism:** `CATALYZES`, `CATALYZES_REACTION`, `HAS_REACTION`, `CONSUMES`, `PRODUCES`, `METABOLITE_IN`

**Literature:** `PUBLISHED_IN`, `MENTIONED_IN`, `CITED_BY`, `CITES`, `CITED_IN_FLOWERING`, `HAS_AUTHOR`, `COAUTHORED_WITH`, `HAS_MESH_TERM`, `LATER_PUBLISHED_AS`, `WORKS_ON_GENE`

**Variant:** `HAS_VARIANT`, `NEAR_VARIANT`, `OVERLAPS_WITH`

**Cross-reference:** `SAME_AS`, `DERIVED_FROM`, `DERIVES_FROM`, `CONNECTED_TO_GRAPH`, `HAS_TAIR_LOCUS`, `HAS_TAIR_OBJECT`, `ONTOLOGY_ANCHOR`, `FROM_DATABASE`, `SUPPORTED_BY`, `DETECTED_BY`, `MARKER_OF`, `MEASURED`, `MEASURED_IN`, `MEASURED_UNDER`, `OBSERVED_IN`, `HAS_EXPERIMENT`, `PART_OF_EXPERIMENT`, `HAS_SAMPLE`, `HAS_PECO_TERM`, `HAS_TO_TERM`, `IN_TAXON`, `PARENT_OF`, `IS_A`
```

- [ ] **Step 3: Write relationship properties section**

```markdown
### Relationship Properties

Properties on edges — use these for filtering and context.

| Relationship | Properties | Notes |
|---|---|---|
| INTERACTS_WITH | `pubmed_ids`, `methods`, `source`, `evidence_codes` | Filter by source or evidence |
| COEXPRESSES_WITH | `correlation_score`, `rank`, `dataset_source`, `source_entrez_id`, `target_entrez_id` | Always has score/rank. Optional: `atted2_v13_lls`, `atted2_v13_rank` |
| ANNOTATED_WITH | `source`, `evidence` | GO evidence codes: IDA, IMP, IGI, IPI, IEP (experimental), IEA (computational) |
| ORTHOLOGOUS_TO | `source` | No score or confidence — just source DB |
| EXPRESSED_UNDER_STRESS | `log2fc` | Indexed — fast range queries |
| DAPSEQ_BINDS_PROMOTER | `source` | Indexed |
| NCBI_ANNOTATED_WITH | `evidence_code` | Indexed |

#### Filter GO to experimental evidence only
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[r:ANNOTATED_WITH]->(go:GO)
WHERE r.evidence IN ["IDA","IMP","IGI","IPI","IEP"]
RETURN go.id, go.name, r.evidence
```

#### Filter co-expression by score
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[r:COEXPRESSES_WITH]->(c:Locus)
WHERE r.correlation_score > 0.8
RETURN c.name, c.symbol, r.correlation_score ORDER BY r.correlation_score DESC LIMIT 20
```

#### Filter stress expression by fold change
```cypher
MATCH (l:Locus {name: "AT1G01010"})-[r:EXPRESSED_UNDER_STRESS]->(s)
WHERE r.log2fc > 1
RETURN s.name, r.log2fc ORDER BY r.log2fc DESC
```
```

- [ ] **Step 4: Write node properties section**

```markdown
### Node Properties (Key Entities)

**Locus (Arabidopsis):** `id`, `name` (AT ID), `symbol`, `species`, `taxon_id`, `chromosome`, `start`, `end`, `strand`, `description`, `gene_type`, `is_tf` (boolean), `tf_family`, `is_arabidopsis` (true), `ncbi_gene_id`, `entrez_id`, `uniprot_id`, `pfam_ids` (array), `subcellular_location`, `peptideatlas_detected`, `ensembl_homolog_count`, `aranet_enriched`, `remap_peak_count`, `remap_tf_count`. Plus hundreds of embedded expression properties: `atgen_drought_*`, `atgen_salt_*`, `atgen_cold_*` (ATGenExpress TPMs), `circ_emexp1304_*` (circadian), `geod*_tpm`/`geod*_lfc` (GEO datasets), `mtab*_tpm` (ArrayExpress).

**Locus (non-Arabidopsis):** `id`, `species`, `taxon_id`, `description`, `description_source`, `source`, `is_arabidopsis` (false), `ortholog_ref_id`. Expression properties are species-prefixed: `bradi_mtab4401_*_tpm`, `bradi_mean_tpm`, `bradi_max_tpm`, etc.

**GO:** `id`, `name`, `name_obo`, `namespace` (molecular_function/biological_process/cellular_component), `definition`, `synonyms` (array), `is_obsolete` (boolean), `enriched` (boolean)

**Publication:** `pubmed_id`, `title`, `abstract`, `authors`, `year`, `journal`, `doi`, `pmc_id`, `citation_count`, `cited_by_count`, `is_open_access`, `full_text_mined`, `gene_count`. WARNING: `research_concepts` has inconsistent type — sometimes `List[str]`, sometimes semicolon-delimited `string`. Handle both.

**Species:** `id` (snake_case), `name`, `common_name`, `taxon_id`, `source`

**Pathway:** `id`, `name`, `kegg_name`, `kegg_category`, `kegg_organism`, `kegg_gene_count`, `kegg_compound_count`, `url`
```

- [ ] **Step 5: Write fulltext indexes and key indexes**

```markdown
### Fulltext Indexes (7)

| Index | Label | Properties | Example |
|---|---|---|---|
| `gene_fulltext` | Locus | symbol, name, description | `CALL db.index.fulltext.queryNodes("gene_fulltext", "trichome birefringence") YIELD node, score` |
| `go_fulltext` | GO | name, definition | `CALL db.index.fulltext.queryNodes("go_fulltext", "kinase activity") YIELD node, score` |
| `publication_fulltext` | Publication | title, abstract | `CALL db.index.fulltext.queryNodes("publication_fulltext", "vernalization FLC") YIELD node, score` |
| `pathway_fulltext` | Pathway | name, common_name | |
| `compound_fulltext` | MetabolicCompound | name, common_name | |
| `enzyme_fulltext` | Enzyme | common_name, id | |
| `reaction_fulltext` | Reaction | common_name, id | |

### Key Indexes

Fast lookups exist for: `Locus.id` (unique constraint), `Locus.name`, `Locus.species`, `Locus.symbol`, `Locus.is_tf`, `Locus.tf_family`, `Locus.taxon_id`, `Locus.entrez_id`, `Locus.ncbi_gene_id`, `Locus.chromosome+start+end` (composite). Relationship indexes: `COEXPRESSES_WITH.correlation_score`, `EXPRESSED_UNDER_STRESS.log2fc`, `INTERACTS_WITH.source`, `DAPSEQ_BINDS_PROMOTER.source`.
```

- [ ] **Step 6: Write expanded Cypher patterns**

The patterns that were moved out of quick reference in Task 3:

```markdown
### Additional Query Patterns

#### DAP-seq validated TF binding (gold standard)
```cypher
MATCH (tf:Locus)-[:DAPSEQ_BINDS_PROMOTER]->(l:Locus {name: "AT2G40150"})
RETURN tf.name, tf.symbol
```

#### GWAS trait associations
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[:ASSOCIATED_WITH_TRAIT]->(t)
RETURN t.name AS trait
```

#### Protein complex membership
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[:MEMBER_OF_COMPLEX|PART_OF_COMPLEX]->(c)
RETURN c.name AS complex_name
```

#### Nutrient/pathogen responses
```cypher
MATCH (l:Locus {name: "AT1G02860"})-[:RESPONDS_TO_NUTRIENT]->(n)
RETURN n.name AS nutrient
MATCH (l:Locus {name: "AT1G01190"})-[:RESPONDS_TO_PATHOGEN]->(p)
RETURN p.name AS pathogen
```

#### Publications mentioning a gene
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[:PUBLISHED_IN|MENTIONED_IN]->(pub:Publication)
RETURN pub.title, pub.pubmed_id, pub.year ORDER BY pub.year DESC LIMIT 10
```

#### Histone marks
```cypher
MATCH (l:Locus {name: "AT3G20810"})-[:HAS_HISTONE_MARK]->(h)
RETURN h.id LIMIT 10
```

#### Cell-type expression (single-cell)
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[:EXPRESSED_IN_CELL_TYPE]->(ct:CellType)
RETURN ct.name LIMIT 10
```

#### Spatial expression
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[:EXPRESSED_IN_REGION]->(r:SpatialRegion)
RETURN r.name LIMIT 10
```

#### Methylation context
```cypher
MATCH (l:Locus {name: "AT1G01010"})-[:HAS_METHYLATION]->(m)
RETURN m.id LIMIT 10
MATCH (l:Locus {name: "AT1G01010"})-[:METHYLATED_IN_CONTEXT]->(mc:MethylationContext)
RETURN mc.id LIMIT 10
```

#### All annotations for a gene (relationship + target dump)
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[r]->(t)
RETURN type(r) AS rel, labels(t)[0] AS target_type,
       coalesce(t.name, t.id) AS target, t.description
LIMIT 50
```
```

- [ ] **Step 7: Commit**

```bash
git add SKILL.md
git commit -m "skill: add Cypher deep reference (labels, relationships, properties, indexes, patterns)"
```

---

### Task 9: Species List, Locus Properties, Best Practices, Common Mistakes

**Files:**
- Modify: `SKILL.md` — update existing sections with new data

- [ ] **Step 1: Update Locus Properties section**

Replace the current "Locus Properties (Arabidopsis)" section (lines 257-266) with a shorter note pointing to the deep reference:

```markdown
### Locus Properties

See "Node Properties (Key Entities)" above for full property lists. Key points:
- Arabidopsis loci carry hundreds of embedded expression values as flat properties (`atgen_*`, `circ_*`, `geod*_tpm`, `mtab*_tpm`)
- Non-Arabidopsis loci have species-prefixed expression (`bradi_mtab4401_*_tpm`, etc.)
- Boolean flags: `is_tf`, `is_arabidopsis`, `aranet_enriched`, `peptideatlas_detected`
```

- [ ] **Step 2: Update Species section**

Replace current "Species in Graph" section (lines 268-272) with expanded data:

```markdown
### Species in Graph (127 species, 1.62M loci)

Top 20 by locus count:

| Species | Loci | | Species | Loci |
|---|---|---|---|---|
| Brassica napus | 149K | | Salix purpurea | 35K |
| Triticum aestivum | 108K | | Populus trichocarpa | 35K |
| Brassica oleracea | 80K | | Solanum pennellii | 34K |
| Glycine max | 61K | | Medicago truncatula | 33K |
| Pinus taeda | 52K | | Quercus lobata | 33K |
| Arabidopsis lyrata | 52K | | Solanum lycopersicum | 32K |
| Arabidopsis thaliana | 44K | | Zea mays | 30K |
| Phaseolus vulgaris | 42K | | Oryza sativa subsp. japonica | 22K |
| Juglans regia | 40K | | Sorghum bicolor | 16K |
| Brassica rapa | 39K | | Brachypodium distachyon | 15K |

Also includes: 9 additional Oryza species, conifers (Picea abies), mosses (Physcomitrium patens), ferns (Selaginella moellendorffii), algae (Galdieria sulphuraria, Chara braunii), liverworts (Marchantia polymorpha), hornworts (Anthoceros angustus).

**Not in graph:** Hordeum vulgare, Brachypodium sylvaticum. Use orthologue chains via Brachypodium distachyon or Arabidopsis.

**Rice note:** `"Oryza sativa"` (13K loci) and `"Oryza sativa subsp. japonica"` (22K loci) are separate entries — check both.
```

- [ ] **Step 3: Update Cypher Best Practices**

Keep the existing best practices section, add two items:

```markdown
- **Filter ANNOTATED_WITH by evidence**: `WHERE r.evidence IN ["IDA","IMP","IGI","IPI","IEP"]` to exclude IEA (computational, floods results)
- **Filter COEXPRESSES_WITH by score**: `WHERE r.correlation_score > 0.8` — 3.8M edges without filter
```

- [ ] **Step 4: Update Common Mistakes**

Keep the existing mistakes list. Add:

```markdown
- Using `/api/` path for paper-evidence endpoints (they use `/api/v1/` prefix)
- Expecting enrichment API to work with non-Arabidopsis gene IDs (GO enrichment is Arabidopsis-only, background = 38,171 genes)
- Not filtering `ANNOTATED_WITH` by `r.evidence` code — IEA (inferred from electronic annotation) floods results with low-confidence terms
- Querying `COEXPRESSES_WITH` without `r.correlation_score` filter — 3.8M edges will be slow/huge
- Assuming `ORTHOLOGOUS_TO` only radiates from Arabidopsis — it connects all species pairs in both directions. Direction is an implementation detail. Always use undirected match.
- Reading `Publication.research_concepts` without type checking — sometimes `List[str]`, sometimes semicolon-delimited `string`
```

- [ ] **Step 5: Commit**

```bash
git add SKILL.md
git commit -m "skill: update species list, best practices, common mistakes"
```

---

### Task 10: R/Python Integration

**Files:**
- Modify: `SKILL.md` — replace current R section, add Python

- [ ] **Step 1: Replace R section and add Python**

Replace current "R Integration" section (lines 274-295) with:

```markdown
## R Integration (httr/jsonlite)

```r
library(httr)
library(jsonlite)

# Auth
plantgraph_login <- function(user, pass) {
  resp <- POST("https://plantgraph.se/api/auth/login",
    body = list(username = user, password = pass), encode = "json",
    content_type_json())
  content(resp)$access_token
}

# Raw Cypher (works without token)
plantgraph_query <- function(cypher, token = NULL) {
  h <- add_headers("Content-Type" = "application/json")
  if (!is.null(token)) h <- add_headers(Authorization = paste("Bearer", token))
  resp <- POST("https://plantgraph.se/api/v1/graph/query",
    body = list(query = cypher), encode = "json", h, content_type_json())
  content(resp, as = "parsed", simplifyVector = TRUE)$results
}

# REST gene lookup
plantgraph_gene <- function(gene_id, token) {
  resp <- GET(paste0("https://plantgraph.se/api/graph/gene/", gene_id),
    add_headers(Authorization = paste("Bearer", token)))
  content(resp, as = "parsed", simplifyVector = TRUE)
}

# Enrichment (selective sources)
plantgraph_enrich <- function(gene_id, sources, token) {
  resp <- GET(paste0("https://plantgraph.se/api/enrichment/gene/", gene_id,
    "?sources=", paste(sources, collapse = ",")),
    add_headers(Authorization = paste("Bearer", token)))
  content(resp, as = "parsed", simplifyVector = TRUE)
}

# GO enrichment analysis
plantgraph_go_enrichment <- function(gene_ids, token, ontology = "BP") {
  resp <- POST("https://plantgraph.se/api/enrichment/analysis/go",
    body = list(genes = gene_ids, background = "genome",
                ontology = ontology, correction = "fdr"),
    encode = "json", content_type_json(),
    add_headers(Authorization = paste("Bearer", token)))
  content(resp, as = "parsed", simplifyVector = TRUE)
}

# Example
token <- plantgraph_login("user", "pass")
plantgraph_query('MATCH (l:Locus {name: "AT2G40150"})-[:ANNOTATED_WITH]->(go:GO) RETURN go.id, go.name')
plantgraph_enrich("AT1G01010", c("uniprot", "string", "alphafold"), token)
```

## Python Integration (requests)

```python
import requests

BASE = "https://plantgraph.se"

def plantgraph_login(user, pwd):
    r = requests.post(f"{BASE}/api/auth/login", json={"username": user, "password": pwd})
    return r.json()["access_token"]

def plantgraph_query(cypher, token=None):
    headers = {"Content-Type": "application/json"}
    if token:
        headers["Authorization"] = f"Bearer {token}"
    r = requests.post(f"{BASE}/api/v1/graph/query", json={"query": cypher}, headers=headers)
    return r.json()["results"]

def plantgraph_gene(gene_id, token):
    r = requests.get(f"{BASE}/api/graph/gene/{gene_id}",
        headers={"Authorization": f"Bearer {token}"})
    return r.json()

def plantgraph_enrich(gene_id, sources, token):
    r = requests.get(f"{BASE}/api/enrichment/gene/{gene_id}",
        params={"sources": ",".join(sources)},
        headers={"Authorization": f"Bearer {token}"})
    return r.json()

def plantgraph_go_enrichment(gene_ids, token, ontology="BP"):
    r = requests.post(f"{BASE}/api/enrichment/analysis/go",
        json={"genes": gene_ids, "background": "genome",
              "ontology": ontology, "correction": "fdr"},
        headers={"Authorization": f"Bearer {token}"})
    return r.json()

# Example
token = plantgraph_login("user", "pass")
plantgraph_query('MATCH (l:Locus {name: "AT2G40150"})-[:ANNOTATED_WITH]->(go:GO) RETURN go.id, go.name')
plantgraph_enrich("AT1G01010", ["uniprot", "string", "alphafold"], token)
```
```

- [ ] **Step 2: Commit**

```bash
git add SKILL.md
git commit -m "skill: expand R integration and add Python integration with auth"
```

---

### Task 11: Deslop Pass

**Files:**
- Modify: `SKILL.md`

- [ ] **Step 1: Run deslop-wiki (iteration 1)**

Invoke `/deslop-wiki` on SKILL.md. Focus on:
- Removing filler phrases ("comprehensive", "powerful", "seamlessly")
- Tightening descriptions to minimum viable words
- Ensuring tables are compact
- Removing redundant explanations

- [ ] **Step 2: Run caveman lite (iteration 1)**

Apply `/caveman lite` compression pass. Drop unnecessary articles, filler, hedging. Keep technical terms exact.

- [ ] **Step 3: Run deslop-wiki + caveman lite (iteration 2)**

Second pass — catch anything missed in first round.

- [ ] **Step 4: Run deslop-wiki + caveman lite (iteration 3)**

Final pass — should be minimal changes. If no changes needed, skip.

- [ ] **Step 5: Commit**

```bash
git add SKILL.md
git commit -m "skill: deslop and compress SKILL.md (3 iterations)"
```

---

### Task 12: Final Validation

- [ ] **Step 1: Verify SKILL.md line count**

```bash
wc -l SKILL.md
```

Target: 900-1000 lines. If significantly over, trim deep reference sections. If under, check nothing was missed from spec.

- [ ] **Step 2: Verify all curl examples work**

Test one endpoint from each API category against live API:

```bash
# Cypher (no auth)
curl -s -X POST https://plantgraph.se/api/v1/graph/query \
  -H "Content-Type: application/json" \
  -d '{"query":"MATCH (l:Locus {name: \"AT1G01010\"}) RETURN l.name LIMIT 1"}'

# Search (no auth)
curl -s "https://plantgraph.se/api/v1/search/genes?q=FLC&limit=3"

# Status
curl -s https://plantgraph.se/api/v1/status
```

- [ ] **Step 3: Verify frontmatter parses correctly**

Check YAML frontmatter has valid syntax — no unescaped special characters, proper `>-` block scalar.

- [ ] **Step 4: Final commit if any fixes needed**

```bash
git add SKILL.md
git commit -m "skill: final validation fixes"
```

- [ ] **Step 5: Update README.md**

Update README.md to reflect new capabilities (14.5M+ nodes, 42 enrichment sources, AI features, auth). Keep it concise.

```bash
git add README.md
git commit -m "docs: update README for expanded skill"
```

Plan complete and saved to `docs/superpowers/plans/2026-04-16-skill-expansion.md`. Two execution options:

**1. Subagent-Driven (recommended)** - I dispatch a fresh subagent per task, review between tasks, fast iteration

**2. Inline Execution** - Execute tasks in this session using executing-plans, batch execution with checkpoints

Which approach?
