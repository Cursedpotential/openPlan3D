# Fork plan: agent assistance and SurrealDB persistence

Fork of `laanlabs/openPlan3D` at `Cursedpotential/openPlan3D`. Owner decisions, 2026-11-19.
Items marked (planner) were added by the planning session and are not the owner's words.

## Owner decisions

- Standalone repo, not a Propria module.
- Persistence: on-device storage stays; SurrealDB is added as a selectable setting.
- SurrealDB home: a new namespace on the existing `surreal-analysis` server (SurrealDB 4.0.1). An embedded
  SurrealDB engine is also offered as a setting.
- Agents edit plans from a description over MCP, and an in-app chat panel edits the open plan.
- Chat panel agents, selectable: Claude Agent SDK (owner's Claude plan), OpenAI Agents SDK (API key),
  Codex SDK (owner's ChatGPT plan).
- A ChatGPT app (Apps SDK) that shows the plan inline. Claude gets the same MCP tools.
- The MCP server is registered in ContextForge so ChatGPT and Claude share it.
- Everything is installed, built, tested and run on the VPS devbox. Nothing runs on the owner's desktop.

## Rules for every stage

- Additive changes only: new files, new routes, new store backends. Touch upstream files as little as possible so
  merges from `upstream/main` stay cheap. Leave `src/lib/server/agentBridgeGuard.ts` alone.
- Reuse `src/lib/utils/agentOps.ts` and `readProject()` for every edit path. No second copy of the operations.
- Every new route, tool and store method carries a doc comment that is the source of its documentation.
- Credentials stay on the server and out of git.
- No mocks standing in for SurrealDB or a model. Tests that need the database use the real engine.
- `npm install`, `npm run check`, `npm test`, builds and servers run on the devbox only.
- Nothing is applied to the live `surreal-analysis` server, ContextForge or Coolify without the owner's go-ahead.
  Prepare the exact statements or config and report them.

## Stages

| Stage | What | Model | Depends on |
|---|---|---|---|
| A | Server-side project store with a folder backend (today's mirror) and a SurrealDB backend; sync routes; a client setting for on-device only or on-device plus Surreal | Opus | none |
| D | Dockerfile, Coolify compose file, env example, deploy notes | Sonnet | none |
| B | Bearer-token remote MCP route over the store, reusing the bridge tools; MCP Apps viewer (2D plan) with the ChatGPT extras on top | Opus | A |
| C | Chat panel and server agent route with the three selectable agents, each given the plan tools | Opus | A, B |
| E | Deploy on Coolify, create the namespace, register in ContextForge, live checks from Claude and ChatGPT | planner, with owner go-ahead | A to D |

## Stage notes

### A. Store
- (planner) One record per project: the validated project JSON, `updatedAt` and a revision used for
  compare-and-write, mirroring `ProjectConflictError` semantics in `src/lib/services/datastore.ts`.
- (planner) Backend is chosen by connection string, so `wss://…` (server) and an embedded engine
  (`@surrealdb/node` with `@surrealdb/node-native@4.0.1`) share one code path.
- The existing mirror logic (`viaMirror`, `syncTab`, `mirrorSave` in `src/lib/server/agentBridge.ts`) is the sibling
  to generalise.
- Schema lives in a `.surql` file in the repo.

### B. Remote MCP and viewer
- New route, new guard (bearer token from env). Works with no editor tab open, straight against the store.
- `delete_projects` is not offered on the remote route. (planner)
- Viewer follows the MCP Apps standard (`ui://` resource, `_meta.ui.resourceUri`), with `openai/*` keys added.
- ContextForge has MCP Apps off by default and an open bug with federated viewers (IBM/mcp-context-forge#7628).
  The viewer resource carries explicit CSP and sandbox metadata.

### C. Chat panel
- The browser talks to a server route; the server runs the chosen agent with the plan tools attached and streams
  back. Edits apply in the tab as one Undo step.
- Not yet proven: plan sign-in on a headless server for the Claude Agent SDK (`claude setup-token`) and whether the
  Codex TypeScript SDK reads a `codex login --device-auth` sign-in. Prove each on the devbox before building on it.

### E. Deploy
- Coolify app on the VPS, tailnet only. Firebase share and handoff features off. (planner)
- Only the MCP tools are public, through ContextForge's token-gated endpoint. (planner)
