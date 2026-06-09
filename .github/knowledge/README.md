# Knowledge Base

Curated, retrievable knowledge for the QA AI Assistant. This is the **RAG layer**:
the orchestrator queries the [context-retriever](../agents/context-retriever.agent.md)
before doing any real work and pulls only the top-K relevant chunks into context.

## Tiers

| Tier | Path | Lifetime | Who writes |
|---|---|---|---|
| **Project context** | [../PROJECT_CONTEXT.md](../PROJECT_CONTEXT.md) | per repo | `project-scanner` (manual) |
| **Skills** | [../skills/](../skills/) | versioned with the pack | maintainers |
| **Lessons** | `/memories/repo/lessons/<slug>.md` | persistent, per repo | `lesson-recorder` (auto, end of `/implement`) |
| **Per-ticket memory** | `/memories/repo/ticket-<KEY>.md` | persistent, per repo | `qa-orchestrator` |
| **Session cache** | `/memories/session/*.md` | per conversation | fetchers |

## INDEX.md

[INDEX.md](INDEX.md) is a flat lookup table of every retrievable chunk.
The retriever reads only this file; it never scans the corpus blindly.

Format — one row per chunk:

```
| id | topic | path | summary |
```

Rules:
- `id` is unique and stable. Use kebab-case (e.g. `lesson-2026-06-09-csv-bom`).
- `topic` is a comma-separated tag list used by the retriever for keyword overlap.
- `path` is workspace-relative.
- `summary` is one line, ≤ 100 chars, plain English.

Add a row whenever a new lesson is written or a new skill is introduced.
Remove a row when the file is archived or deleted.

## Chunk frontmatter

Every chunk file SHOULD start with YAML frontmatter so the curator can prune intelligently:

```yaml
---
id: lesson-2026-06-09-csv-bom
topic: [csv, encoding, excel]
source: ticket ABC-123
confidence: high     # high | medium | low
last_used: 2026-06-09
---
```

`last_used` is bumped whenever the retriever returns the chunk. Chunks not used in
90 days are archived by `/curate`.

## Anti-rot rules

- **Distill, don't dump.** A lesson is one fact in ≤ 10 lines. Longer = it's documentation, not a lesson.
- **Cite the source.** Every lesson must reference the ticket / file / commit it came from.
- **No verbatim ticket bodies.** The KB is for *learnings*, not a Jira mirror.
- **One topic per file.** Easier to retrieve, easier to retire.
