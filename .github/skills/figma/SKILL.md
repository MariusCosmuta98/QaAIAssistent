---
name: figma
description: "Use when a Figma URL or node id is referenced and UI structure, copy, states, or flow info is needed for QA."
---

# Figma

## When to Use
- A Jira ticket links to a Figma file.
- The implementer needs component names, copy strings, or interaction states.

## Tools
Prefer the Figma MCP server (`figma/*` or `figma-dev-mode/*`). Fallback: REST `https://api.figma.com/v1/files/{file_key}/nodes?ids=<node-id>` with `X-Figma-Token: $FIGMA_TOKEN`.

## Procedure
1. Parse URL: `figma.com/(file|design)/<fileKey>/...?node-id=<encoded>`. Decode `node-id` (replace `-` with `:`).
2. Fetch the node tree (depth ≤ 3 — deeper trees explode token usage).
3. Walk children and extract:
   - `name` of frames/components
   - `characters` of TEXT nodes (= visible copy)
   - variants / component properties (= states like hover, disabled, error)
   - `prototypeStartNodeID` / interactions (= flow)
4. Return only the [figma-fetcher agent](../../agents/figma-fetcher.agent.md) format.

## Required Config (see [.env](../../../.env))
- `FIGMA_TOKEN` — personal access token from https://www.figma.com/developers/api#access-tokens

## Pitfalls
- Do NOT fetch the full file — always target specific node ids.
- Do NOT export images unless asked; QA usually needs structure + copy, not pixels.
- Variant names encode states — parse them (e.g. `State=Error, Size=Md`).
