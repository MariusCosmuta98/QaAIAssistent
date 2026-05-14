---
description: "Quick read-only summary of a Jira ticket + its Zephyr tests + Figma links. No code changes."
agent: "qa-orchestrator"
argument-hint: "JIRA-KEY"
---

Summarize **${input:ticketKey:Jira ticket key}** for QA review only.

Delegate to `jira-fetcher`, `zephyr-fetcher`, and `figma-fetcher` (if linked). Combine outputs. Do NOT touch the codebase.
