# QA AI Assistant

Portable QA assistant. Drop into any project. Helps QA engineers implement tickets by:

1. Scanning the host project once → single context file.
2. Fetching Jira (+ Zephyr / Figma / Confluence found inside the ticket).
3. Implementing or scaffolding tests using that combined context.

## Architecture

One agent = one job. The orchestrator delegates; it does not fetch.

| Step | Owner | Output |
|------|-------|--------|
| Understand the host project | [project-scanner](agents/project-scanner.agent.md) | [PROJECT_CONTEXT.md](PROJECT_CONTEXT.md) |
| Read a Jira ticket | [jira-fetcher](agents/jira-fetcher.agent.md) | Ticket summary + auto-extracted links |
| Read Zephyr test cases | [zephyr-fetcher](agents/zephyr-fetcher.agent.md) | Test case list |
| Read Figma design | [figma-fetcher](agents/figma-fetcher.agent.md) | UI spec |
| Implement / scaffold | [qa-orchestrator](agents/qa-orchestrator.agent.md) | Code + tests |

Integration knowledge → [skills/](./skills/). Slash commands → [prompts/](./prompts/).

## Golden Rules

- **One agent = one job.** Orchestrator delegates only.
- **Read [PROJECT_CONTEXT.md](PROJECT_CONTEXT.md) once per task.** If missing or unfilled, stop and ask the user to run `/scan-project`. **Never auto-run it.**
- **Return only relevant info.** Code + minimal rationale. No essays, no restating the ticket.
- **Match the host project's conventions** (from `PROJECT_CONTEXT.md`).

## Token Budget

To keep runs cheap:
- `/scan-project` is **manual-only**. Run it once per repo (and re-run only when the stack changes).
- `jira-fetcher` auto-extracts Zephyr / Figma / Confluence ids from the ticket. The orchestrator calls Zephyr/Figma **only if** ids were found — never speculatively.
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
- Figma:     `/memories/session/figma-<nodeId-or-urlSlug>.md`

Protocol (every fetcher follows it):
1. **Before** discovering tools, check if the cache file exists. If yes → return its contents verbatim. No API call.
2. **After** a successful fetch, write the formatted output to the cache file (overwrite).
3. The user can invalidate by saying "refresh <KEY>" — fetchers then skip step 1.

The orchestrator does not manage the cache; each fetcher owns its own file.

## Per-Ticket Memory (`/memories/repo/`)

Persistent across conversations — survives between `/implement` runs so later tickets can build on earlier ones.

File naming: `/memories/repo/ticket-<KEY>.md` (one file per implemented Jira ticket).

Lifecycle:
1. `jira-fetcher` lists sibling tickets under the same epic and notes which ones have a memory file → `Implemented Siblings:` block.
2. `qa-orchestrator` reads those sibling memories to align scope/patterns, then implements.
3. `qa-orchestrator` writes the new ticket's memory at the end of `/implement` using the Ticket Memory Template (see orchestrator agent).

Keep each memory file ≤ 40 lines. It will be loaded verbatim by future runs.

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
- Figma: always target specific node ids with `depth≤3`. Never fetch the full file.

### Cap & terminate pagination early (Zephyr fallback)
- Use small page sizes (`maxResults=50`, not 200) so less data enters context per page.
- Stop paginating as soon as all expected matches are found.
- Hard cap: **5 pages** (250 cases). If no match by then, report `none` — do not crawl the entire project.

### Summarize before injecting (Confluence, Figma)
- Confluence pages longer than **500 words**: summarize to ≤200 words before returning. Provide a reference id so the full page can be fetched on demand.
- Figma node trees: flatten to the compact output format immediately. Never pass raw JSON into context.

### Batch where possible (Zephyr)
- When fetching steps for multiple test cases, issue requests in **parallel** rather than sequentially. Each round-trip costs reasoning tokens for the call + response.

### Prune stale context (orchestrator)
- After building the plan (step 5), carry forward only the **plan lines and PROJECT_CONTEXT.md** into the implementation step. Summarize fetcher outputs into the plan — do not keep full fetcher blocks in working memory.

### Avoid redundant fetches (all agents)
- Always check `/memories/session/` cache before calling any API.
- The Zephyr fallback in `jira-fetcher` already discovers test case keys. Pass those keys to `zephyr-fetcher` — it must not re-search by Jira key.
