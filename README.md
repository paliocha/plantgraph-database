# plantgraph-database

A Claude Code skill for querying [PlantGraph](https://plantgraph.se) (plantgraph.se) — a Neo4j knowledge graph and REST API platform for plant gene annotations, interactions, co-expression, orthologues, GO terms, pathways, protein structures, and enrichment data.

PlantGraph contains 14.5M+ nodes and 47.9M+ relationships across 127 plant species, with 132 node types and 149 relationship types. Integrates 140+ data sources with 42 real-time enrichment services. Built and maintained at UPSC (Umeå Plant Science Centre).

## What's in the graph

- Gene annotations (GO, PO, protein domains, MapMan bins, pathways)
- Protein-protein interactions and co-expression networks
- Cross-species orthologues and paralogues
- Transcription factor binding (including DAP-seq validated)
- GWAS trait associations
- Histone marks and other epigenetic data
- Single-cell expression profiles
- Nutrient and pathogen response data
- Publications and literature links

## Species coverage

Major species: Arabidopsis thaliana, Brassica napus, Triticum aestivum, Oryza sativa (japonica + indica), Brachypodium distachyon, Zea mays, Populus trichocarpa, and 120+ more.

Not in graph: Hordeum vulgare, Brachypodium sylvaticum. The skill documents orthologue chain strategies for these.

## Installation

Copy `SKILL.md` to your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/plantgraph-database
cp SKILL.md ~/.claude/skills/plantgraph-database/SKILL.md
```

## Usage

In Claude Code, invoke with `/plantgraph-database` when you need to query plant gene data. The skill provides:

- JWT authentication setup and rate limits
- Complete REST API reference (Graph, Enrichment, AI, Paper Evidence endpoints)
- 42 real-time enrichment sources (UniProt, STRING, AlphaFold, KEGG, etc.)
- AI features: question generation, hypothesis generation, GNN predictions, chat
- Paper evidence verification (abstract/PDF scoring, claim verification via SSE)
- ID format rules per species (Arabidopsis uses `l.name`, others use `l.id`)
- Phytozome-to-PlantGraph ID conversion tables
- Ready-to-use Cypher query patterns for common tasks
- Full schema reference: 132 node labels, 149 relationship types with counts
- R and Python integration code with auth
- Common mistakes and how to avoid them

## License

MIT
