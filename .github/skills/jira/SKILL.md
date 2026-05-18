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
2. Fetch the issue with fields: `summary,status,issuetype,description,customfield_acceptance,issuelinks,attachment,subtasks`.
3. Parse description for embedded Figma/Confluence URLs and Zephyr cycle ids.
4. Return only the [jira-fetcher agent](../../agents/jira-fetcher.agent.md) output format.

## Field Map (common)
- Acceptance Criteria: usually `customfield_10000`+ — search description if missing.
- Zephyr cycle: link of type "Tests" or label `zephyr-cycle:<id>`.
- Figma: any `figma.com/file/...` or `figma.com/design/...` URL in description/comments.

## Required Config (see [.env](../../.env))
- `JIRA_BASE_URL` — e.g. `https://your-domain.atlassian.net`
- `JIRA_USER_EMAIL` — Atlassian account email
- `JIRA_API_TOKEN` — API token from https://id.atlassian.com/manage-profile/security/api-tokens

## Pitfalls
- Sub-tasks have their own ACs — check `subtasks` and aggregate only if the parent is the target.
- Description may be ADF (Atlassian Document Format), not Markdown — flatten before extracting links.
