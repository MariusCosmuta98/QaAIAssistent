---
name: zephyr
description: "Use when fetching, listing, or creating Zephyr Scale test cases linked to a Jira ticket or cycle. Covers the read/write procedures and output shape. For connection setup (MCP/REST/auth/env) see the zephyr-connection skill."
---

# Zephyr (Scale) — Read & Write

This skill covers **how to read and create Zephyr Scale test cases**. For connection setup (MCP discovery, REST fallback, auth, env vars) see [zephyr-connection skill](../zephyr-connection/SKILL.md).

## When to Use
- A Jira ticket needs its test cases.
- A Zephyr cycle/folder id is provided.
- New test cases must be created from acceptance criteria.

## Procedure
### Read test cases for a Jira key
1. Establish a connection per [zephyr-connection skill](../zephyr-connection/SKILL.md).
2. `GET /testcases?jql=issueKey = "<KEY>"` (or `/issuelinks/testcases?issueKey=<KEY>`).
3. For each case, fetch `/testcases/{key}/teststeps` for steps + expected. **Batch**: issue step-fetch requests in parallel (not sequentially) to reduce round-trip reasoning overhead.
4. Return in the [zephyr-fetcher agent](../../agents/zephyr-fetcher.agent.md) format. Drop all fields not in the output template (e.g. `createdOn`, `owner`, `customFields`).

### Create a new test case
1. Establish a connection per [zephyr-connection skill](../zephyr-connection/SKILL.md).
2. `POST /testcases` with: `projectKey`, `name`, `objective`, `precondition`, `priorityName`.
3. `POST /testcases/{key}/teststeps` with the ordered steps.
4. Link to Jira: `POST /testcases/{key}/links/issues` with the Jira key.

## Pitfalls
- Steps API is paginated; always loop until `isLast`.
- Do not create duplicates — search by name first.
- Project key in Zephyr may differ from the Jira project key — see the zephyr-connection skill.
