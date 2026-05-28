---
description: "Quick read-only summary of a Jira ticket + its Zephyr tests. No code changes."
agent: "jira-fetcher"
argument-hint: "JIRA-KEY"
---

Fetch **${input:ticketKey:Jira ticket key}** following your normal procedure (with session cache).

After returning your standard output block, additionally:
- If `zephyr:` lists ids/cycle, also call `zephyr-fetcher` with them.

Do NOT touch the codebase. Do NOT read `.github/PROJECT_CONTEXT.md`. Do NOT involve the orchestrator.
