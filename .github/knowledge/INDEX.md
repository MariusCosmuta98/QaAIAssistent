# Knowledge Index

Lookup table for the [context-retriever](../agents/context-retriever.agent.md).
Add one row per retrievable chunk. See [README.md](README.md) for the format.

| id | topic | path | summary |
|---|---|---|---|
| skill-jira | jira, ticket, acceptance-criteria, adf | .github/skills/jira/SKILL.md | How to read & interpret a Jira issue, including ADF flatten and Zephyr extraction. |
| skill-jira-connection | jira, mcp, rest, auth, env | .github/skills/jira-connection/SKILL.md | How to connect to Jira (MCP discovery, REST fallback, auth, env vars). |
| skill-zephyr | zephyr, test-case, steps, cycle | .github/skills/zephyr/SKILL.md | How to read & create Zephyr Scale test cases (pagination, links.issues match). |
| skill-zephyr-connection | zephyr, mcp, rest, auth, env | .github/skills/zephyr-connection/SKILL.md | How to connect to Zephyr Scale (Bearer token, project key, REST base URL). |
| project-context | project, stack, conventions, build, lint | .github/PROJECT_CONTEXT.md | Per-repo facts: stack, layout, QA conventions, build/run commands, repo setup. |
