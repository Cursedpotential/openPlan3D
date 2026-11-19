# Deploying the fork

**Nothing in this folder has been built, run or deployed.** It was written by reading the code only. First build and
checks happen on the VPS devbox, never on the desktop.

## Pieces

- `Dockerfile` (build context = repo root): Node 24 (`.nvmrc`), Debian slim (glibc, so `@surrealdb/node-native`
  can be added later), `npm ci` + `npm run build`, runs `node build` as the non-root `node` user.
- `openplan3d.yaml`: Coolify Docker Compose file. One service, port published on a tailnet address only, no public
  router labels, 1 GB memory cap, healthcheck on `/`, host bind mount `/data/openplan3d/mirror` -> `/data/mirror`.
- `.env.example`: every variable, name only.
- `../.dockerignore`: keeps `node_modules`, `.git`, `.claude`, root screenshots out of the build context.

## Steps (not yet done)

1. Create the host directory on the VPS and make it writable by uid 1000: `/data/openplan3d/mirror`.
2. In Coolify create a Docker Compose app from `Cursedpotential/openPlan3D`, compose file
   `deploy/openplan3d.yaml`. Auto-deploy stays off.
3. Set `BIND_IP`, `ORIGIN`, and (once stages A/B land) the `SURREAL_*` and `MCP_BEARER_TOKEN` values in Coolify env.
4. Deploy explicitly, then check `http://<BIND_IP>:8480/` over the tailnet.

## Things to know

- Firebase handoff uploads and assistant shares are off because `HANDOFF_UPLOADS_ENABLED`,
  `ASSISTANT_SHARES_ENABLED` and `HANDOFF_BUCKET` are not passed to the container. `PUBLIC_ENABLE_ANALYTICS=false`
  stops the browser loading the Firebase SDK.
- `BODY_SIZE_LIMIT=68M` is upstream's value.
- The upstream agent bridge answers loopback clients only (`isLocalClient` in `src/lib/server/agentBridge.ts`),
  so even with `OPENPLAN3D_AGENT_BRIDGE=1` it will answer 404 to a request arriving through Docker's port mapping.
  The remote MCP route in stage B is what is meant to serve agents.
- `SURREAL_*` and `MCP_BEARER_TOKEN` are names chosen here; no code reads them yet.
- The build needs `NODE_ENV=production` and dev dependencies installed (`vite.config.ts` enforces it); the
  Dockerfile sets both.
