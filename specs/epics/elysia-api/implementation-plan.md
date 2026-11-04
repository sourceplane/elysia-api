# elysia-api — implementation plan

Three milestones, one task each, each landed as one pull request.

## EL0 — the spec (EL-1)

The repository is born with nitro-boilerplate's frame — `intent.yaml` (dev
verify-only, stage, prod after stage; the placeholder workspace
`ws_CHANGE_ME`, so CI plans nothing until it is linked), `ci.yml` byte for
byte turso-starter's (as hono-api carries it), the shared TypeScript and
ESLint configuration, the wrangler renderer, the pnpm workspace and turbo
pipeline, and the spec index — and this doc set and the three task
contracts land on it.

## EL1 — Elysia on Workers (EL-2)

- `apps/api/src/index.ts` is upstream's, on `new Elysia({ adapter:
  CloudflareAdapter })` with `.compile()`, its title and heading read from
  `SITE_NAME` (HTML-escaped), its Vercel logo and link now Elysia's and
  Cloudflare's, and `/health` chained on, reading `env` from
  `cloudflare:workers`; `apps/api/src/identity.ts` holds `SITE_NAME`;
  `apps/api/LICENSE.md` is vercel/vercel's Apache-2.0 licence.
- `elysia` pinned to 1.4.30 (design §3); `apps/api/wrangler.jsonc`
  (`src/index.ts` as `main`, no `nodejs_compat`, no assets, per-environment
  vars) and `component.yaml` (`cloudflare-worker-turbo`) with a smoke over
  `/health`.
- `tests/api`: vitest importing the same module and calling `/`, `HEAD /`,
  `POST /`, `/health` with and without an environment, and a 404, through
  `app.handle(new Request(...))`, `cloudflare:workers` stood in for.
- The README: run, test, deploy; how each Vercel piece maps.

Done when typecheck, lint, test and build are green, and `wrangler dev`
answers every route.

## EL2 — a baseline (EL-3)

nitro-boilerplate's machinery on this tree: the card, the build document (one
landing, `01-product`, then the record, `02-docs`), `.baseline/` (identity,
the product's README, task contracts), the rename engine (the `SITE_NAME`
seam alone), the bootstrap helpers, the deployment-record renderer (probing
`/health` and `/`), the phase pages, the context pack, and the baseline's
tests — with the end-to-end tier replaying the stage lane in a staged job
tree (no build output in it), checking the dry run is one bundle with no
assets, and reading every route from the staged Worker under `wrangler
dev`. Done when every Baseline CI tier is green; tagged `baseline-v1`.
