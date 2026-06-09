---
description: "Audit the knowledge base. Lists stale lessons (last_used > 90 days) and asks which to archive."
agent: "context-retriever"
argument-hint: "(no args)"
---

Curate the knowledge base.

Steps:
1. Read `.github/knowledge/INDEX.md` and list every row.
2. For each row whose `path` starts with `/memories/repo/lessons/`, read the file
   and parse its `last_used:` frontmatter date.
3. Group rows into:
   - **Fresh** — `last_used` within the last 90 days.
   - **Stale** — older than 90 days, or `last_used` missing.
   - **Broken** — file not found at `path`.
4. Print a compact report:

   ```
   Knowledge audit (<n> total chunks)
   Fresh:   <n>
   Stale:   <n>
     - <id>  (last_used=<date>)
   Broken:  <n>
     - <id>  -> <path>
   ```

5. Ask the user which stale entries to archive (move file to
   `/memories/repo/lessons/archive/` and remove the INDEX row) and which broken
   entries to drop. **Do not modify anything without explicit approval.**

Read-only by default. No code changes.
