# elysia-api

Vercel's [Elysia on Vercel](https://github.com/vercel/vercel/tree/main/examples/elysia)
template — an [Elysia](https://elysiajs.com/) server whose one route, `GET /`,
answers with the template's welcome page — running as one **Cloudflare
Worker**, served by Elysia's own Cloudflare adapter (in place of Vercel's Bun
runtime) and bundled by wrangler, and packaged as an **Orun baseline**: one command
takes a new app from nothing to live on stage and prod, in its own
repository, under its own name — in ONE landing.

The template's code is the code. What the port adds is Elysia's Cloudflare
adapter and `.compile()` on the template's app, the Worker's config, the
monorepo wrapper, a test package, the deploy contract and a `/health` route
the deploy smoke-tests; what it swaps is the platform's name and links in the
page, Bun for workerd, and `elysia: latest` for a pinned `1.4.30`. What the baseline adds is everything needed to make a
copy of it somebody else's — built the way
[cirrus-baseline](https://github.com/sourceplane/cirrus-baseline) is. See
[`specs/epics/elysia-api`](specs/epics/elysia-api/README.md)
for the design, the milestones and the as-built status.

The example is Apache-2.0-licensed (vercel/vercel); its licence ships with the
app as `apps/api/LICENSE.md`.

## Instantiating a product

```bash
git clone --depth 1 --branch <baseline-tag> https://github.com/sourceplane/elysia-api
cd elysia-api
orun new --blueprint repo-blueprint.yaml --out ~/apps/acme-api --run-hooks --resume \
  --set reponame=acme-api --set productname="Acme API" \
  --set githuborg=acme-inc --set subdomain=<workers-dev-subdomain>
```

Or, once the platform's registry lists it, from the Orun console (Baselines →
`elysia-api` → Build). Two phases — `01-product` (the repository, its CI and
the whole app in one pull request, deployed to stage and prod by one
convergence, smoke-tested) and `02-docs` (the live deployment recorded) —
each landing one pull request in the new repository.
**[BOOTSTRAP.md](BOOTSTRAP.md)** is the operator guide;
[docs/phases/](docs/phases/README.md) has a page per phase.

The new repository receives the **app only** — the Worker, its tests, its CI,
its tooling, a README of its own and its operating contract — renamed
throughout to the app's name, repository and Cloudflare Worker. None of this
repository's machinery ships, and nothing that ships names it.

| What makes it a baseline | Where |
|---|---|
| The console card (inputs, secrets, programme) | `blueprint.yaml` |
| The build document (modules, phases, hooks) | `repo-blueprint.yaml` |
| This tree's identity, and the bootstrap's task contracts | `.baseline/` |
| The rename engine | `tooling/rebrand/rebrand.mjs` |
| The bootstrap's helpers (deferrals, redeploy marker, work-tree strip) | `tooling/bootstrap/` |
| The deployment-record renderer | `hooks/render-deployment-docs.sh` |
| The app's README and context pack | `.baseline/product/README.md`, `ai/context/` |
| The baseline's tests: contract, leak, offline bootstrap, end to end | `testing/`, `.github/workflows/baseline.yml` |
| Cutting a `baseline-vN` tag | `.github/workflows/tag.yml` |

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
| `baseline.yml` | the baseline's own tiers: contract tests, the blueprint read by the pinned orun, the leak gate, the offline bootstrap, the app bundled by wrangler, dry-deployed, served and read route by route | no |
| `tag.yml` | cuts a `baseline-vN` tag (manual) | no |
