# plantgraph-database

A Claude Code skill for querying PlantGraph (plantgraph.se), a Neo4j knowledge graph of plant gene annotations, interactions, co-expression, orthologues, GO terms, and pathways.

PlantGraph covers 1.6M loci across 60+ plant species, with 132 node types and 149 relationship types. Built and maintained at UPSC (Umeå Plant Science Centre). No authentication required.

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

Major species: Arabidopsis thaliana, Brassica napus, Triticum aestivum, Oryza sativa, Brachypodium distachyon, Zea mays, Populus trichocarpa, and 50+ more.

Not in graph: Hordeum vulgare, Brachypodium sylvaticum. The skill documents orthologue chain strategies for these.

## Installation

Copy `SKILL.md` to your Claude Code skills directory:

```bash
mkdir -p ~/.claude/skills/plantgraph-database
cp SKILL.md ~/.claude/skills/plantgraph-database/SKILL.md
```

## Usage

In Claude Code, invoke with `/plantgraph-database` when you need to query plant gene data. The skill provides:

- API endpoint formats and curl examples
- ID format rules per species (Arabidopsis uses `l.name`, others use `l.id`)
- Phytozome-to-PlantGraph ID conversion tables
- Ready-to-use Cypher query patterns for common tasks
- R integration code (httr/jsonlite)
- Common mistakes and how to avoid them

## License

MIT
