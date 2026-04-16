---
name: plantgraph-database
description: >-
  Use when querying PlantGraph (plantgraph.se) — Neo4j graph + REST API for plant gene
  annotations, interactions, co-expression, orthologues, GO terms, pathways, protein
  structures, enrichment. Covers Arabidopsis, rice, Brachypodium, wheat, maize, poplar,
  120+ species (14.5M+ nodes, 47.9M+ rels). 42 enrichment sources (UniProt, STRING,
  AlphaFold, KEGG, InterPro, Expression Atlas), AI question/hypothesis generation,
  GNN predictions, paper evidence verification, chat, NL graph queries. Use when user
  mentions PlantGraph or needs cross-species gene annotation, protein interactions,
  pathway enrichment, or literature evidence via a plant biology knowledge graph.
---

# PlantGraph Knowledge Graph & API

Public API at `plantgraph.se` (UPSC). 14.5M+ nodes, 47.9M+ rels, 127 species, 132 node types, 149 rel types. 140+ data sources, 42 real-time enrichment services.

## Authentication

**Base URL:** `https://plantgraph.se`

`/api/` requires JWT. `/api/v1/` (Cypher, search, paper evidence) mostly no auth.

### JWT Login
```bash
curl -s -X POST https://plantgraph.se/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username": "YOUR_USER", "password": "YOUR_PASS"}'
# Returns: {"access_token": "eyJ...", "token_type": "bearer", "expires_in": 86400}
```

Use token: `-H "Authorization: Bearer YOUR_TOKEN"`

Refresh: `POST /api/auth/refresh` with bearer header.

### Rate Limits

| Tier | Limit | Burst |
|---|---|---|
| Guest | 60/hour | 10/minute |
| Researcher | 1000/hour | 60/minute |

Exceeding returns `429`.

---

## API Endpoints

### Graph

| Endpoint | Method | Auth | Purpose |
|---|---|---|---|
| `/api/graph/gene/{gene_id}` | GET | JWT | Gene details |
| `/api/graph/search?q=&limit=&offset=` | GET | JWT | Search genes (max 100) |
| `/api/graph/gene/{gene_id}/neighbors` | GET | JWT | Connected nodes (filter by rel type, direction) |
| `/api/graph/gene/{gene_id}/pathways` | GET | JWT | Pathway membership |
| `/api/graph/gene/{gene_id}/stats` | GET | JWT | Counts |
| `/api/graph/pathway/{pathway_id}` | GET | JWT | Pathway with genes, reactions, compounds |
| `/api/graph/pathways?source=&q=` | GET | JWT | List/search pathways |
| `/api/graph/network/interactions` | POST | JWT | Interaction network for gene list |
| `/api/graph/network/coexpression` | POST | JWT | Co-expression network |
| `/api/graph/network/regulatory` | POST | JWT | Regulatory network |
| `/api/graph/stats` | GET | JWT | Graph-wide counts |
| `/api/graph/query` | POST | JWT | Raw Cypher |
| `/api/graph/query/natural` | POST | JWT | NL -> Cypher -> results |
| `/api/v1/graph/query` | POST | No | Raw Cypher (no auth needed) |

### Search

| Endpoint | Method | Auth | Purpose |
|---|---|---|---|
| `/api/v1/search/terms?q=` | GET | No | Term resolution (Arabidopsis only) |
| `/api/v1/search/genes?q=&limit=&offset=` | GET | No | Multi-species gene search |

### Enrichment

| Endpoint | Method | Auth | Purpose |
|---|---|---|---|
| `/api/enrichment/gene/{gene_id}` | GET | JWT | All 42 sources (~30s) |
| `/api/enrichment/gene/{gene_id}?sources=a,b` | GET | JWT | Selective sources |
| `/api/enrichment/gene/{gene_id}/{source}` | GET | JWT | Single source |
| `/api/enrichment/batch` | POST | JWT | Multiple genes + sources |
| `/api/enrichment/analysis/go` | POST | JWT | GO enrichment analysis |
| `/api/enrichment/analysis/pathway` | POST | JWT | Pathway enrichment analysis |
| `/api/enrichment/cache/status/{gene_id}` | GET | JWT | Cache info |
| `/api/v1/gene-enrichment/analyze` | POST | No | Legacy GO (Ath only, min 2 genes) |

### AI

| Endpoint | Method | Auth | Purpose |
|---|---|---|---|
| `/api/questions/generate-from-abstract` | POST | JWT | Research questions from abstract |
| `/api/questions/generate-from-pmid` | POST | JWT | Research questions from PMID |
| `/api/questions/random-pmid` | GET | JWT | Random paper |
| `/api/chat/message` | POST | JWT | Chat with sources |
| `/api/chat/sessions` | POST/GET | JWT | Session management |
| `/api/chat/sessions/{id}/history` | GET | JWT | Conversation history |
| `/api/chat/sessions/{id}` | DELETE | JWT | Delete session |
| `/api/ai/query` | POST | JWT | NL question -> Cypher -> results + confidence |
| `/api/ai/hypothesis` | POST | JWT | Hypothesis generation |
| `/api/gnn/predict/{gene_id}` | GET | JWT | GNN function predictions |
| `/api/gnn/predict/links` | POST | JWT | Predict novel relationships |
| `/api/qa-discovery/start` | POST | JWT | Autonomous Q&A discovery |
| `/api/qa-discovery/{id}/status` | GET | JWT | Progress |
| `/api/qa-discovery/{id}/results` | GET | JWT | Results + synthesis |

### Paper Evidence

| Endpoint | Method | Auth | Purpose |
|---|---|---|---|
| `/api/v1/paper-evidence/check-abstract` | POST | No | Score abstract (0-100) |
| `/api/v1/paper-evidence/verify-claims` | POST | No | Verify claims (SSE stream) |
| `/api/v1/paper-evidence/check` | POST | No | PDF upload score |
| `/api/v1/paper-evidence/verify-claims-pdf` | POST | No | PDF verify (SSE stream) |

### Auth & Status

| Endpoint | Method | Auth | Purpose |
|---|---|---|---|
| `/api/auth/login` | POST | No | JWT login |
| `/api/auth/refresh` | POST | JWT | Token refresh |
| `/api/v1/status` | GET | No | Health check |

---

## ID Format by Species

**Critical:** Arabidopsis uses `l.name` for lookups. All other species use `l.id`.

| Species | Example ID | Lookup property |
|---|---|---|
| Arabidopsis thaliana | `AT2G40150` | `l.name` or `l.id` |
| Brachypodium distachyon | `BRADI_1g16610v3` | `l.id` |
| Oryza sativa subsp. japonica | `Os01g0111600` (RAP) | `l.id` |
| Triticum aestivum | `TraesCS1D02G002600` | `l.id` |
| Zea mays | `Zm00001eb078980` | `l.id` |
| Hordeum vulgare | **Not in graph** | Use rice/Brachypodium orthologues |

### ID Conversion from Phytozome to PlantGraph

| Phytozome ID | PlantGraph ID | Rule |
|---|---|---|
| `Bradi1g16610` | `BRADI_1g16610v3` | `Bradi` -> `BRADI_`, append `v3` |
| `LOC_Os05g03390` | Not used | Use RAP format `Os05g0124800` instead |
| `HORVU.MOREX.r3.*` | N/A | Barley not in graph; go via Arabidopsis orthologues |

### B. sylvaticum Strategy

Not in PlantGraph. Use orthologue chain:
1. Map Bsyl gene -> Bdis orthologue (Phytozome BioMart)
2. Convert Bdis ID to PlantGraph format (`BRADI_{chr}g{num}v3`)
3. Query PlantGraph for Bdis locus or follow `ORTHOLOGOUS_TO` to Arabidopsis

## Key Node Labels

**Gene-level:** `Locus`, `Transcript`, `Protein`, `Gene`
**Annotation:** `GO`, `PO`, `ProteinDomain`, `InterProDomain`, `MapManBin`, `Pathway`
**Taxonomy:** `Species`, `GeneFamily`, `PhytozomeGeneFamily`, `ProteinFamily`
**Regulation:** `TranscriptionFactorFamily`, `Promoter`, `BindingSite`, `BindingMotif`, `TFMotif`, `RegulatoryElement`
**Expression:** `Tissue`, `CellType`, `ExpressionProfile`, `DiurnalProfile`, `DevelopmentalStage`
**Literature:** `Publication`, `Author`, `Journal`, `MeSH`
**Variants:** `SNP`, `INDEL`, `GWASStudy`, `Trait`, `PlantTrait`
**Other:** `UniProt`, `Phenotype`, `Hormone`, `Pathogen`, `Complex`, `Metabolite`

## Key Relationship Types (149 total)

**Core biology:** `INTERACTS_WITH`, `COEXPRESSES_WITH`, `COFUNCTIONAL_WITH` (AraNet), `ORTHOLOGOUS_TO`, `PARALOGOUS_TO`
**Annotation:** `ANNOTATED_WITH` (GO), `ANNOTATED_WITH_PO`, `HAS_DOMAIN`, `HAS_INTERPRO_DOMAIN`, `HAS_MAPMAN_BIN`
**Regulation (outgoing):** `ACTIVATES`, `REPRESSES`, `REGULATES`, `BINDS_TO`, `BINDS_PROMOTER`, `TARGETS`
**Regulation (incoming):** `REGULATES`, `DAPSEQ_BINDS_PROMOTER` (gold-standard DAP-seq TF binding), `TARGETS`, `ACTIVATES`, `REPRESSES`
**Membership:** `MEMBER_OF`, `MEMBER_OF_TF_FAMILY`, `MEMBER_OF_GENE_FAMILY`, `MEMBER_OF_COMPLEX`, `PART_OF_COMPLEX`, `PARTICIPATES_IN`
**Expression:** `EXPRESSED_IN`, `EXPRESSED_IN_TISSUE` (target is DevelopmentalStage, not Tissue), `EXPRESSED_IN_CELL_TYPE`, `EXPRESSED_UNDER_STRESS`
**Responses:** `RESPONDS_TO_HORMONE`, `RESPONDS_TO_NUTRIENT` (N, P, K, Fe, Ca, S, Zn), `RESPONDS_TO_PATHOGEN`
**Life history:** `INVOLVED_IN_FLOWERING` (1,479 genes), `CITED_IN_FLOWERING`, `INVOLVED_IN_ADAPTATION` (vernalisation, photoperiod, thermosensing, drought, heat)
**GWAS/traits:** `ASSOCIATED_WITH_TRAIT` (flowering_time, biomass, root_length, seed_yield, stress_response)
**Epigenetics:** `HAS_HISTONE_MARK` (H3K27me3, H3K4me3, H4K16ac, etc.), `HAS_METHYLATION`
**PTMs:** `HAS_PHOSPHORYLATION_SITE`, `HAS_UBIQUITINATION_SITE`, `HAS_ACETYLATION_SITE`, `HAS_SUMOYLATION_SITE`, `HAS_SNITROSYLATION_SITE`, `HAS_SULFENYLATION_SITE`
**Literature:** `PUBLISHED_IN`, `MENTIONED_IN`, `CITED_IN_FLOWERING`
**Pathway/enzyme:** `BELONGS_TO_PATHWAY`, `CATALYZES`, `CATALYZES_REACTION`, `ENCODES_ENZYME`

## Lookup Methods

### 1. By ID (primary)
```cypher
-- Arabidopsis (has both l.name and l.id set)
MATCH (l:Locus {name: "AT2G40150"}) RETURN l.name, l.symbol, l.description

-- Non-Arabidopsis (l.name is null, use l.id)
MATCH (l:Locus {id: "BRADI_1g16610v3"}) RETURN l.id, l.species, properties(l)
```

### 2. By gene symbol
```cypher
MATCH (l:Locus {symbol: "TBL28"}) RETURN l.id, l.species
-- Pattern match:
MATCH (l:Locus) WHERE l.symbol CONTAINS "TBL" RETURN l.id, l.symbol LIMIT 20
```
Only Ath and some rice loci have symbols.

### 3. By fulltext search (cross-species)
```cypher
CALL db.index.fulltext.queryNodes("gene_fulltext", "trichome birefringence")
YIELD node, score
RETURN node.id, node.symbol, node.species, score LIMIT 10
```
Indexes: `gene_fulltext`, `go_fulltext`, `publication_fulltext`, `pathway_fulltext`, `compound_fulltext`, `enzyme_fulltext`, `reaction_fulltext`.

### 4. By UniProt accession (reverse lookup)
```cypher
MATCH (l:Locus)-[:MAPS_TO]->(u:UniProt {id: "Q94K00"}) RETURN l.id, l.name, l.species
```

### 5. Via /search/terms (Arabidopsis only)
```
GET /api/v1/search/terms?q=TBL28      -> {"term":"TBL28","id":"AT2G40150","type":"gene"}
GET /api/v1/search/terms?q=AT2G40150   -> same
```
Resolves Ath symbols and TAIR IDs only.

---

## Top 10 Common Query Patterns

### 1. Get GO annotations
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[:ANNOTATED_WITH]->(go:GO)
RETURN go.id, go.name
```

### 2. Get protein interactions
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[:INTERACTS_WITH]->(p:Locus)
RETURN p.name, p.symbol, p.description
```

### 3. Get co-expression partners
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[:COEXPRESSES_WITH]->(c:Locus)
RETURN c.name, c.symbol, c.description
```

### 4. Get regulatory network (incoming edges)
```cypher
MATCH (tf:Locus)-[r:REGULATES|ACTIVATES|REPRESSES]->(l:Locus {name: "AT2G40150"})
RETURN tf.name, tf.symbol, type(r) AS regulation_type
```

### 5. Get orthologues (specific species)

**Note:** `ORTHOLOGOUS_TO` direction is arbitrary. Always use undirected match for cross-species lookups.

```cypher
MATCH (a:Locus {name: "AT2G40150"})-[:ORTHOLOGOUS_TO]->(o:Locus)
WHERE o.species = "Brachypodium distachyon"
RETURN o.id, o.species
```

### 6. Get orthologues (all species with counts)
```cypher
MATCH (a:Locus {name: "AT2G40150"})-[:ORTHOLOGOUS_TO]->(o:Locus)
WHERE o.species IS NOT NULL
RETURN o.species, count(*) AS n ORDER BY n DESC
```

### 7. Find Ath orthologue from non-Ath gene

**Critical:** Use **undirected** match (no arrow):

```cypher
MATCH (b:Locus {id: "BRADI_1g16610v3"})-[:ORTHOLOGOUS_TO]-(a:Locus)
WHERE a.species = "Arabidopsis thaliana"
RETURN a.name, a.symbol, a.description
```

### 8. Multi-hop: Brachypodium -> Arabidopsis -> interactions
```cypher
MATCH (b:Locus {id: "BRADI_1g16610v3"})-[:ORTHOLOGOUS_TO]-(a:Locus)
WHERE a.species = "Arabidopsis thaliana"
WITH a
MATCH (a)-[:INTERACTS_WITH]->(p:Locus)
RETURN a.name AS ath_gene, p.name AS interactor, p.description
```

### 9. Bulk query: multiple genes
```cypher
MATCH (l:Locus)
WHERE l.name IN ["AT1G01430","AT2G40150","AT5G06700"]
OPTIONAL MATCH (l)-[:ANNOTATED_WITH]->(go:GO)
RETURN l.name, l.symbol, collect(DISTINCT go.name) AS go_terms
```

### 10. Schema introspection
```cypher
CALL db.labels() YIELD label RETURN label
CALL db.relationshipTypes() YIELD relationshipType RETURN relationshipType
MATCH (l:Locus) RETURN DISTINCT l.species, count(*) AS n ORDER BY n DESC
```

---

## Graph API (REST)

All `/api/graph/` require JWT (`-H "Authorization: Bearer $TOKEN"`).

### GET examples
```bash
curl -s https://plantgraph.se/api/graph/gene/AT2G40150 -H "Authorization: Bearer $TOKEN"
curl -s "https://plantgraph.se/api/graph/search?q=trichome&limit=10" -H "Authorization: Bearer $TOKEN"
curl -s "https://plantgraph.se/api/graph/gene/AT2G40150/neighbors?rel_type=INTERACTS_WITH&direction=both" -H "Authorization: Bearer $TOKEN"
curl -s https://plantgraph.se/api/graph/gene/AT2G40150/pathways -H "Authorization: Bearer $TOKEN"
curl -s https://plantgraph.se/api/graph/gene/AT2G40150/stats -H "Authorization: Bearer $TOKEN"
curl -s https://plantgraph.se/api/graph/pathway/PWY-5136 -H "Authorization: Bearer $TOKEN"
curl -s "https://plantgraph.se/api/graph/pathways?source=KEGG&q=photosynthesis" -H "Authorization: Bearer $TOKEN"
curl -s https://plantgraph.se/api/graph/stats -H "Authorization: Bearer $TOKEN"
```

### POST examples
```bash
# Interaction network
curl -s -X POST https://plantgraph.se/api/graph/network/interactions \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"gene_ids": ["AT2G40150", "AT1G01430", "AT5G06700"]}'

# Co-expression network
curl -s -X POST https://plantgraph.se/api/graph/network/coexpression \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"gene_ids": ["AT2G40150", "AT1G01430"], "min_score": 0.7}'

# Regulatory network (TF -> target)
curl -s -X POST https://plantgraph.se/api/graph/network/regulatory \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"gene_ids": ["AT2G40150"], "include_dapseq": true}'

# NL query -> Cypher + results + confidence
curl -s -X POST https://plantgraph.se/api/graph/query/natural \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"question": "What genes interact with FLC in Arabidopsis?"}'

# Raw Cypher (authenticated)
curl -s -X POST https://plantgraph.se/api/graph/query \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"query": "MATCH (l:Locus {name: \"AT2G40150\"}) RETURN l LIMIT 1"}'
```

### No-auth endpoints
```bash
# Raw Cypher
curl -s -X POST https://plantgraph.se/api/v1/graph/query \
  -H "Content-Type: application/json" \
  -d '{"query":"MATCH (l:Locus {name: \"AT2G40150\"}) RETURN l.name, l.description LIMIT 1"}'

# Multi-species gene search
curl -s "https://plantgraph.se/api/v1/search/genes?q=flowering&limit=20&offset=0"
```

---

## Enrichment API

`/api/enrichment/` requires JWT unless noted.

### Query patterns
```bash
# All 42 sources (~30s)
curl -s https://plantgraph.se/api/enrichment/gene/AT2G40150 -H "Authorization: Bearer $TOKEN"
# Selective sources
curl -s "https://plantgraph.se/api/enrichment/gene/AT2G40150?sources=uniprot,string,alphafold" -H "Authorization: Bearer $TOKEN"
# Single source
curl -s https://plantgraph.se/api/enrichment/gene/AT2G40150/uniprot -H "Authorization: Bearer $TOKEN"
```

### 42 Enrichment Sources

**Protein & Structure (7):** `uniprot` (function, PTMs, sequence), `alphafold` (3D structure, pLDDT), `interpro` (domains, sites), `panther` (families, orthologs), `pdbe` (experimental structures), `rcsb_pdb` (structures + quality), `ebi_proteins` (features, variants)

**Interactions (3):** `string` (PPI scores, `score_threshold` param, default 0.4), `intact` (experimental interactions), `biogrid` (physical + genetic)

**Pathways (3):** `kegg` (500+ Ath pathways), `reactome` (curated reactions), `wikipathways` (community, 200+ species)

**Expression (3):** `expression_atlas` (3000+ experiments, `experiment_type` param), `bar_efp` (tissue expression), `atted2` (co-expression scores)

**Ontology (3):** `quickgo` (GO + evidence codes), `chebi` (chemicals, SMILES/InChI), `ols` (200+ EBI ontologies)

**Literature (3):** `europepmc` (40M+ articles, `sort` param), `semantic_scholar` (200M+ papers), `crossref` (130M+ DOIs)

**Comparative (4):** `ensembl` (orthologs, gene trees, 7 plant spp), `oma` (2100+ genomes), `gramene` (66 plant spp), `plaza` (families, synteny, 50+ genomes)

**Plant-specific (6):** `planttfdb` (TF classification, 2000+ Ath TFs), `suba` (subcellular localization), `aranet` (co-functional, 22K+ genes), `arapheno` (QTL, 1135 accessions), `thalemine` (integrated info), `1001genomes` (SNPs/indels, 1135 accessions)

**Other (4):** `rnacentral` (ncRNA, 20M+ seq), `pubchem` (100M+ compounds), `wikidata` (linked data), `mygene` (annotation aggregation)

### Batch, GO, Pathway enrichment
```bash
# Batch (multiple genes + sources)
curl -s -X POST https://plantgraph.se/api/enrichment/batch \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"genes":["AT2G40150","AT1G01430","AT5G06700"],"sources":["uniprot","string"],"parallel":true}'

# GO enrichment (returns enriched terms with p-values, fold enrichment)
curl -s -X POST https://plantgraph.se/api/enrichment/analysis/go \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"genes":["AT2G40150","AT1G01430","AT5G06700","AT3G20810"],"background":"genome","ontology":"BP","correction":"fdr"}'

# Pathway enrichment
curl -s -X POST https://plantgraph.se/api/enrichment/analysis/pathway \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"genes":["AT2G40150","AT1G01430","AT5G06700"],"databases":["kegg","reactome"],"p_threshold":0.05}'

# Legacy (Ath only, no auth, min 2 genes, p_threshold max 0.5)
curl -s -X POST https://plantgraph.se/api/v1/gene-enrichment/analyze \
  -H "Content-Type: application/json" \
  -d '{"gene_ids":["AT1G01010","AT1G01020"],"categories":["GO"],"p_threshold":0.05}'

# Cache status
curl -s https://plantgraph.se/api/enrichment/cache/status/AT2G40150 -H "Authorization: Bearer $TOKEN"
```

---

## AI API

`/api/` AI endpoints require JWT.

### Questions & Chat
```bash
# Generate questions from abstract
curl -s -X POST https://plantgraph.se/api/questions/generate-from-abstract \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"abstract": "We investigated the role of FLC in vernalization-mediated flowering...", "num_questions": 5}'

# From PMID
curl -s -X POST https://plantgraph.se/api/questions/generate-from-pmid \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"pmid": "34567890"}'

# Random PMID from 290K publications
curl -s https://plantgraph.se/api/questions/random-pmid -H "Authorization: Bearer $TOKEN"

# Chat (with sources)
curl -s -X POST https://plantgraph.se/api/chat/message \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"session_id": "sess_abc123", "message": "What are the main regulators of flowering time in Arabidopsis?"}'

# Session: create / list / history
curl -s -X POST https://plantgraph.se/api/chat/sessions \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"name": "Flowering research"}'
curl -s https://plantgraph.se/api/chat/sessions -H "Authorization: Bearer $TOKEN"
curl -s https://plantgraph.se/api/chat/sessions/sess_abc123/history -H "Authorization: Bearer $TOKEN"
```

### AI Query, Hypothesis, GNN
```bash
# NL -> Cypher + results + confidence
curl -s -X POST https://plantgraph.se/api/ai/query \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"question": "Which transcription factors regulate FLC expression?"}'

# Hypothesis generation
curl -s -X POST https://plantgraph.se/api/ai/hypothesis \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"gene_ids": ["AT5G10140", "AT3G18990"], "context": "cold stress response"}'

# GNN function predictions
curl -s https://plantgraph.se/api/gnn/predict/AT2G40150 -H "Authorization: Bearer $TOKEN"

# GNN link prediction (novel relationships with probability)
curl -s -X POST https://plantgraph.se/api/gnn/predict/links \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"gene_id": "AT2G40150", "rel_types": ["INTERACTS_WITH", "COEXPRESSES_WITH"], "top_k": 20}'
```

### Q&A Discovery (autonomous loop)
```bash
# Start
curl -s -X POST https://plantgraph.se/api/qa-discovery/start \
  -H "Authorization: Bearer $TOKEN" -H "Content-Type: application/json" \
  -d '{"topic": "cell wall biosynthesis in grasses", "max_iterations": 10, "depth": "deep"}'

# Check progress / Get results
curl -s https://plantgraph.se/api/qa-discovery/disc_xyz789/status -H "Authorization: Bearer $TOKEN"
curl -s https://plantgraph.se/api/qa-discovery/disc_xyz789/results -H "Authorization: Bearer $TOKEN"
```

---

## Paper Evidence API

`/api/v1/` prefix, no auth. Claim verification takes 30-70s.

### Check abstract (score 0-100)
```bash
curl -s -X POST https://plantgraph.se/api/v1/paper-evidence/check-abstract \
  -H "Content-Type: application/json" \
  -d '{"abstract_text": "FLC is a key repressor of flowering in Arabidopsis, regulated by vernalization through epigenetic silencing via Polycomb..."}'
# Returns: {overall_score, score_tier, entities[], evidence{graph_support, literature_support, interaction_support}}
```

### Verify claims (SSE stream)
```bash
curl -s -X POST https://plantgraph.se/api/v1/paper-evidence/verify-claims \
  -H "Content-Type: application/json" \
  -d '{"abstract_text": "FLC is repressed by VIN3 during vernalization...", "claims": ["FLC interacts with SVP to repress flowering", "VIN3 is induced by cold exposure"]}'
# SSE events: claim_extracted -> entity_report -> claim_verified (verdict + confidence) -> complete
```

If `claims` omitted, auto-extracted from abstract. Verdicts: `supported`, `partially_supported`, `not_found`.

### PDF endpoints
```bash
curl -s -X POST https://plantgraph.se/api/v1/paper-evidence/check -F "file=@paper.pdf"           # Score
curl -s -X POST https://plantgraph.se/api/v1/paper-evidence/verify-claims-pdf -F "file=@paper.pdf" # SSE stream
```

---

## Cypher Deep Reference

### Node Labels (132)

**Gene-level:**
`Locus`, `Transcript`, `Protein`, `Gene`, `SmallORF`/`sORF`, `CircRNA`/`CircularRNA`, `LncRNA`/`lncRNA`, `SmallRNA`

**Annotation:**
`GO`, `PO`, `TO`, `BTO`, `PECO`, `ENVO`, `ProteinDomain`, `InterProDomain`, `MapManBin`, `Pathway`, `Enzyme`, `Reaction`, `MetabolicCompound`

**Taxonomy:**
`Species`, `NCBITaxon`, `GeneFamily`, `PhytozomeGeneFamily`, `ProteinFamily`

**Regulation:**
`TranscriptionFactorFamily`, `Promoter`, `BindingSite`, `BindingMotif`, `TFMotif`, `Motif`, `RegulatoryElement`, `ReMapPeak`, `PeakRegion`, `DHSRegion`, `ChromatinAccessibilityPeak`, `SignalTrack`

**Expression:**
`Tissue`, `CellType`, `ExpressionProfile` (7.3M), `DiurnalProfile`, `DevelopmentalStage`, `SpatialRegion`, `SpatialZone`, `DiurnalCondition`

**Experiment:**
`Experiment`, `Study`, `Sample`, `GrowthCondition`, `Condition`, `StressCondition`, `StressExperiment`, `StressTimePoint`, `TimePoint`, `RnaSeqLibrary`, `MicroarrayPlatform`, `EBIAtlasExperiment`, `BARExperiment`, `GSE`, `GSM`, `MetaboLightsStudy`

**Literature:**
`Publication` (290K), `Preprint`, `Author` (105K), `Journal`, `MeSH`

**Variants/GWAS:**
`SNP` (1.3M), `INDEL`, `GWASStudy`, `Trait`, `PlantTrait`, `AraphenoStudy`, `Germplasm`

**Protein features:**
`UniProt`, `ProteinComplex`, `Complex`, `ProteinStructure`, `ProteinFeature`, `ProteinLigand`, `Ligand`, `LigandBinding`, `LigandBindingSite`, `ProteomicsEvidence`

**PTM sites:**
`PhosphorylationSite` (190K), `UbiquitinationSite`, `AcetylationSite`, `SUMOylationSite`, `SnitrosylationSite`, `SulfenylationSite`, `GlcNAcylationSite`, `MethylationSite` (2.9M), `PTMSite`

**Chemistry:**
`ChEBICompound`, `KnapsackCompound`, `Metabolite`, `NaturalProduct`, `Phytochemical`, `PhytochemicalCompound`, `PhytohubCompound`

**Epigenetics:**
`MethylationContext`, `HistoneModification` (154K)

**Pathogen:**
`Pathogen`, `PathogenEffector`, `PathogenGene`, `PHIGene`, `PHIInteraction`, `PHIOrganism`, `PHIPOTerm`, `MicrobeInteraction`

**Biosynthesis:**
`BGC_Cluster`, `BGCluster`, `BiosynthesisCluster`, `BiosyntheticCluster` (legacy duplicates)

**Other:**
`SubcellularCompartment`, `CellularCompartment`, `GeneXref`, `OntologyAnchor`, `Hormone`, `Nutrient`, `Phenotype`, `ClimateAdaptation`, `Institution`, `PMNDatabase`, `TAIRLocus`, `TAIRObject`

### Relationship Types (149)

**Top 20 by count:**

| Relationship | Count | Notes |
|---|---|---|
| `EXPRESSED_IN_CELL_TYPE` | 22.1M | Single-cell expression |
| `ORTHOLOGOUS_TO` | 18.0M | Cross-species, use undirected match |
| `ANNOTATED_WITH` | 14.2M | GO terms, filter by evidence code |
| `INTERACTS_WITH` | 8.2M | PPI |
| `HAS_PROFILE` | 7.7M | Expression profiles |
| `EXPRESSED_IN` | 7.4M | Tissue/organ expression |
| `BINDS_TO` | 5.4M | TF binding |
| `REGULATES` | 4.4M | Regulatory edges |
| `COEXPRESSES_WITH` | 3.8M | Filter by correlation_score |
| `DAPSEQ_BINDS_PROMOTER` | 3.4M | Gold-standard TF binding |
| `HAS_METHYLATION` | 2.9M | DNA methylation |
| `PARTICIPATES_IN` | 2.7M | Pathway membership |
| `HAS_MESH_TERM` | 2.6M | Publication MeSH |
| `COFUNCTIONAL_WITH` | 2.6M | AraNet |
| `BINDS_PROMOTER` | 2.4M | Promoter binding |
| `METHYLATED_IN_CONTEXT` | 1.6M | CpG/CHG/CHH context |
| `ANNOTATED_WITH_PO` | 1.6M | Plant Ontology |
| `HAS_MAPMAN_BIN` | 1.6M | MapMan functional bins |
| `HAS_DOMAIN` | 1.5M | Protein domains |
| `IN_TAXON` | 1.5M | Taxonomic assignment |

**LIMIT aggressively** on high-count rels. `INTERACTS_WITH`/`COEXPRESSES_WITH` can return 100K+ rows.

**Full list by category:**

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

### Relationship Properties

| Relationship | Properties | Notes |
|---|---|---|
| `INTERACTS_WITH` | `pubmed_ids`, `methods`, `source`, `evidence_codes` | Filter by source or evidence |
| `COEXPRESSES_WITH` | `correlation_score`, `rank`, `dataset_source`, `source_entrez_id`, `target_entrez_id` | Always has score/rank. Optional: `atted2_v13_lls`/`rank` |
| `ANNOTATED_WITH` | `source`, `evidence` | Evidence: IDA, IMP, IGI, IPI, IEP (experimental), IEA (computational) |
| `ORTHOLOGOUS_TO` | `source` | Source DB only, no score |
| `EXPRESSED_UNDER_STRESS` | `log2fc` | Indexed — fast range queries |
| `DAPSEQ_BINDS_PROMOTER` | `source` | Indexed |
| `NCBI_ANNOTATED_WITH` | `evidence_code` | Indexed |

**Filter GO to experimental evidence:**
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[r:ANNOTATED_WITH]->(go:GO)
WHERE r.evidence IN ["IDA", "IMP", "IGI", "IPI", "IEP"]
RETURN go.id, go.name, r.evidence
```

**Filter co-expression by score:**
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[r:COEXPRESSES_WITH]->(c:Locus)
WHERE r.correlation_score > 0.8
RETURN c.name, c.symbol, r.correlation_score
ORDER BY r.correlation_score DESC LIMIT 20
```

**Filter stress expression by log2fc:**
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[r:EXPRESSED_UNDER_STRESS]->(s)
WHERE r.log2fc > 1
RETURN s.name AS stress, r.log2fc
ORDER BY r.log2fc DESC
```

### Node Properties

**Locus (Arabidopsis):**
`id`, `name` (AT ID), `symbol`, `species`, `taxon_id`, `chromosome`, `start`, `end`, `strand`, `description`, `gene_type`, `is_tf`, `tf_family`, `is_arabidopsis` (true), `ncbi_gene_id`, `entrez_id`, `uniprot_id`, `pfam_ids` (array), `subcellular_location`, `peptideatlas_detected`, `ensembl_homolog_count`, `aranet_enriched`, `remap_peak_count`, `remap_tf_count`. Embedded expression: `atgen_drought_*`/`atgen_salt_*`/`atgen_cold_*` (ATGenExpress TPMs), `circ_emexp1304_*` (circadian), `geod*_tpm`/`geod*_lfc`, `mtab*_tpm`.

**Locus (non-Arabidopsis):**
`id`, `species`, `taxon_id`, `description`, `description_source`, `source`, `is_arabidopsis` (false), `ortholog_ref_id`. Expression species-prefixed: `bradi_mtab4401_*_tpm`, `bradi_mean_tpm`, `bradi_max_tpm`. `name` is null -- use `id`.

**GO:** `id`, `name`, `name_obo`, `namespace` (molecular_function/biological_process/cellular_component), `definition`, `synonyms` (array), `is_obsolete`, `enriched`

**Publication:** `pubmed_id`, `title`, `abstract`, `authors`, `year`, `journal`, `doi`, `pmc_id`, `citation_count`, `cited_by_count`, `is_open_access`, `full_text_mined`, `gene_count`. **Warning:** `research_concepts` type inconsistent -- sometimes `List[str]`, sometimes semicolon-delimited string.

**Species:** `id` (snake_case), `name`, `common_name`, `taxon_id`, `source`

**Pathway:** `id`, `name`, `kegg_name`, `kegg_category`, `kegg_organism`, `kegg_gene_count`, `kegg_compound_count`, `url`

### Fulltext Indexes (7)

| Index Name | Label | Properties |
|---|---|---|
| `gene_fulltext` | Locus | symbol, name, description |
| `go_fulltext` | GO | name, definition |
| `publication_fulltext` | Publication | title, abstract |
| `pathway_fulltext` | Pathway | name, common_name |
| `compound_fulltext` | MetabolicCompound | name, common_name |
| `enzyme_fulltext` | Enzyme | common_name, id |
| `reaction_fulltext` | Reaction | common_name, id |

```cypher
CALL db.index.fulltext.queryNodes("gene_fulltext", "cellulose synthase")
YIELD node, score
RETURN node.id, node.symbol, node.species, node.description, score
ORDER BY score DESC LIMIT 15
```

### Key Indexes

**Node:** `Locus.id` (unique), `Locus.name`, `Locus.species`, `Locus.symbol`, `Locus.is_tf`, `Locus.tf_family`, `Locus.taxon_id`, `Locus.entrez_id`, `Locus.ncbi_gene_id`, `Locus.chromosome+start+end` (composite).
**Relationship:** `COEXPRESSES_WITH.correlation_score`, `EXPRESSED_UNDER_STRESS.log2fc`, `INTERACTS_WITH.source`, `DAPSEQ_BINDS_PROMOTER.source`.

### Additional Query Patterns
```cypher
-- DAP-seq TF binding (gold standard)
MATCH (tf:Locus)-[:DAPSEQ_BINDS_PROMOTER]->(l:Locus {name: "AT2G40150"})
RETURN tf.name, tf.symbol

-- GWAS trait associations
MATCH (l:Locus {name: "AT2G40150"})-[:ASSOCIATED_WITH_TRAIT]->(t)
RETURN t.name AS trait

-- Protein complex membership
MATCH (l:Locus {name: "AT2G40150"})-[:MEMBER_OF_COMPLEX|PART_OF_COMPLEX]->(c)
RETURN c.name AS complex_name

-- Nutrient responses
MATCH (l:Locus {name: "AT1G02860"})-[:RESPONDS_TO_NUTRIENT]->(n)
RETURN n.name AS nutrient

-- Pathogen responses
MATCH (l:Locus {name: "AT1G01190"})-[:RESPONDS_TO_PATHOGEN]->(p)
RETURN p.name AS pathogen

-- Publications mentioning gene
MATCH (l:Locus {name: "AT2G40150"})-[:PUBLISHED_IN|MENTIONED_IN]->(pub:Publication)
RETURN pub.title, pub.pmid LIMIT 10

-- Histone marks (h.id contains mark type, e.g. "AraENC_H3K27me3_Chr3_...")
MATCH (l:Locus {name: "AT3G20810"})-[:HAS_HISTONE_MARK]->(h)
RETURN h.id LIMIT 10

-- Cell-type expression (single-cell)
MATCH (l:Locus {name: "AT2G40150"})-[:EXPRESSED_IN_CELL_TYPE]->(ct:CellType)
RETURN ct.name LIMIT 10

-- Spatial expression
MATCH (l:Locus {name: "AT2G40150"})-[:EXPRESSED_IN_REGION]->(sr:SpatialRegion)
RETURN sr.name LIMIT 10

-- Methylation + context
MATCH (l:Locus {name: "AT3G20810"})-[:HAS_METHYLATION]->(m)
RETURN m.id LIMIT 10
MATCH (l:Locus {name: "AT3G20810"})-[:METHYLATED_IN_CONTEXT]->(mc:MethylationContext)
RETURN mc.id LIMIT 10

-- All-annotations dump
MATCH (l:Locus {name: "AT2G40150"})-[r]->(t)
RETURN type(r) AS rel, labels(t)[0] AS target_type,
       coalesce(t.name, t.id) AS target, t.description
LIMIT 50
```

---

## Species in Graph

127 species, 1.62M loci. Top 20 by locus count:

| Species | Loci | Species | Loci |
|---|---|---|---|
| Brassica napus | 149K | Salix purpurea | 35K |
| Triticum aestivum | 108K | Populus trichocarpa | 35K |
| Brassica oleracea | 80K | Solanum pennellii | 34K |
| Glycine max | 61K | Medicago truncatula | 33K |
| Pinus taeda | 52K | Quercus lobata | 33K |
| Arabidopsis lyrata | 52K | Solanum lycopersicum | 32K |
| Arabidopsis thaliana | 44K | Zea mays | 30K |
| Phaseolus vulgaris | 42K | Oryza sativa subsp. japonica | 22K |
| Juglans regia | 40K | Sorghum bicolor | 16K |
| Brassica rapa | 39K | Brachypodium distachyon | 15K |

Also: 9 Oryza spp, conifers (Picea abies), mosses (Physcomitrium patens), ferns (Selaginella moellendorffii), algae, liverworts, hornworts.

**Not in graph:** Hordeum vulgare, B. sylvaticum. Use orthologue chains via Bdis or Ath.

**Rice note:** Appears as both `"Oryza sativa"` (13K) and `"Oryza sativa subsp. japonica"` (22K). Check both.

---

## Cypher Best Practices

- **NULL on sorts**: `WHERE prop IS NOT NULL` or `ORDER BY prop NULLS LAST`
- **`count{}` not `size()`**: `WHERE count{(l)-[:ANNOTATED_WITH]->()} > 3`
- **`EXISTS{}`**: `WHERE EXISTS {MATCH (l)-[:INTERACTS_WITH]->()}` (not deprecated `exists()`)
- **Explicit grouping**: `WITH` clauses before aggregations
- **`elementId()` not `id()`**: `id()` is deprecated
- **`coalesce(l.name, l.id)`**: handles species where `l.name` is null
- **Always LIMIT**: 14.5M+ nodes; unbounded queries are slow
- **Filter ANNOTATED_WITH**: `WHERE r.evidence IN ["IDA","IPI","IMP","IGI","IEP"]` (exclude IEA floods)
- **Filter COEXPRESSES_WITH**: `WHERE r.correlation_score > 0.7` (3.8M edges)

### Query checklist
1. No deprecated functions (`id()`, `size()`, `exists()`)
2. NULL filters on sorted properties
3. LIMIT on exploratory queries
4. Undirected match for reverse orthologue lookups
5. `coalesce(t.name, t.id)` for cross-species results

---

## Common Mistakes

- `l.name` for non-Ath species -- use `l.id`
- Phytozome `Bradi1g*` format -- convert to `BRADI_1g*v3`
- MSU rice IDs `LOC_Os*` -- use RAP format `Os*`
- Expecting barley `HORVU.*` -- not in graph; use orthologue hop
- Directed `->` for reverse orthologue lookups -- use undirected `-[:ORTHOLOGOUS_TO]-`
- Wrong regulatory direction -- edges point FROM regulator TO target: `(tf)-[:REGULATES]->(l {name: "X"})`
- `EXPRESSED_IN_TISSUE` with `(t:Tissue)` -- target is `DevelopmentalStage`, use `(t)` unlabeled
- Missing `LIMIT` -- `INTERACTS_WITH`/`COEXPRESSES_WITH` can return 100K+ rows
- Rice species string -- check both `"Oryza sativa"` (13K) and `"Oryza sativa subsp. japonica"` (22K)
- `/api/` vs `/api/v1/` confusion -- `/api/` needs JWT; `/api/v1/` mostly no auth
- Enrichment for non-Ath genes -- support varies by source
- Unfiltered `ANNOTATED_WITH` -- IEA floods results; filter by evidence code
- Unfiltered `COEXPRESSES_WITH` -- 3.8M edges; add `correlation_score > 0.7`
- `Publication.research_concepts` type varies -- sometimes `List[str]`, sometimes semicolon-delimited string; handle both

---

## R Integration

```r
library(httr)
library(jsonlite)

plantgraph_login <- function(username, password) {
  resp <- POST(
    "https://plantgraph.se/api/auth/login",
    body = list(username = username, password = password),
    encode = "json",
    content_type_json()
  )
  content(resp, as = "parsed")$access_token
}

plantgraph_query <- function(cypher, token = NULL) {
  if (is.null(token)) {
    url <- "https://plantgraph.se/api/v1/graph/query"
    resp <- POST(url, body = list(query = cypher), encode = "json", content_type_json())
  } else {
    url <- "https://plantgraph.se/api/graph/query"
    resp <- POST(url, body = list(query = cypher), encode = "json",
                 content_type_json(), add_headers(Authorization = paste("Bearer", token)))
  }
  content(resp, as = "parsed", simplifyVector = TRUE)$results
}

plantgraph_gene <- function(gene_id, token) {
  resp <- GET(
    paste0("https://plantgraph.se/api/graph/gene/", gene_id),
    add_headers(Authorization = paste("Bearer", token))
  )
  content(resp, as = "parsed", simplifyVector = TRUE)
}

plantgraph_enrich <- function(gene_id, sources = NULL, token) {
  url <- paste0("https://plantgraph.se/api/enrichment/gene/", gene_id)
  if (!is.null(sources)) url <- paste0(url, "?sources=", paste(sources, collapse = ","))
  resp <- GET(url, add_headers(Authorization = paste("Bearer", token)))
  content(resp, as = "parsed", simplifyVector = TRUE)
}

plantgraph_go_enrichment <- function(gene_ids, p_threshold = 0.05, token) {
  resp <- POST(
    "https://plantgraph.se/api/enrichment/analysis/go",
    body = list(gene_ids = gene_ids, categories = list("biological_process"),
                p_threshold = p_threshold, correction = "benjamini_hochberg"),
    encode = "json",
    content_type_json(),
    add_headers(Authorization = paste("Bearer", token))
  )
  content(resp, as = "parsed", simplifyVector = TRUE)
}

# Example: Brachypodium GO terms (no auth)
plantgraph_query('
  MATCH (l:Locus {id: "BRADI_1g16610v3"})-[:ANNOTATED_WITH]->(go:GO)
  RETURN go.id, go.name
')

# token <- plantgraph_login("user", "pass")
# plantgraph_enrich("AT2G40150", sources = c("uniprot", "string"), token = token)
```

---

## Python Integration

```python
import requests

BASE_URL = "https://plantgraph.se"

def plantgraph_login(username: str, password: str) -> str:
    """Return JWT token."""
    resp = requests.post(
        f"{BASE_URL}/api/auth/login",
        json={"username": username, "password": password}
    )
    resp.raise_for_status()
    return resp.json()["access_token"]


def plantgraph_query(cypher: str, token: str | None = None) -> list:
    """Run Cypher. Uses /api/v1/ (no auth) if token is None."""
    if token is None:
        url = f"{BASE_URL}/api/v1/graph/query"
        headers = {"Content-Type": "application/json"}
    else:
        url = f"{BASE_URL}/api/graph/query"
        headers = {
            "Content-Type": "application/json",
            "Authorization": f"Bearer {token}",
        }
    resp = requests.post(url, json={"query": cypher}, headers=headers)
    resp.raise_for_status()
    return resp.json().get("results", [])


def plantgraph_gene(gene_id: str, token: str) -> dict:
    """Get gene details."""
    resp = requests.get(
        f"{BASE_URL}/api/graph/gene/{gene_id}",
        headers={"Authorization": f"Bearer {token}"}
    )
    resp.raise_for_status()
    return resp.json()


def plantgraph_enrich(gene_id: str, token: str, sources: list[str] | None = None) -> dict:
    """Enrich gene from external sources."""
    url = f"{BASE_URL}/api/enrichment/gene/{gene_id}"
    if sources:
        url += f"?sources={','.join(sources)}"
    resp = requests.get(url, headers={"Authorization": f"Bearer {token}"})
    resp.raise_for_status()
    return resp.json()


def plantgraph_go_enrichment(
    gene_ids: list[str], token: str, p_threshold: float = 0.05
) -> dict:
    """GO enrichment on gene list."""
    resp = requests.post(
        f"{BASE_URL}/api/enrichment/analysis/go",
        json={
            "gene_ids": gene_ids,
            "categories": ["biological_process"],
            "p_threshold": p_threshold,
            "correction": "benjamini_hochberg",
        },
        headers={
            "Content-Type": "application/json",
            "Authorization": f"Bearer {token}",
        },
    )
    resp.raise_for_status()
    return resp.json()


# No-auth example
results = plantgraph_query('''
    MATCH (l:Locus {name: "AT2G40150"})-[:ANNOTATED_WITH]->(go:GO)
    RETURN go.id, go.name
''')

# token = plantgraph_login("user", "pass")
# gene = plantgraph_gene("AT2G40150", token)
# enrichment = plantgraph_enrich("AT2G40150", token, sources=["uniprot", "string"])
# go_results = plantgraph_go_enrichment(["AT2G40150", "AT1G01430", "AT5G06700"], token)
```
