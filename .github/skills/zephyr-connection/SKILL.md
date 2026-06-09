---
name: zephyr-connection
description: "Use when an agent needs to establish a Zephyr Scale connection (discover the Zephyr MCP server or build a REST client). Covers tool discovery, auth, env vars, and base URL — NOT how to read or create test cases."
---

# Zephyr (Scale) Connection

This skill is about **how to connect to Zephyr Scale**, not what to do with the data. For reading/creating test cases, see [zephyr skill](../zephyr/SKILL.md).

## When to Use
- An agent is about to make its first Zephyr call in a run.
- The agent needs to know which tool/transport to use or which env vars to read.

## Tool Discovery (in order)
1. **Zephyr MCP server.** Look for a registered tool whose name matches `*zephyr*` (e.g. `zephyr/getTestCase`, `zephyr/searchTestCases`). If found, use it — it handles auth via the MCP server's own configuration.
2. **REST fallback.** If no MCP tool is available, use the Zephyr Scale REST API directly: `{ZEPHYR_BASE_URL}/...` with `Authorization: Bearer {ZEPHYR_TOKEN}`.
3. **No tool available.** If neither is configured, return `NO_ZEPHYR_TOOL_AVAILABLE: <key-or-cycle>` and stop. Do not fabricate responses.

## Required Config (see [.env](../../../.env))
- `ZEPHYR_BASE_URL` — defaults to `https://api.zephyrscale.smartbear.com/v2`
- `ZEPHYR_TOKEN` — API token (Bearer)
- `ZEPHYR_PROJECT_KEY` — Zephyr project key (may differ from Jira project key)

## Auth
- REST: `Authorization: Bearer {ZEPHYR_TOKEN}`.
- MCP: handled by the MCP server config (`.vscode/mcp.json` / `.junie/mcp.json`).

## Connection Health Check (optional)
If a call returns `401`/`403`, the token is wrong or revoked — stop and tell the user. Do not retry blindly.

## Pitfalls
- `ZEPHYR_PROJECT_KEY` is often equal to the Jira project key but not always. When in doubt, prefer the env var.
- The default `ZEPHYR_BASE_URL` is the Cloud endpoint. For Zephyr Scale Server/Data Center, the base URL and auth differ — confirm with the user before assuming Cloud.
- Most Zephyr endpoints paginate; pagination itself is a reading concern (see the zephyr skill), not a connection concern.
