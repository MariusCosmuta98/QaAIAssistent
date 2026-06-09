---
description: "Implement a Jira ticket end-to-end: fetch ticket + Zephyr tests, then write code/tests in the host project's style."
agent: "qa-orchestrator"
argument-hint: "JIRA-KEY or Jira URL"
---

Implement the ticket: **${input:ticket:Jira ticket key (ABC-123) or full URL}**.

Follow the Anti-Hallucination Rules in [../AGENTS.md](../AGENTS.md) and the orchestrator's procedure in [../agents/qa-orchestrator.agent.md](../agents/qa-orchestrator.agent.md).

Non-negotiable reminders:
- Read `.github/PROJECT_CONTEXT.md` first. If it still says `> Not yet generated`, stop and ask the user to run `/scan-project`.
- Delegate to `jira-fetcher` (and `zephyr-fetcher` only if Zephyr ids were found).
- Verify every file write by reading it back.
- End by writing `/memories/repo/ticket-<KEY>.md` and printing only the `Ticket / Plan / Changes / Memory / Run` block.
