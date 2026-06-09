---
name: jira
description: "Use when reading or interpreting Jira issues (ticket key like ABC-123). Covers which fields to request, how to parse them, and how to auto-extract Zephyr references. For connection setup (MCP/REST/auth/env) see the jira-connection skill."
---

# Jira (Read)

This skill covers **how to read and interpret a Jira ticket**. For connection setup (MCP discovery, REST fallback, auth, env vars) see [jira-connection skill](../jira-connection/SKILL.md).

## When to Use
- A ticket key is mentioned (e.g. `ABC-123`).
- The agent needs acceptance criteria, status, or links to Zephyr.

## Procedure
1. Establish a connection per [jira-connection skill](../jira-connection/SKILL.md).
2. Resolve ticket key (uppercase, e.g. `abc-123` → `ABC-123`).
3. Fetch the issue **requesting only needed fields**: `GET /rest/api/3/issue/{key}?fields=summary,status,issuetype,description,comment,issuelinks&expand=renderedFields`. Omit `attachment`, `subtasks`, `changelog`, `parent`, and epic-link custom fields — they bloat the response and are not used.
4. Run the **Zephyr auto-extraction** below over every text field.
5. Return only the [jira-fetcher agent](../../agents/jira-fetcher.agent.md) output format. Drop all fields not in the output template.

> **Note:** Epic lookup and sibling-ticket discovery have been removed. Do not call the epic endpoint or run JQL searches for siblings.

## Auto-Extract Linked Resources
Scan `description`, `comment.comments[*].body`, `issuelinks[*]`, `remotelinks`, and any custom field text for:

### Zephyr
Pull **all** matches and de-duplicate. Pass them to the orchestrator so `zephyr-fetcher` can query directly (saves a round-trip search by Jira key).

#### Step A — Scan Jira fields
- Test case keys: regex `\b[A-Z][A-Z0-9_]+-T\d+\b` (Zephyr Scale convention, e.g. `ABC-T42`).
- Cycle ids: regex `(?:cycle[:#=/-]?)\s*([A-Z0-9_-]{4,})` or links containing `/cycles/` / `testPlayer/`.
- Labels: `zephyr-cycle:<id>`, `zephyr-tc:<key>`.
- Issue links of type `Tests` / `is tested by` → use the linked key.
- Remote links whose URL host matches `*zephyrscale*` or path contains `/zephyr/`.

#### Step B — Zephyr-side fallback (if Step A found nothing)
Zephyr Scale stores coverage links **on its own side** — they do NOT appear in any Jira REST API response (not in `issuelinks`, `remotelinks`, labels, or description). When Step A yields no Zephyr references, query the Zephyr Scale API directly:

1. Get the Jira issue numeric id from the fetched issue (`id` field in the Jira response, e.g. `1491225`).
2. Paginate through `GET {ZEPHYR_BASE_URL}/testcases?projectKey={PROJECT_KEY}&startAt={n}&maxResults=50`.
   - Use `ZEPHYR_TOKEN` (Bearer) and `ZEPHYR_PROJECT_KEY` from env (fall back to the Jira project key).
   - **Page size 50** (not 200) — smaller pages mean less wasted context if the match is found early.
3. For each test case, check `links.issues[*].issueId` — if it matches the Jira issue numeric id, that test case covers this ticket.
4. Collect matching test case keys (e.g. `QE-T1403`).
5. **Early termination**: stop as soon as matches are found and the current page has been fully checked. **Hard cap**: stop after **5 pages** (250 cases) regardless — if no match, report `none`.

> **Why this is needed:** Zephyr Scale's "Coverage" link type is stored only in Zephyr's database and surfaced only through the Zephyr API. The Jira UI shows it via the Zephyr Scale plugin panel, but the Jira REST API has no visibility into it.

#### Output format
- test case keys → `ABC-T1, ABC-T2`
- cycle only → `cycle:<id>`
- both → `ABC-T1, ABC-T2; cycle:<id>`
- nothing found → `none`

## Field Map (common)
- Acceptance Criteria: usually `customfield_10000`+ — search description if missing.
- Description may be ADF (Atlassian Document Format), not Markdown — flatten before regex scanning.

## Pitfalls
- **Zephyr coverage links are invisible from Jira.** The Jira UI shows them via the Zephyr Scale plugin panel, but the Jira REST API returns nothing — no issue links, no remote links, no custom fields. Always run the Zephyr-side fallback (Step B above) when Jira fields yield no Zephyr references.
- Sub-tasks have their own ACs and their own Zephyr links — aggregate only if the parent is the target.
- A Jira key (`ABC-123`) and a Zephyr test case key (`ABC-T123`) look similar — only the `-T<digits>` form is a Zephyr case.
