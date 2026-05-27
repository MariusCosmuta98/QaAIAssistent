---
description: "Generate Zephyr test cases from a Jira ticket's acceptance criteria (does not write code)."
agent: "qa-orchestrator"
argument-hint: "JIRA-KEY"
---

Generate test cases for **${input:ticketKey:Jira ticket key}**.

Rules:
- Project context lives at **`.github/PROJECT_CONTEXT.md`** (not the workspace root).
- Agents live under **`.github/agents/`**.

Steps:
1. Delegate to `jira-fetcher` for acceptance criteria. If it lists `Implemented Siblings:`, peek at their `/memories/repo/ticket-<KEY>.md` to avoid duplicating existing test coverage.
2. Delegate to `zephyr-fetcher` to check what already exists — do not duplicate.
3. Propose new test cases (id placeholder, name, preconditions, steps, expected) in the Zephyr output format.
4. Ask me before pushing them to Zephyr.
