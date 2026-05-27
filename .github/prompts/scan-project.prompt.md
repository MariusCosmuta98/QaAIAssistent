---
description: "Scan the host project and (re)generate .github/PROJECT_CONTEXT.md."
agent: "project-scanner"
argument-hint: "(no args needed)"
---

Scan this workspace and refresh **`.github/PROJECT_CONTEXT.md`** following your procedure.

Rules:
- Target file path is **`.github/PROJECT_CONTEXT.md`** (it already exists from the template — use your edit tool to update it, never a create-file tool).
- Keep it under 80 lines.
- Fill the `## Repo Setup` section with verified facts only (runtimes, env files, first-run sequence, CI entrypoint). Leave bullets blank if not verifiable.
- Do not modify any source files.
