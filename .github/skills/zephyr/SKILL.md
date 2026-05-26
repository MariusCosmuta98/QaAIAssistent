---
name: zephyr
description: "Use when fetching, listing, or creating Zephyr Scale test cases linked to a Jira ticket or cycle."
---

# Zephyr (Scale)

## When to Use
- A Jira ticket needs its test cases.
- A Zephyr cycle/folder id is provided.
- New test cases must be created from acceptance criteria.

## Tools
Prefer Zephyr MCP server (`zephyr/*`) if configured. Fallback: Zephyr Scale REST API `https://api.zephyrscale.smartbear.com/v2/`.
Auth: `ZEPHYR_TOKEN` env var as `Bearer`.

## Procedure
### Read test cases for a Jira key
1. `GET /testcases?jql=issueKey = "<KEY>"` (or `/issuelinks/testcases?issueKey=<KEY>`).
2. For each case, fetch `/testcases/{key}/teststeps` for steps + expected. **Batch**: issue step-fetch requests in parallel (not sequentially) to reduce round-trip reasoning overhead.
3. Return in the [zephyr-fetcher agent](../../agents/zephyr-fetcher.agent.md) format. Drop all fields not in the output template (e.g. `createdOn`, `owner`, `customFields`).

### Create a new test case
1. `POST /testcases` with: `projectKey`, `name`, `objective`, `precondition`, `priorityName`.
2. `POST /testcases/{key}/teststeps` with the ordered steps.
3. Link to Jira: `POST /testcases/{key}/links/issues` with the Jira key.

## Required Config (see [.env](../../../.env))
- `ZEPHYR_BASE_URL` — defaults to `https://api.zephyrscale.smartbear.com/v2`
- `ZEPHYR_TOKEN` — API token (Bearer)
- `ZEPHYR_PROJECT_KEY` — Zephyr project key (may differ from Jira)

## Pitfalls
- Project key in Zephyr ≠ Jira project key sometimes — confirm via env var `ZEPHYR_PROJECT_KEY`.
- Steps API is paginated; always loop until `isLast`.
- Do not create duplicates — search by name first.
