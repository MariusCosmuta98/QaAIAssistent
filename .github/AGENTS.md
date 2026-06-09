# QA AI Assistant

Portable QA assistant. Drop into any project. Helps QA engineers implement tickets by:

1. Scanning the host project once → single context file.
2. Fetching Jira (+ Zephyr test cases found inside the ticket).
3. Implementing or scaffolding tests using that combined context.

## Architecture


| Step | Owner | Output |
|------|-------|--------|
| Understand the host project | [project-scanner](agents/project-scanner.agent.md) | [PROJECT_CONTEXT.md](PROJECT_CONTEXT.md) |
| Read a Jira ticket | [jira-fetcher](agents/jira-fetcher.agent.md) | Ticket summary + auto-extracted links |
| Read Zephyr test cases | [zephyr-fetcher](agents/zephyr-fetcher.agent.md) | Test case list |
| Implement / scaffold | [qa-orchestrator](agents/qa-orchestrator.agent.md) | Code + tests |

Integration knowledge → [skills/](skills/). Slash commands → [prompts/](prompts/).

## Golden Rules

- **No hallucination.** Every fact used (ticket fields, file contents, API responses) MUST come from an actual tool call. Never fabricate tool results, file contents, or API responses. If a tool call fails, report the error — do not invent a successful response.
- **One agent = one job.** Orchestrator delegates only.
- **Read [PROJECT_CONTEXT.md](PROJECT_CONTEXT.md) once per task.** If missing or unfilled, stop and ask the user to run `/scan-project`. **Never auto-run it.**
- **Return only relevant info.** Code + minimal rationale. No essays, no restating the ticket.
- **Match the host project's conventions** (from `PROJECT_CONTEXT.md`).
- **Verify writes.** After writing any file, read it back to confirm the content was saved.

## Anti-Hallucination Rules (canonical)

These apply to **every** agent and prompt in this pack. Agent/prompt files reference this section instead of repeating it.

1. **NEVER fabricate tool results.** If you call a subagent, tool, or REST endpoint, you MUST wait for the actual response. If you find yourself writing a `<tool_result>` block, you are hallucinating — STOP immediately.
2. **NEVER assume file contents.** If you need data from a file, call `read_file` and wait for the real output. If the file does not exist, treat it as non-existent — do not invent its contents.
3. **NEVER invent acceptance criteria, test steps, API endpoints, selectors, or any detail that did not come from the actual Jira ticket or actual project files.** If you don't have it, ask the user.
4. **NEVER skip subagent delegation.** The orchestrator MUST call `runSubagent` for Jira/Zephyr work and wait for the real response. If a subagent returns an error, report it — do not fabricate success.
5. **After every file write, call `read_file` on the written file to verify content.** If verification fails, retry once, then stop and ask the user.

## Token Budget

To keep runs cheap:
- `/scan-project` is **manual-only**. Run it once per repo (and re-run only when the stack changes).
- `jira-fetcher` auto-extracts Zephyr test case ids from the ticket. The orchestrator calls Zephyr **only if** ids were found — never speculatively.
- `zephyr-fetcher` queries by extracted ids directly; it does not also search by Jira key.
- Subagents return compact fixed-format blocks. Do not re-read skill files from the orchestrator.
- Do not re-read `PROJECT_CONTEXT.md` mid-task. Read once.
- `/review-ticket` skips the orchestrator AND `PROJECT_CONTEXT.md` entirely — it calls `jira-fetcher` directly.
- Cap Zephyr results at 10 cases; cap `PROJECT_CONTEXT.md` at ~80 lines.

## Session Cache (`/memories/session/`)

Fetcher outputs are cached for the duration of the conversation so the same ticket isn't fetched twice.

Cache file naming:
- Jira:      `/memories/session/jira-<KEY>.md`
- Zephyr:    `/memories/session/zephyr-<KEY-or-cycleId>.md`

Protocol (every fetcher follows it):
1. **Before** discovering tools, check if the cache file exists. If yes → return its contents verbatim. No API call.
2. **After** a successful fetch, write the formatted output to the cache file (overwrite).
3. The user can invalidate by saying "refresh <KEY>" — fetchers then skip step 1.

The orchestrator does not manage the cache; each fetcher owns its own file.

## Per-Ticket Memory (`/memories/repo/`)

Persistent across conversations — survives between `/implement` runs as a record of what was done per ticket.

File naming: `/memories/repo/ticket-<KEY>.md` (one file per implemented Jira ticket).

Lifecycle:
1. `qa-orchestrator` writes the ticket's memory at the end of `/implement` using the Ticket Memory Template (see orchestrator agent).

Keep each memory file ≤ 40 lines.

## Knowledge Base & Retrieval (RAG layer)

The pack uses a lightweight RAG layout. See [knowledge/README.md](knowledge/README.md) for the full design.

Flow on every `/implement`:
1. **Retrieve** — orchestrator calls `context-retriever` (Step 1.5) to score [knowledge/INDEX.md](knowledge/INDEX.md) rows by lexical overlap with the task; reads only the top-K chunks.
2. **Use** — those chunks are cited in the final `Sources:` output line.
3. **Learn** — at the end (Step 10), `lesson-recorder` distills at most one durable lesson into `/memories/repo/lessons/<slug>.md` and appends a row to `INDEX.md`. May decline with `NO_LESSON`.
4. **Curate** — the manual `/curate` command reports stale or broken chunks; archival happens only with user approval.

Rules:
- Retriever is offline & lexical — no embeddings, no API calls.
- Lessons are ≤ 10 body lines and must cite the source ticket.
- INDEX.md is the single source of truth for what is retrievable.

## Path Rules

All assistant config lives under **`.github/`**, never the workspace root:
- `.github/PROJECT_CONTEXT.md` — project context (read once per task)
- `.github/agents/*.agent.md` — subagent definitions
- `.github/skills/<topic>/SKILL.md` — integration knowledge
- `.github/prompts/*.prompt.md` — slash commands

If `.github/PROJECT_CONTEXT.md` cannot be read, **stop** and ask the user to run `/scan-project`. Never fall back to `./PROJECT_CONTEXT.md` at the workspace root.

## Token Optimization Rules

These rules apply to all agents and skills:

### Filter API responses (all fetchers)
- When using REST fallbacks, always request **only needed fields** (e.g. Jira `&fields=summary,status,...`). Never fetch a full object when a subset suffices.

### Cap & terminate pagination early (Zephyr fallback)
- Use small page sizes (`maxResults=50`, not 200) so less data enters context per page.
- Stop paginating as soon as all expected matches are found.
- Hard cap: **5 pages** (250 cases). If no match by then, report `none` — do not crawl the entire project.

### Batch where possible (Zephyr)
- When fetching steps for multiple test cases, issue requests in **parallel** rather than sequentially. Each round-trip costs reasoning tokens for the call + response.

### Prune stale context (orchestrator)
- After building the plan (step 5), carry forward only the **plan lines and PROJECT_CONTEXT.md** into the implementation step. Summarize fetcher outputs into the plan — do not keep full fetcher blocks in working memory.

### Avoid redundant fetches (all agents)
- Always check `/memories/session/` cache before calling any API.
- The Zephyr fallback in `jira-fetcher` already discovers test case keys. Pass those keys to `zephyr-fetcher` — it must not re-search by Jira key.
