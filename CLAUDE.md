# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A Claude Code skill (`SKILL.md`) for querying PlantGraph (plantgraph.se), a public Neo4j knowledge graph of plant gene annotations, interactions, co-expression, orthologues, GO terms, and pathways. 1.6M loci across 60+ plant species. No authentication required.

This repo has no build system, tests, or compiled code. The deliverable is `SKILL.md` — a skill definition file users copy to `~/.claude/skills/plantgraph-database/`.

## Repo Structure

- `SKILL.md` — The skill definition. Contains API endpoints, Cypher query patterns, ID format rules, species coverage, and R integration code. This is the core artifact.
- `README.md` — Installation and usage docs.

## Editing SKILL.md

The skill frontmatter (`name`, `description`) controls when Claude Code invokes it. The `description` field must contain trigger keywords (species names, "PlantGraph", "Neo4j knowledge graph", etc.).

Key constraints to preserve when editing:
- Arabidopsis uses `l.name` for lookups; all other species use `l.id`
- `ORTHOLOGOUS_TO` edges point FROM Arabidopsis outward — reverse lookups need undirected match
- Phytozome-to-PlantGraph ID conversion rules (e.g., `Bradi1g*` → `BRADI_1g*v3`)
- Species not in graph (barley, B. sylvaticum) have documented orthologue chain strategies
- Cypher must follow Neo4j 5+ patterns (`count{}` not `size()`, `elementId()` not `id()`)

## Installation (for users)

```bash
mkdir -p ~/.claude/skills/plantgraph-database
cp SKILL.md ~/.claude/skills/plantgraph-database/SKILL.md
```

Invoke with `/plantgraph-database` in Claude Code.
