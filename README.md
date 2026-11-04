# elysia-api

Vercel's [Elysia on Vercel](https://github.com/vercel/vercel/tree/main/examples/elysia)
template — an [Elysia](https://elysiajs.com/) server whose one route, `GET /`,
answers with the template's welcome page — running as one **Cloudflare
Worker**, served by Elysia's own Cloudflare adapter (in place of Vercel's Bun
runtime) and bundled by wrangler, deployed by
[Orun](https://github.com/sourceplane/orun) to stage and prod on every merge.
Packaging it as an Orun baseline is the next milestone (EL2).

The template's code is the code. What the port adds is Elysia's Cloudflare
adapter and `.compile()` on the template's app, the Worker's config, the
monorepo wrapper, a test package, the deploy contract and a `/health` route
the deploy smoke-tests; what it swaps is the platform's name and links in the
page, Bun for workerd, and `elysia: latest` for a pinned `1.4.30`. See
[`specs/epics/elysia-api`](specs/epics/elysia-api/README.md)
for the design, the milestones and the as-built status.

The example is Apache-2.0-licensed (vercel/vercel); its licence ships with the
app as `apps/api/LICENSE.md`.

## What is here (the product)

| Surface | Where |
|---|---|
| The welcome page, `GET /` — the template's route, named by the site's name, and `GET /health` — the site's name and the environment it runs as, on one Elysia app compiled by its Cloudflare adapter | `apps/api/src/index.ts` |
| The site's name (the rename's seam) | `apps/api/src/identity.ts` |
| Worker config (name, compatibility, `src/index.ts` as the entry, per-environment vars) | `apps/api/wrangler.jsonc` |
| Deploy contract | `apps/api/component.yaml` (`cloudflare-worker-turbo`) |
| Tests | `tests/api` (vitest: imports the same app module and calls every route with `app.handle(new Request(...))` on Node) |
| The example's licence (Apache-2.0) | `apps/api/LICENSE.md` |

## Getting started

```bash
pnpm install
pnpm typecheck
pnpm lint
pnpm test
pnpm build            # wrangler's dry run → apps/api/dist (the Worker's one bundle)
```

Run the Worker under workerd:

```bash
pnpm --filter @app/api dev            # wrangler dev, http://localhost:8787
curl http://localhost:8787/           # the welcome page
curl http://localhost:8787/health     # {"status":"ok","name":"Elysia on Workers","environment":"local"}
```

Add a route by chaining it onto the app in `apps/api/src/index.ts`
(`.get("/hello", () => "hi")`, before `.compile()`); validate a body with
Elysia's `t` (`.post("/echo", ({ body }) => body, { body: t.Object({ name: t.String() }) })`
answers 422 to a body that does not match). Read the Worker's bindings with
`import { env } from "cloudflare:workers"`, as `/health` does.

## How it maps to Vercel

| Vercel | Here |
|---|---|
| Vercel runs the app on Bun (`vercel.json` `bunVersion: 1.x`, `bun-types`) | The same app on workerd: `new Elysia({ adapter: CloudflareAdapter })` from `elysia/adapter/cloudflare-worker`, and `.compile()` once at module scope (a Worker may generate code at startup, not per request); the default export is what the runtime calls |
| Vercel's zero-config detection of `src/index.ts` | `wrangler.jsonc` names `src/index.ts` as the Worker's `main`; wrangler bundles it (Elysia and its TypeBox validator, about 160 KiB gzipped) |
| `elysia: latest` | `elysia: 1.4.30`, pinned: a floating version would change the Worker under a build unannounced |
| The page's title "Elysia on Vercel", its Vercel-hosted logo and its Vercel docs link | the site's name (`src/identity.ts`), Elysia's own logo, and links to the Workers docs and Elysia's Cloudflare Worker page |
| `bun run --watch src/index.ts` · `vc dev` · `vc deploy` | `wrangler dev` · `wrangler deploy --env <stage\|prod>` |
| Deploy on push | Orun deploys `<repo>-api-{stage,prod}` on merge to `main` and smoke-tests `/health` (`apps/api/component.yaml`) |
| "Deploy with Vercel" (clone the template) | `orun new --blueprint repo-blueprint.yaml` — a new repository, branded, deployed |

## CI

| Workflow | What it runs | Ships to an app |
|---|---|---|
| `ci.yml` | the Orun pipeline, and the only workflow an app carries: `orun plan --changed`, then one job per lane. A pull request runs the verify lanes (the Worker bundled by wrangler, type-checked and dry-deployed per environment; the tests' `quick-check` lane installs and checks the package, it does not run them), except the bootstrap's own landings (`orun/BASE-*`), where `plan` alone is the check; a merge deploys stage → prod. It plans nothing while `intent.yaml` names the placeholder workspace `ws_CHANGE_ME`, as this repository does. A manual run (Orunbase's Deploy button, or Run workflow on `main`) plans the environment and components it names under the push-to-main trigger, so they deploy as a merge deploys them | yes |
