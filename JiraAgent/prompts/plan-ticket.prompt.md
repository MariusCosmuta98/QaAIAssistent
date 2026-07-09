---
description: "Generate a Markdown test-plan folder from a Jira ticket. Standalone JiraAgent flow — does not touch the codebase, does not push to Zephyr."
agent: "test-plan-generator"
argument-hint: "JIRA-KEY"
---

Generate a test plan for **${input:ticketKey:Jira ticket key}**.

## Output

You will create:
- `tests/plans/${input:ticketKey}/test-plan.md` (index with all ACs and TC traceability)
- `tests/plans/${input:ticketKey}/TC-01-<slug>.md`, `TC-02-<slug>.md`, … (one per test case)

## Rules

1. **Delegate to jira-reader** — do NOT read Jira directly.
2. **One primary TC per AC**, plus at most one negative TC per AC.
    - Add negative TC only if the AC contains: "reject", "invalid", "must not", "cannot", "error", "fail", "deny", "unless", "except", "only if".
    - Otherwise, emit only the primary TC.
3. **If the ticket has no ACs**, you MUST draft 3–7 conservative, testable ACs and ask me to validate before writing files.
4. **Mark `NEEDS-CLARIFICATION`** if a TC cannot be tested because:
    - AC uses vague metrics ("fast", "large") without numbers.
    - AC depends on undefined configs or environments.
    - Expected result is subjective ("looks good", user is happy") instead of observable.
    - Preconditions are unclear (e.g., "admin account" without setup steps).
5. **Never invent** thresholds, timeouts, environments, or criteria not in the ticket. If missing, mark `NEEDS-CLARIFICATION`.
6. Verify every file with `read_file` after writing.
7. Emit the standard traceability summary.

## Constraints

- Do NOT touch the codebase. Do NOT push to Zephyr. Do NOT read `.github/PROJECT_CONTEXT.md`.
- If `tests/plans/${input:ticketKey}/` exists and I did NOT say "refresh", stop immediately and report the path.
