# QA AI Assistant

This workspace ships a portable QA assistant that can be dropped into **any** project (any language, any framework). It helps QA engineers implement tickets faster by:

1. Scanning the host project and producing a single context file.
2. Fetching Jira ticket + Zephyr test cases + (optional) Figma + Confluence info.
3. Implementing or scaffolding the feature/test using that combined context.

## Architecture

Each step is owned by a **dedicated agent**. Do not merge responsibilities.

| Step | Owner | Output |
|------|-------|--------|
| Understand the host project | [project-scanner](./agents/project-scanner.agent.md) | [PROJECT_CONTEXT.md](./PROJECT_CONTEXT.md) |
| Read a Jira ticket | [jira-fetcher](./agents/jira-fetcher.agent.md) | Ticket summary |
| Read Zephyr test cases | [zephyr-fetcher](./agents/zephyr-fetcher.agent.md) | Test case list |
| Read Figma design | [figma-fetcher](./agents/figma-fetcher.agent.md) | UI spec |
| Implement / scaffold | [qa-orchestrator](./agents/qa-orchestrator.agent.md) | Code + tests |

Domain integration knowledge lives in [skills/](./skills/). User-facing entry points live in [prompts/](./prompts/).

## Golden Rules

- **One agent = one job.** The orchestrator delegates; it does not fetch directly.
- **Always read [PROJECT_CONTEXT.md](./PROJECT_CONTEXT.md) before writing code.** If missing or stale, run `/scan-project`.
- **Return only relevant info.** No essays, no restating the ticket. Code + minimal rationale.
- **Match the host project's conventions** (language, framework, test runner, folder layout) as captured in `PROJECT_CONTEXT.md`.
