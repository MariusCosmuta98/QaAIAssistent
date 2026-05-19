---
name: jira
description: "Use when reading or updating Jira issues (ticket key like ABC-123). Covers MCP/REST tool usage and which fields matter for QA work."
---

# Jira

## When to Use
- A ticket key is mentioned (e.g. `ABC-123`).
- The agent needs acceptance criteria, status, links to Zephyr/Figma/Confluence.

## Tools
Prefer the Atlassian MCP server if configured (`atlassian/*` or `mcp-atlassian/*`). Fallback: REST API `GET /rest/api/3/issue/{key}` with `JIRA_BASE_URL` + token from env.

## Procedure
1. Resolve ticket key (uppercase, e.g. `abc-123` → `ABC-123`).
2. Fetch the issue with fields: `summary,status,issuetype,description,customfield_acceptance,issuelinks,attachment,subtasks,comment`.
3. Run the **Zephyr / Figma / Confluence auto-extraction** below over every text field.
4. Return only the [jira-fetcher agent](../../agents/jira-fetcher.agent.md) output format.

## Auto-Extract Linked Resources
Scan `description`, `comment.comments[*].body`, `issuelinks[*]`, `remotelinks`, and any custom field text for:

### Zephyr
Pull **all** matches and de-duplicate. Pass them to the orchestrator so `zephyr-fetcher` can query directly (saves a round-trip search by Jira key).
- Test case keys: regex `\b[A-Z][A-Z0-9_]+-T\d+\b` (Zephyr Scale convention, e.g. `ABC-T42`).
- Cycle ids: regex `(?:cycle[:#=/-]?)\s*([A-Z0-9_-]{4,})` or links containing `/cycles/` / `testPlayer/`.
- Labels: `zephyr-cycle:<id>`, `zephyr-tc:<key>`.
- Issue links of type `Tests` / `is tested by` → use the linked key.
- Remote links whose URL host matches `*zephyrscale*` or path contains `/zephyr/`.
- Output format inside the fetcher's `zephyr:` line:
  - test case keys → `ABC-T1, ABC-T2`
  - cycle only → `cycle:<id>`
  - both → `ABC-T1, ABC-T2; cycle:<id>`
  - nothing found → `none`

### Figma
- Any URL matching `figma\.com/(file|design|proto)/[^\s)]+`. Keep the full URL.

### Confluence
- Any URL matching `\.atlassian\.net/wiki/` or `confluence[^\s)]*`.

## Field Map (common)
- Acceptance Criteria: usually `customfield_10000`+ — search description if missing.
- Description may be ADF (Atlassian Document Format), not Markdown — flatten before regex scanning.

## Required Config (see [.env](../../../.env))
- `JIRA_BASE_URL` — e.g. `https://your-domain.atlassian.net`
- `JIRA_USER_EMAIL` — Atlassian account email
- `JIRA_API_TOKEN` — API token from https://id.atlassian.com/manage-profile/security/api-tokens

## Pitfalls
- Sub-tasks have their own ACs and their own Zephyr links — aggregate only if the parent is the target.
- A Jira key (`ABC-123`) and a Zephyr test case key (`ABC-T123`) look similar — only the `-T<digits>` form is a Zephyr case.
