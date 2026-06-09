---
name: jira-connection
description: "Use when an agent needs to establish a Jira connection (discover the Atlassian MCP server or build a REST client). Covers tool discovery, auth, env vars, and base URL — NOT how to read or interpret tickets."
---

# Jira Connection

This skill is about **how to connect to Jira**, not what to do with the data. For reading tickets, see [jira skill](../jira/SKILL.md).

## When to Use
- An agent is about to make its first Jira call in a run.
- The agent needs to know which tool/transport to use or which env vars to read.

## Tool Discovery (in order)
1. **Atlassian MCP server.** Look for a registered tool whose name matches `*atlassian*` or `*jira*` (e.g. `atlassian/searchIssues`, `mcp-atlassian/get_issue`). If found, use it — it handles auth via the MCP server's own configuration.
2. **REST fallback.** If no MCP tool is available, use the Jira Cloud REST API directly: `GET {JIRA_BASE_URL}/rest/api/3/issue/{key}` with Basic auth (`JIRA_USER_EMAIL:JIRA_API_TOKEN`, base64-encoded).
3. **No tool available.** If neither is configured, return `NO_JIRA_TOOL_AVAILABLE: <key>` and stop. Do not fabricate responses.

## Required Config (see [.env](../../../.env))
- `JIRA_BASE_URL` — e.g. `https://your-domain.atlassian.net`
- `JIRA_USER_EMAIL` — Atlassian account email
- `JIRA_API_TOKEN` — API token from https://id.atlassian.com/manage-profile/security/api-tokens

## Auth
- REST: HTTP Basic, header `Authorization: Basic base64(JIRA_USER_EMAIL:JIRA_API_TOKEN)`.
- MCP: handled by the MCP server config (`.vscode/mcp.json` / `.junie/mcp.json`).

## Connection Health Check (optional)
If a call returns `401`/`403`, the token or email is wrong — stop and tell the user to refresh credentials. Do not retry blindly.

## Pitfalls
- The token is NOT a password. Generate it from the Atlassian account page.
- `JIRA_BASE_URL` must include the scheme (`https://`) and must NOT end with a slash.
- Some MCP servers expose multiple tools (search, get, comment). Pick the most specific one for the task; do not search when you already have the key.
