---
description: "Implement a Jira ticket end-to-end: fetch ticket + Zephyr tests + Figma, then write code/tests in the host project's style."
agent: "qa-orchestrator"
argument-hint: "JIRA-KEY (e.g. ABC-123)"
---

Implement the ticket: **${input:ticketKey:Jira ticket key (e.g. ABC-123)}**.

Steps:
1. Make sure `.github/PROJECT_CONTEXT.md` exists. If not, stop and tell me to run `/scan-project`.
2. Delegate to `jira-fetcher`, `zephyr-fetcher`, and (if a Figma link exists) `figma-fetcher` — in parallel.
3. Produce a short plan, then implement tests following the project's conventions.
4. Reply in your standard output format. No extra commentary.
