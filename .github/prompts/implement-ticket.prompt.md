---
description: "Implement a Jira ticket end-to-end: fetch ticket + Zephyr tests, then write code/tests in the host project's style."
agent: "qa-orchestrator"
argument-hint: "JIRA-KEY or Jira URL"
---

Implement the ticket: **${input:ticket:Jira ticket key (ABC-123) or full URL}**.

## Mandatory rules — violating any one invalidates the run

1. **No hallucination.** Every fact you use (acceptance criteria, API endpoints, selectors, file contents) MUST come from an actual tool call response. If you catch yourself writing a `<tool_result>` block, you are hallucinating — stop immediately.
2. **Actually call tools.** Use `read_file` to read files. Use `runSubagent` to delegate to `jira-fetcher` and `zephyr-fetcher`. Wait for the real response before proceeding.
3. **Read `.github/PROJECT_CONTEXT.md` first** (stop if missing or unfilled — tell the user to run `/scan-project`).
4. **Delegate to `jira-fetcher`** via `runSubagent`. Read its actual response. Do not invent ticket content.
5. **Sibling memories are optional.** Only read `/memories/repo/ticket-<KEY>.md` files that actually exist (read_file succeeds). If a file doesn't exist, skip it — do not fabricate its contents. Max 3 siblings.
6. **Call `zephyr-fetcher`** only if the jira-fetcher output lists Zephyr ids (not `none`).
7. **Verify every file write** by reading the file back afterward.
8. **Write `/memories/repo/ticket-<KEY>.md`** at the end using the Ticket Memory Template.
9. **Output only** the `Ticket / Reused siblings / Plan / Changes / Memory / Run` block.
