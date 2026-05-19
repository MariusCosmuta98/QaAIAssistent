# QA AI Assistant

Portable QA assistant. Drop into any project. Helps QA engineers implement tickets by:

1. Scanning the host project once → single context file.
2. Fetching Jira (+ Zephyr / Figma / Confluence found inside the ticket).
3. Implementing or scaffolding tests using that combined context.

## Architecture

One agent = one job. The orchestrator delegates; it does not fetch.

| Step | Owner | Output |
|------|-------|--------|
| Understand the host project | [project-scanner](./agents/project-scanner.agent.md) | [PROJECT_CONTEXT.md](./PROJECT_CONTEXT.md) |
| Read a Jira ticket | [jira-fetcher](./agents/jira-fetcher.agent.md) | Ticket summary + auto-extracted links |
| Read Zephyr test cases | [zephyr-fetcher](./agents/zephyr-fetcher.agent.md) | Test case list |
| Read Figma design | [figma-fetcher](./agents/figma-fetcher.agent.md) | UI spec |
| Implement / scaffold | [qa-orchestrator](./agents/qa-orchestrator.agent.md) | Code + tests |

Integration knowledge → [skills/](./skills/). Slash commands → [prompts/](./prompts/).

## Golden Rules

- **One agent = one job.** Orchestrator delegates only.
- **Read [PROJECT_CONTEXT.md](./PROJECT_CONTEXT.md) once per task.** If missing or unfilled, stop and ask the user to run `/scan-project`. **Never auto-run it.**
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
