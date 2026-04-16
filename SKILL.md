---
name: plantgraph-database
description: Use when querying PlantGraph (plantgraph.se) for plant gene annotations, interactions, co-expression, orthologues, GO terms, pathways, or knowledge graph data. Covers Arabidopsis, rice, Brachypodium, wheat, maize, poplar, and 60+ other plant species. Use when user mentions PlantGraph, or needs cross-species gene annotation via a Neo4j knowledge graph.
---

# PlantGraph Neo4j Knowledge Graph

Public API at `plantgraph.se` (UPSC, Umeå Plant Science Centre). 1.6M loci across 60+ plant species, 132 node types, 149 relationship types. No authentication required for graph queries.

## API Endpoints

| Endpoint | Method | Auth | Purpose |
|---|---|---|---|
| `/api/v1/graph/query` | POST | No | **Raw Cypher against Neo4j** |
| `/api/v1/gene-enrichment/analyze` | POST | No | GO enrichment (Arabidopsis IDs only, min 2) |
| `/api/v1/status` | GET | No | Health check |
| `/api/v1/search/terms` | GET | No | Term search |

### Query Format

```bash
curl -s -X POST https://plantgraph.se/api/v1/graph/query \
  -H "Content-Type: application/json" \
  -d '{"query":"MATCH (l:Locus {name: \"AT2G40150\"}) RETURN l.name, l.description LIMIT 1"}'
```

### Enrichment Format

```bash
curl -s -X POST https://plantgraph.se/api/v1/gene-enrichment/analyze \
  -H "Content-Type: application/json" \
  -d '{"gene_ids":["AT1G01010","AT1G01020"],"categories":["GO"],"p_threshold":0.05}'
```

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

B. sylvaticum is not in PlantGraph. Use the orthologue chain:
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
**Regulation (incoming — who regulates a gene):** `REGULATES`, `DAPSEQ_BINDS_PROMOTER` (gold-standard DAP-seq validated TF binding), `TARGETS`, `ACTIVATES`, `REPRESSES`
**Membership:** `MEMBER_OF`, `MEMBER_OF_TF_FAMILY`, `MEMBER_OF_GENE_FAMILY`, `MEMBER_OF_COMPLEX`, `PART_OF_COMPLEX`, `PARTICIPATES_IN`
**Expression:** `EXPRESSED_IN`, `EXPRESSED_IN_TISSUE` (target is DevelopmentalStage, not Tissue), `EXPRESSED_IN_CELL_TYPE`, `EXPRESSED_UNDER_STRESS`
**Responses:** `RESPONDS_TO_HORMONE`, `RESPONDS_TO_NUTRIENT` (N, P, K, Fe, Ca, S, Zn), `RESPONDS_TO_PATHOGEN`
**Life history:** `INVOLVED_IN_FLOWERING` (1,479 genes, pathway-categorised), `CITED_IN_FLOWERING`, `INVOLVED_IN_ADAPTATION` (vernalisation, photoperiod, thermosensing, drought, heat)
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
Only Arabidopsis and some rice loci have symbols populated.

### 3. By fulltext search (cross-species, searches description)
```cypher
CALL db.index.fulltext.queryNodes("gene_fulltext", "trichome birefringence")
YIELD node, score
RETURN node.id, node.symbol, node.species, score LIMIT 10
```
Indexes: `gene_fulltext` (Locus: symbol, name, description), `go_fulltext` (GO: name, definition), `publication_fulltext` (Publication: title, abstract), `pathway_fulltext`, `compound_fulltext`, `enzyme_fulltext`, `reaction_fulltext`.

### 4. By UniProt accession (reverse lookup)
```cypher
MATCH (l:Locus)-[:MAPS_TO]->(u:UniProt {id: "Q94K00"}) RETURN l.id, l.name, l.species
```

### 5. Via /search/terms endpoint (Arabidopsis only)
```
GET /api/v1/search/terms?q=TBL28      -> {"term":"TBL28","id":"AT2G40150","type":"gene"}
GET /api/v1/search/terms?q=AT2G40150   -> same
```
Only resolves Arabidopsis symbols and TAIR IDs. Does not find Brachypodium or other species.

## Common Query Patterns

### Get GO annotations
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[:ANNOTATED_WITH]->(go:GO)
RETURN go.id, go.name
```

### Get protein interactions
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[:INTERACTS_WITH]->(p:Locus)
RETURN p.name, p.symbol, p.description
```

### Get co-expression partners
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[:COEXPRESSES_WITH]->(c:Locus)
RETURN c.name, c.symbol, c.description
```

### Get regulatory network (who regulates a gene — incoming edges)
```cypher
MATCH (tf:Locus)-[r:REGULATES|ACTIVATES|REPRESSES]->(l:Locus {name: "AT2G40150"})
RETURN tf.name, tf.symbol, type(r) AS regulation_type
```

### Get DAP-seq validated TF binding (gold standard)
```cypher
MATCH (tf:Locus)-[:DAPSEQ_BINDS_PROMOTER]->(l:Locus {name: "AT2G40150"})
RETURN tf.name, tf.symbol
```

### Get GWAS trait associations
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[:ASSOCIATED_WITH_TRAIT]->(t)
RETURN t.name AS trait
```

### Get protein complex membership
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[:MEMBER_OF_COMPLEX|PART_OF_COMPLEX]->(c)
RETURN c.name AS complex_name
```

### Get nutrient/pathogen responses
```cypher
MATCH (l:Locus {name: "AT1G02860"})-[:RESPONDS_TO_NUTRIENT]->(n)
RETURN n.name AS nutrient
-- Pathogen:
MATCH (l:Locus {name: "AT1G01190"})-[:RESPONDS_TO_PATHOGEN]->(p)
RETURN p.name AS pathogen
```

### Get publications mentioning a gene
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[:PUBLISHED_IN|MENTIONED_IN]->(pub:Publication)
RETURN pub.title, pub.pmid LIMIT 10
```

### Get histone marks
```cypher
MATCH (l:Locus {name: "AT3G20810"})-[:HAS_HISTONE_MARK]->(h)
RETURN h.id LIMIT 10
-- Note: h.id contains mark type e.g. "AraENC_H3K27me3_Chr3_..."
```

### Get cell-type expression (single-cell resolution)
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[:EXPRESSED_IN_CELL_TYPE]->(ct:CellType)
RETURN ct.name LIMIT 10
```

### Get orthologues (specific species)
```cypher
MATCH (a:Locus {name: "AT2G40150"})-[:ORTHOLOGOUS_TO]->(o:Locus)
WHERE o.species = "Brachypodium distachyon"
RETURN o.id, o.species
```

### Get orthologues (all species with counts)
```cypher
MATCH (a:Locus {name: "AT2G40150"})-[:ORTHOLOGOUS_TO]->(o:Locus)
WHERE o.species IS NOT NULL
RETURN o.species, count(*) AS n ORDER BY n DESC
```

### Find Arabidopsis orthologue FROM a non-Arabidopsis gene

**Critical:** `ORTHOLOGOUS_TO` edges point FROM Arabidopsis outward. To find the Arabidopsis orthologue of a non-Ath gene, use **undirected** match (no arrow):

```cypher
MATCH (b:Locus {id: "BRADI_1g16610v3"})-[:ORTHOLOGOUS_TO]-(a:Locus)
WHERE a.species = "Arabidopsis thaliana"
RETURN a.name, a.symbol, a.description
```

### Multi-hop: Brachypodium -> Arabidopsis -> interactions
```cypher
MATCH (b:Locus {id: "BRADI_1g16610v3"})-[:ORTHOLOGOUS_TO]-(a:Locus)
WHERE a.species = "Arabidopsis thaliana"
WITH a
MATCH (a)-[:INTERACTS_WITH]->(p:Locus)
RETURN a.name AS ath_gene, p.name AS interactor, p.description
```

### Get all annotations for a gene (relationships + targets)
```cypher
MATCH (l:Locus {name: "AT2G40150"})-[r]->(t)
RETURN type(r) AS rel, labels(t)[0] AS target_type,
       coalesce(t.name, t.id) AS target, t.description
LIMIT 50
```

### Bulk query: multiple genes
```cypher
MATCH (l:Locus)
WHERE l.name IN ["AT1G01430","AT2G40150","AT5G06700"]
OPTIONAL MATCH (l)-[:ANNOTATED_WITH]->(go:GO)
RETURN l.name, l.symbol, collect(DISTINCT go.name) AS go_terms
```

### Schema introspection
```cypher
CALL db.labels() YIELD label RETURN label
CALL db.relationshipTypes() YIELD relationshipType RETURN relationshipType
MATCH (l:Locus) RETURN DISTINCT l.species, count(*) AS n ORDER BY n DESC
```

## Locus Properties (Arabidopsis)

Arabidopsis `Locus` nodes carry embedded expression data as properties:
- `atgen_drought_*`, `atgen_salt_*`, `atgen_cold_*` — ATGenExpress stress TPMs
- `circ_emexp1304_*` — circadian expression
- `geod*_lfc` — log fold-changes from GEO studies
- `aranet_enriched` — boolean, co-functional network membership
- `ensembl_homolog_count` — cross-species homologue count

Non-Arabidopsis loci also carry expression properties (e.g., `bradi_mtab4401_leaf_tpm`).

## Species in Graph (top by locus count)

Brassica napus (149K), Triticum aestivum (108K), Brassica oleracea (80K), Glycine max (61K), Arabidopsis thaliana (44K), Oryza sativa japonica (22K), Brachypodium distachyon (15K), Sorghum bicolor (16K), Setaria italica (16K), Zea mays (30K), Populus trichocarpa (35K), Vitis vinifera (37K), plus ~50 more.

**Not in graph:** Hordeum vulgare, Brachypodium sylvaticum. For these species, use orthologue chains via Brachypodium distachyon or Arabidopsis.

## R Integration (httr/jsonlite)

```r
library(httr)
library(jsonlite)

plantgraph_query <- function(cypher) {
  resp <- POST(
    "https://plantgraph.se/api/v1/graph/query",
    body = list(query = cypher),
    encode = "json",
    content_type_json()
  )
  content(resp, as = "parsed", simplifyVector = TRUE)$results
}

# Example: get GO terms for a Brachypodium gene
plantgraph_query('
  MATCH (l:Locus {id: "BRADI_1g16610v3"})-[:ANNOTATED_WITH]->(go:GO)
  RETURN go.id, go.name
')
```

## Cypher Best Practices (for PlantGraph queries)

Based on [neo4j-cypher-guide](https://github.com/tomasonjo/blogs) modern Cypher patterns:

- **NULL filtering on sorts**: Always add `WHERE prop IS NOT NULL` or `ORDER BY prop NULLS LAST` when sorting
- **Use `count{}` not `size()`**: `WHERE count{(l)-[:ANNOTATED_WITH]->()} > 3` instead of `size((l)-[:ANNOTATED_WITH]->())`
- **Use `EXISTS{}`**: `WHERE EXISTS {MATCH (l)-[:INTERACTS_WITH]->()}` instead of deprecated `exists()`
- **Explicit grouping**: Use `WITH` clauses before aggregations to make grouping keys explicit
- **`elementId()` not `id()`**: The `id()` function is deprecated; use `elementId()` (returns string)
- **`coalesce()` for optional props**: `coalesce(l.name, l.id)` handles species where `l.name` is null
- **Always LIMIT**: This graph has 1.6M loci; unbounded queries will be slow

### Query checklist
1. No deprecated functions (`id()`, `size()` for patterns, `exists()`)
2. NULL filters on all sorted properties
3. LIMIT on all exploratory queries
4. Undirected match for reverse orthologue lookups
5. `coalesce(t.name, t.id)` when displaying cross-species results

## Common Mistakes

- Using `l.name` for non-Arabidopsis species (use `l.id`)
- Using Phytozome `Bradi1g*` format (must convert to `BRADI_1g*v3`)
- Using MSU rice IDs `LOC_Os*` (PlantGraph uses RAP format `Os*`)
- Expecting barley (`HORVU.*`) to exist (it doesn't; use orthologue hop)
- Using directed `->` for reverse orthologue lookups. `ORTHOLOGOUS_TO` edges radiate FROM Arabidopsis. To find the Ath orthologue of a Bdis gene, use undirected `-[:ORTHOLOGOUS_TO]-` (no arrow)
- Querying regulatory network in wrong direction. `REGULATES`, `ACTIVATES`, `REPRESSES`, `DAPSEQ_BINDS_PROMOTER` edges point FROM the regulator TO the target. To find "who regulates gene X", use `(tf)-[:REGULATES]->(l {name: "X"})` (incoming to your gene)
- Using `EXPRESSED_IN_TISSUE` with `(t:Tissue)` -- the target is actually `DevelopmentalStage`, not `Tissue`. Use `(t)` without label constraint
- Forgetting `LIMIT` on broad queries (the graph is large). `INTERACTS_WITH` and `COEXPRESSES_WITH` can return 100K+ rows
- Rice species string is `"Oryza sativa"` (13K loci) or `"Oryza sativa subsp. japonica"` (22K loci) -- check both
