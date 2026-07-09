# JiraAgent

Standalone Jira → Markdown test-plan generator. Fully self-contained — does not depend on `.github/` in this repo.

## What it does

1. Reads a Jira ticket via the Atlassian MCP server (Claude Sonnet 4.6, single API call).
2. Turns each acceptance criterion into a Markdown test case.
3. Writes everything to `tests/plans/<TICKET-KEY>/` at the workspace root.

## Layout

```
JiraAgent/
├── README.md                      ← you are here
├── agents/
│   ├── jira-reader.agent.md            ← read-only Jira agent (Claude Sonnet 4.6)
│   └── test-plan-generator.agent.md    ← consumes the reader, writes the plan
├── skills/
│   └── jira-connection.skill.md        ← how to call Jira (MCP only)
└── prompts/
    └── plan-ticket.prompt.md           ← /plan-ticket <KEY> slash command
```

---

## Prerequisites

### 1. Atlassian MCP server (required)

The Jira reader uses the **Atlassian remote MCP server** — a hosted SSE endpoint provided by Atlassian. The reader will not fall back to REST; if the MCP server is unavailable it stops with `NO_JIRA_MCP_AVAILABLE`.

#### Install via VS Code

1. Open the Command Palette (`⌘⇧P` / `Ctrl+Shift+P`) and run **MCP: Add Server**.
2. Choose **HTTP (SSE)** as the transport type.
3. Enter the server URL: `https://mcp.atlassian.com/v1/sse`
4. Give it the name `atlassian`.
5. VS Code writes the entry to `.vscode/mcp.json` automatically.

Alternatively, create or edit `.vscode/mcp.json` manually:

```jsonc
{
  "servers": {
    "atlassian": {
      "type": "sse",
      "url": "https://mcp.atlassian.com/v1/sse",
      "env": {
        "JIRA_BASE_URL": "${env:JIRA_BASE_URL}",
        "JIRA_USER_EMAIL": "${env:JIRA_USER_EMAIL}",
        "JIRA_API_TOKEN": "${env:JIRA_API_TOKEN}"
      }
    }
  }
}
```

> **Tip:** The `${env:…}` references are resolved from your `.env` file at the workspace root (see below). VS Code loads `.env` automatically when the MCP server starts.

#### Verify the server is running

Open the **MCP** pane in the VS Code sidebar (or run **MCP: List Servers**). The `atlassian` entry should show **Connected** and expose tools like `getJiraIssue` / `com.atlassian/atlassian-mcp-server/getJiraIssue`.

### 2. Jira credentials

Create a `.env` file at the workspace root (it is already in `.gitignore`):

```env
JIRA_BASE_URL=https://your-domain.atlassian.net
JIRA_USER_EMAIL=you@example.com
JIRA_API_TOKEN=<token from https://id.atlassian.com/manage-profile/security/api-tokens>
```

To generate an API token: Atlassian account settings → **Security** → **API tokens** → **Create API token**.

---

## Activate the agents / prompts in VS Code Copilot Chat

VS Code Copilot Chat discovers agent and prompt files from `.github/agents/` and `.github/prompts/` by default. The agents in this folder are already wired up under `.github/` in this repo, so **no extra configuration is needed if you are using this repo directly**.

If you copy JiraAgent into another project, pick one of:

**Option A — symlink (keeps this folder as source of truth):**

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

**Option C — copy** the files into `.github/agents/` and `.github/prompts/`.

---

## Run

Once the MCP server is connected and credentials are set, open Copilot Chat and run:

```
/plan-ticket ABC-123
```

Result: `tests/plans/ABC-123/test-plan.md` (index) + `TC-01-*.md`, `TC-02-*.md`, … (one per acceptance criterion).

To regenerate an existing plan, add the word `refresh`:

```
/plan-ticket ABC-123 refresh
```

---

## Contract

- **Input:** a Jira ticket key (regex `[A-Z][A-Z0-9_]+-\d+`).
- **Output:** `tests/plans/<KEY>/` containing `test-plan.md` + N × `TC-NN-<slug>.md`.
- **Non-goals:** does not run tests, does not push to Zephyr, does not modify the codebase.

## Anti-hallucination

Every field written into the plan MUST come from a real Jira API response. If the reader cannot reach the MCP server, it stops with `NO_JIRA_MCP_AVAILABLE` — it never fabricates ticket contents. If an acceptance criterion is too vague to test, the generator emits a `NEEDS-CLARIFICATION` test case rather than inventing thresholds or steps.
