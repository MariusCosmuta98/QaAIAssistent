---
name: jira-connection
description: "How to connect to Jira from the JiraAgent reader — MCP only. Self-contained; does not reference .github/."
---

# Jira Connection (JiraAgent)

Self-contained connection guide for the [jira-reader agent](../agents/jira-reader.agent.md). Covers how to reach Jira via MCP and which fields to pull.

## Tool discovery

1. **Atlassian MCP server.** Check your available tools list for any MCP tool whose name contains `atlassian` or `jira` (e.g. `getJiraIssue`, `mcp-atlassian/get_issue`, `jira_get_issue`, `com-atlassian-atlassian-mcp-server-getJiraIssue`). Prefer the most specific "get by key" tool. If found, use it — auth is handled by the MCP server itself.
2. **No MCP tool found.** Return `NO_JIRA_MCP_AVAILABLE: <KEY>` and stop. **Never fabricate a response.**


## Fields to request

Only what the reader's output block needs. Do **not** fetch `attachment`, `subtasks`, `changelog`, or epic-link custom fields — they bloat the response.

Required fields: `summary`, `status`, `issuetype`, `description`, `comment`, `priority`, `labels`, `parent`, `customfield_10050` (Acceptance Criteria).

- `priority` — simple object; read `.name` (e.g. `"High"`).
- `labels` — string array; join with `, `. Empty array → write `<none>`.
- `parent` — present when the issue is a child (story/sub-task). Read `.key` and `.fields.issuetype.name`; only surface the key in the `Epic:` line when the parent's issue type is `Epic`. Omit the line entirely otherwise.


## Parsing rules

- The Jira description is **Atlassian Document Format (ADF)**, not Markdown. Flatten it (walk the node tree, concatenate `text` nodes with newlines between blocks) before extracting acceptance criteria.
- Acceptance criteria detection (in order of preference):
    1. **Custom field first.** If `customfield_10050` (or any field whose name matches `acceptance criteria` case-insensitively via `expand=names`) is non-empty, flatten its ADF/HTML value and extract every bullet/numbered/checklist item. This takes precedence over anything in the description.
    2. A heading whose text case-insensitively matches `acceptance criteria`, `AC`, or `given/when/then` → take all bullet/numbered items that follow until the next heading.
    3. Any bullet list where the first item starts with `Given `, `When `, or `Then `.
    4. A checklist (`[ ]` / `[x]`) inside the description.
    5. If none found, return `- <none>`.
- Comments output limit: cap to 3 comments and 300 characters per comment; truncate with `...`.
- Goal output limit: cap to 2 lines and 220 characters total; truncate with `...`.
