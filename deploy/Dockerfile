# openPlan3D fork: SvelteKit + adapter-node. Not built or deployed yet (stage D is files only).
# glibc base (Debian slim), not Alpine, so native modules such as @surrealdb/node-native can load later.
# Build context is the repo root: docker build -f deploy/Dockerfile .
# Node 24 matches .nvmrc.

FROM node:24-bookworm-slim AS build
WORKDIR /app
# vite.config.ts refuses to build unless NODE_ENV=production; the compiler lives in devDependencies, so install them
# explicitly (the same pair apphosting.yaml sets for Firebase builds).
ENV NODE_ENV=production \
    NPM_CONFIG_INCLUDE=dev
COPY package.json package-lock.json .npmrc ./
RUN npm ci
COPY . .
RUN npm run build
# Runtime copy keeps production dependencies only. The browser libraries (three, firebase, ...) sit in
# devDependencies because adapter-node bundles them into build/.
RUN npm prune --omit=dev

FROM node:24-bookworm-slim AS runtime
WORKDIR /app
ENV NODE_ENV=production \
    PORT=3000 \
    HOST=0.0.0.0
COPY --from=build --chown=node:node /app/build ./build
COPY --from=build --chown=node:node /app/node_modules ./node_modules
COPY --from=build --chown=node:node /app/package.json ./package.json
# Data directory the compose file bind-mounts over (mirror folder for the agent bridge).
RUN mkdir -p /data && chown node:node /data
USER node
EXPOSE 3000
CMD ["node", "build"]
