# plantgraph-database

Claude Code skill for [PlantGraph](https://plantgraph.se) — Neo4j knowledge graph + REST API for plant gene annotations, interactions, co-expression, orthologues, GO terms, pathways, protein structures, and enrichment.

14.5M+ nodes, 47.9M+ relationships, 127 species, 132 node types, 149 relationship types. 140+ data sources, 42 real-time enrichment services. Built at UPSC (Umeå Plant Science Centre).

## Graph contents

- Gene annotations (GO, PO, protein domains, MapMan bins, pathways)
- Protein-protein interactions, co-expression networks
- Cross-species orthologues and paralogues
- TF binding (DAP-seq validated)
- GWAS trait associations
- Histone marks, epigenetic data
- Single-cell expression profiles
- Nutrient/pathogen response data
- 290K+ publications

## Species

Arabidopsis thaliana, Brassica napus, Triticum aestivum, Oryza sativa (japonica + indica), Brachypodium distachyon, Zea mays, Populus trichocarpa, and 120+ more.

Not in graph: Hordeum vulgare, Brachypodium sylvaticum — orthologue chain strategies documented.

## Install

```bash
mkdir -p ~/.claude/skills/plantgraph-database
cp SKILL.md ~/.claude/skills/plantgraph-database/SKILL.md
```

## What the skill covers

Invoke `/plantgraph-database` in Claude Code. Provides:

- JWT auth setup and rate limits
- REST API reference: Graph, Enrichment, AI, Paper Evidence endpoints
- 42 enrichment sources (UniProt, STRING, AlphaFold, KEGG, InterPro, Expression Atlas, ...)
- AI: question generation, hypothesis generation, GNN predictions, chat
- Paper evidence: abstract/PDF scoring, claim verification (SSE)
- ID format rules per species (Arabidopsis `l.name`, others `l.id`)
- Phytozome-to-PlantGraph ID conversion
- Cypher query patterns for common tasks
- Full schema: 132 node labels, 149 relationship types with counts
- R and Python integration with auth

## License

MIT
