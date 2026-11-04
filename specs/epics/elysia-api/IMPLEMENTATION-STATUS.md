# elysia-api — implementation status

| Milestone | State | Landed |
|---|---|---|
| EL0 — the spec | ✅ | EL-1 |
| EL1 — Elysia on Workers | ✅ | EL-2 |
| EL2 — a baseline | planned | EL-3 |

## Departures from the upstream template

Everything not listed here is the template as it is upstream at `84878e5`.

| # | What | Why |
|---|---|---|
| 1 | The app lives in `apps/api` of a pnpm workspace (`@app/api`); upstream's `package.json`, `.gitignore`, `tsconfig.json`, `vercel.json`, `bun.lock` and README are replaced by the workspace's; upstream's `dev` (`bun run --watch`) is `wrangler dev`, and `build` (wrangler's dry run), `preview`, `deploy`, `typecheck` and `lint` are added | The workspace's shape (nitro-boilerplate's); Bun is not the runtime here |
| 2 | `elysia: 1.4.30` (upstream `latest`), with `wrangler` and `@cloudflare/workers-types` beside it; `bun-types` dropped | A pinned Elysia (risk EL-B); the deploy tool and the Worker's types |
| 3 | `src/index.ts`: `new Elysia({ adapter: CloudflareAdapter })` (from `elysia/adapter/cloudflare-worker`) and `.compile()` on upstream's app | Elysia's own path to Workers in place of Bun (design §3) |
| 4 | `src/index.ts`: `<title>` and the heading read `SITE_NAME` (`"Elysia on Workers"`; upstream's are "Elysia on Vercel" and "Welcome to Elysia"), HTML-escaped; the logo is Elysia's own (`elysiajs.com/assets/elysia.svg`, upstream Vercel's framework-logo CDN); the "Vercel docs" link is "Cloudflare Workers docs" (developers.cloudflare.com/workers/) and the "Elysia docs" link points at Elysia's Cloudflare Worker page | One rename seam; the page no longer names a platform it does not run on |
| 5 | `GET /health` chained on: `{"status":"ok","name":SITE_NAME,"environment":…}` from `env.ENVIRONMENT` (`cloudflare:workers`) | The deploy's smoke and the bootstrap's probes read it (hono-api's contract) |
| 6 | `src/identity.ts` added: `export const SITE_NAME = "Elysia on Workers"` | The rename's one seam |
| 7 | `tsconfig.json` extends the workspace's Worker config (ES2022 + WebWorker, `@cloudflare/workers-types`, strict) in place of upstream's Bun config | The Worker's types; `cloudflare:workers` is declared there |
| 8 | `wrangler.jsonc` (`<repo>-api`, `main: src/index.ts`, no `nodejs_compat`, no assets, `env.{dev,stage,prod}` vars) and `component.yaml` (`cloudflare-worker-turbo`, a smoke over `/health`) | The Worker's config and the deploy contract replace Vercel's (design §2–3) |
| 9 | `apps/api/LICENSE.md` is vercel/vercel's Apache-2.0 licence, with its copyright line ("Copyright 2017 Vercel, Inc."); upstream's `package.json` names none and the repository ships no NOTICE | The example ships under its repository's licence; the product carries it |
| 10 | Tests: `tests/api/src/app.test.ts` (vitest): the same app module called with `app.handle(new Request(...))` on Node, `cloudflare:workers` stood in for by `tests/api/src/stubs/cloudflare-workers.ts` | The template ships none (its `test` script exits 1) |
