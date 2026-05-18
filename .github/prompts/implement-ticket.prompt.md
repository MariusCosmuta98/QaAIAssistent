---
description: "Implement a Jira ticket end-to-end: fetch ticket + Zephyr tests + Figma, then write code/tests in the host project's style."
agent: "qa-orchestrator"
argument-hint: "JIRA-KEY or Jira URL"
---

Implement the ticket: **${input:ticket:Jira ticket key (ABC-123) or full URL}**.

Rules:
1. Make sure `.github/PROJECT_CONTEXT.md` exists. If not, stop and tell me to run `/scan-project`.
2. Extract the Jira key from the input above (it can be a key or a URL).
3. Delegate to `jira-fetcher`, `zephyr-fetcher`, and (if a Figma link is found) `figma-fetcher`.
4. Do NOT read the skill files yourself — the subagents handle that.
5. Do NOT print `<tool_use>` text. Use real tool calls or stop with a clear error.
6. Output your standard `Ticket / Plan / Changes / Run` block. Nothing else.
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
