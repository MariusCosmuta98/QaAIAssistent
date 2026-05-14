---
name: confluence
description: "Use when QA documentation, test plans, or specs are stored in Confluence and need to be read or updated."
---

# Confluence

## When to Use
- Jira ticket links to a Confluence page.
- A test plan / QA spec / runbook must be read or updated.

## Tools
Prefer Atlassian MCP server (`atlassian/*`). Fallback: REST `GET /wiki/rest/api/content/{id}?expand=body.storage`.

## Procedure
### Read a page
1. Resolve page id from URL (`.../pages/<id>/...`) or via `cql=title="<title>" AND space=<KEY>`.
2. Fetch with `expand=body.storage,version`.
3. Convert storage XHTML to plain Markdown before returning. Strip macros.

### Append QA notes
1. Get current `version.number`.
2. `PUT /content/{id}` with incremented version and merged body. Never overwrite blindly.

## Pitfalls
- Confluence storage format is XHTML, not Markdown.
- Always pass the new `version.number = current + 1` or the update is rejected.
- Respect space restrictions — fail fast if 403.
