# JiraAgent

Standalone Jira → Markdown test-plan generator. Fully self-contained — does not depend on `.github/` in this repo.

## What it does

1. Reads a Jira ticket (cheap model, single API call, no reasoning).
2. Turns each acceptance criterion into a Markdown test case.
3. Writes everything to `tests/plans/<TICKET-KEY>/` at the workspace root.

## Layout

```
JiraAgent/
├── README.md                      ← you are here
├── agents/
│   ├── jira-reader.agent.md            ← cheap read-only Jira agent (GPT-4.1 mini)
│   └── test-plan-generator.agent.md    ← consumes the reader, writes the plan
├── skills/
│   └── jira-connection.skill.md        ← how to call Jira (MCP + REST fallback)
├── prompts/
│   └── plan-ticket.prompt.md           ← /plan-ticket <KEY> slash command
└── examples/
    └── EXAMPLE-1/                       ← what a generated plan looks like
        ├── test-plan.md
        ├── TC-01-user-can-log-in-with-valid-credentials.md
        └── TC-02-login-rejects-invalid-password.md
```

## How to use

### Activate the agents / prompts in VS Code Copilot Chat

VS Code Copilot Chat discovers agent and prompt files under `.github/agents/` and `.github/prompts/` by default. Because this folder is standalone, pick one of:

**Option A — symlink (recommended, keeps this folder as source of truth):**

```bash
mkdir -p .github/agents .github/prompts
ln -s ../../JiraAgent/agents/jira-reader.agent.md          .github/agents/jira-reader.agent.md
ln -s ../../JiraAgent/agents/test-plan-generator.agent.md  .github/agents/test-plan-generator.agent.md
ln -s ../../JiraAgent/prompts/plan-ticket.prompt.md        .github/prompts/plan-ticket.prompt.md
```

**Option B — configure additional locations** in `.vscode/settings.json`:

```jsonc
{
  "chat.promptFilesLocations": { "JiraAgent/prompts": true },
  "chat.instructionsFilesLocations": { "JiraAgent/agents": true }
}
```

(Setting names vary between VS Code versions — check your version's docs if they don't take effect.)

**Option C — copy** the files into `.github/agents/` and `.github/prompts/` (loses the "standalone" property but zero config).

### Configure Jira credentials

Add to `.env` at the workspace root:

```
JIRA_BASE_URL=https://your-domain.atlassian.net
JIRA_USER_EMAIL=you@example.com
JIRA_API_TOKEN=<token from https://id.atlassian.com/manage-profile/security/api-tokens>
```

If you also have the Atlassian MCP server registered in `.vscode/mcp.json`, the reader will prefer that over REST.

### Run

In Copilot Chat:

```
/plan-ticket ABC-123
```

Result: `tests/plans/ABC-123/test-plan.md` (index) + `TC-01-*.md`, `TC-02-*.md`, … (one per acceptance criterion). See `examples/EXAMPLE-1/` for the exact shape.

## Contract

- **Input:** a Jira ticket key (regex `[A-Z][A-Z0-9_]+-\d+`).
- **Output:** `tests/plans/<KEY>/` containing `test-plan.md` + N × `TC-NN-<slug>.md`.
- **Non-goals:** does not run tests, does not push to Zephyr, does not modify the codebase.

## Anti-hallucination

Every field written into the plan MUST come from a real Jira API response. If the reader cannot reach Jira, it stops with `NO_JIRA_TOOL_AVAILABLE` — it does not invent ticket contents. If an acceptance criterion is too vague to test, the generator emits a `NEEDS-CLARIFICATION` test case rather than making up thresholds or steps.
