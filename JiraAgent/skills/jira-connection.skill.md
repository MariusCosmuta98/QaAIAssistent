---
name: jira-connection
description: "How to connect to Jira from the JiraAgent reader — MCP discovery first, REST fallback, auth, env vars, and which fields to request. Self-contained; does not reference .github/."
---

# Jira Connection (JiraAgent)

Self-contained connection guide for the [jira-reader agent](../agents/jira-reader.agent.md). Covers how to reach Jira and which fields to pull.

Runtime config note:
- JetBrains/Junie typically reads MCP server configuration from `.junie/mcp.json`.
- VS Code setups typically read MCP server configuration from `.vscode/mcp.json`.


## Tool discovery (in order)

Normalize the ticket key before any lookup: trim whitespace and uppercase (for example `abc-123` -> `ABC-123`).

1. **Atlassian MCP server.** Look for a registered tool whose name matches `*atlassian*` or `*jira*` (e.g. `getJiraIssue`, `mcp-atlassian/get_issue`, `jira_get_issue`). Prefer the most specific "get by key" tool. If found, use it — auth is handled by the MCP server itself.
2. **REST fallback.** If no MCP tool is available, call Jira Cloud REST directly:
   ```
   GET {JIRA_BASE_URL}/rest/api/3/issue/{KEY}?fields=summary,status,issuetype,description,comment&expand=renderedFields
   ```
   with HTTP Basic auth: `Authorization: Basic base64(JIRA_USER_EMAIL:JIRA_API_TOKEN)`.
3. **No tool available.** Return `NO_JIRA_TOOL_AVAILABLE: <KEY>` and stop. **Never fabricate a response.**


## Required env (read from `.env` at workspace root)

| Var | Purpose |
|-----|---------|
| `JIRA_BASE_URL` | e.g. `https://your-domain.atlassian.net` — include scheme, no trailing slash |
| `JIRA_USER_EMAIL` | Atlassian account email |
| `JIRA_API_TOKEN` | Token from https://id.atlassian.com/manage-profile/security/api-tokens (NOT a password) |

The runtime must expose these vars regardless of how they are loaded (`.env`, IDE secrets, CI variables, etc.).


## Fields to request

Only what the reader's output block needs. Do **not** fetch `attachment`, `subtasks`, `changelog`, `parent`, or epic-link custom fields — they bloat the response.

Required fields: `summary`, `status`, `issuetype`, `description`, `comment`.

Timeout and retry policy (REST fallback):
- Timeout: 15s per request.
- Retries: up to 2 retries for `429` and any `5xx` response.
- Backoff: exponential (`1s`, then `2s`) with light jitter (+/- 200ms).
- If retries are exhausted:
    - `429` -> return `JIRA_ERROR: <KEY> 429`
    - `5xx` -> return `JIRA_ERROR: <KEY> <status-code>`
- Do not retry `400`, `401`, `403`, or `404`.


## Parsing rules

- The Jira description is **Atlassian Document Format (ADF)**, not Markdown. Flatten it (walk the node tree, concatenate `text` nodes with newlines between blocks) before extracting acceptance criteria.
- Acceptance criteria detection (in order of preference):
    1. A heading whose text case-insensitively matches `acceptance criteria`, `AC`, or `given/when/then` → take all bullet/numbered items that follow until the next heading.
    2. Any bullet list where the first item starts with `Given `, `When `, or `Then `.
    3. A checklist (`[ ]` / `[x]`) inside the description.
    4. If none found, return `- <none>` so the downstream planner can draft ACs for user validation.
- Comments output limit: cap to 3 comments and 300 characters per comment; truncate with `...`.
- Goal output limit: cap to 2 lines and 220 characters total; truncate with `...`.

