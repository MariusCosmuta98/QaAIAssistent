---
description: "Implement a Jira ticket end-to-end: fetch ticket + Zephyr tests + Figma, then write code/tests in the host project's style."
agent: "qa-orchestrator"
argument-hint: "JIRA-KEY or Jira URL"
---

Implement the ticket: **${input:ticket:Jira ticket key (ABC-123) or full URL}**.

Rules:
1. All context/agent/skill files live under **`.github/`** — never look in the workspace root.
   - Read project context from `.github/PROJECT_CONTEXT.md` (stop if missing or unfilled — tell the user to run `/scan-project`).
   - Subagent definitions are at `.github/agents/*.agent.md`.
2. Extract the Jira key from the input (key or URL).
3. Delegate to `jira-fetcher` first. From its output:
   - If `Implemented Siblings:` lists keys, load each `/memories/repo/ticket-<KEY>.md` and reuse their scope/patterns where they fit.
   - From `Links:`: call `zephyr-fetcher` only if Zephyr ids/cycle were extracted (pass those ids, not the Jira key). Call `figma-fetcher` only if a Figma URL was extracted.
4. Do NOT read the skill files yourself — the subagents handle that.
5. After implementing, write `/memories/repo/ticket-<KEY>.md` using the Ticket Memory Template from the orchestrator agent.
6. Output only the `Ticket / Reused siblings / Plan / Changes / Memory / Run` block.
