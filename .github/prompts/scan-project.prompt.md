---
description: "Scan the host project and (re)generate .github/PROJECT_CONTEXT.md."
agent: "project-scanner"
argument-hint: "(no args needed)"
---

Scan this workspace and refresh **`.github/PROJECT_CONTEXT.md`** following your procedure.

Rules:
- Target file path is **`.github/PROJECT_CONTEXT.md`** (absolute path: use the workspace root + `.github/PROJECT_CONTEXT.md`).
- Prefer `replace_string_in_file` to edit the file. If it fails, use `create_file` to overwrite the file — this is explicitly allowed for this file only.
- You MUST call `read_file` on `.github/PROJECT_CONTEXT.md` AFTER writing to verify the content changed. If the file still contains `> Not yet generated`, your write failed — retry with `create_file`.
- Keep it under 80 lines.
- Fill the `## Repo Setup` section with verified facts only (runtimes, env files, first-run sequence, CI entrypoint). Leave bullets blank if not verifiable.
- Do not modify any source files.
