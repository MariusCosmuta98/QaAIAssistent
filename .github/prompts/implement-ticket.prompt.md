---
description: "Implement a Jira ticket end-to-end: fetch ticket + Zephyr tests + Figma, then write code/tests in the host project's style."
agent: "qa-orchestrator"
argument-hint: "JIRA-KEY or Jira URL"
---

Implement the ticket: **${input:ticket:Jira ticket key (ABC-123) or full URL}**.

Rules:
1. Extract the Jira key from the input (key or URL).
2. Delegate to `jira-fetcher` first. From its `Links:` line:
   - call `zephyr-fetcher` only if Zephyr ids/cycle were extracted; pass those ids (not the Jira key).
   - call `figma-fetcher` only if a Figma URL was extracted.
3. Do NOT read the skill files yourself — the subagents handle that.
4. Output only the `Ticket / Plan / Changes / Run` block.
