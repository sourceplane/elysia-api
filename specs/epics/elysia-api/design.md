# elysia-api — design

## 1. The starter

vercel/vercel `examples/elysia` at `84878e5` (the example last changed in
`b998cf7`) is an Elysia project of six files: `src/index.ts` (one
`new Elysia().get("/", …)` returning an inline HTML welcome page as a
`Response` with `Content-Type: text/html` — a title, a heading, a tagline,
two links and Elysia's logo hot-linked from Vercel's framework-logo CDN —
exported as the module's default), a `package.json` whose one dependency is
`elysia: latest` (1.4.30 on the day of the port) beside `bun-types`, a
`tsconfig.json` with Bun's types, a `vercel.json` asking for Bun
(`bunVersion: 1.x`), a `.gitignore` and a README. Vercel runs it on Bun,
detecting `src/index.ts` with zero configuration.

## 2. How each Vercel piece maps

| Vercel | Here |
|---|---|
| Bun on Vercel (`vercel.json` `bunVersion: 1.x`, `bun-types`) | workerd: `new Elysia({ adapter: CloudflareAdapter })` from `elysia/adapter/cloudflare-worker`, and `.compile()` at module scope; the default export is what the runtime calls |
| Zero-config detection of `src/index.ts` | `apps/api/wrangler.jsonc`, committed: the Worker's name, compatibility date, `src/index.ts` as `main` (wrangler bundles it), per-environment `ENVIRONMENT` vars |
| `elysia: latest` | `elysia: 1.4.30`, pinned |
| The page's title "Elysia on Vercel", its Vercel-hosted logo and its Vercel docs link | the site's name (`src/identity.ts`), Elysia's own logo (`elysiajs.com/assets/elysia.svg`), the Workers docs and Elysia's Cloudflare Worker page |
| Deploy on push | `apps/api/component.yaml` (`cloudflare-worker-turbo`): build, dry run on pull requests; build, deploy and a smoke over `/health` on merge |

## 3. What the spike proved

In a scratch copy, before any of the repository: elysia 1.4.30 (npm's
`latest` on 2026-11-05) with `new Elysia({ adapter: CloudflareAdapter })`,
the template's route, a `/health` route and `.compile()`, as the default
export of a wrangler `main`, `compatibility_date` 2026-10-01 and no
`nodejs_compat`. Under `wrangler dev`, `GET /` answers the template's page
(200, `text/html`), `HEAD /` answers 200 with no body (Elysia answers HEAD for
a GET route), `/health` reading `env.ENVIRONMENT` from `cloudflare:workers`
answers that environment's var (`--env prod` → `prod`), an unknown path and an
unknown method answer Elysia's own 404 (`NOT_FOUND`, plain text), and a probe
route with a `t.Object` body schema answers 200 to a valid body and 422 to a
bad one — TypeBox's compiled validators run on workerd. The dry run is
886 KiB, 162 KiB gzipped, with `ENVIRONMENT` its only binding.

Three choices fell out of it:

- **`.compile()` at module scope.** A Worker may generate code while it starts
  (global scope) but not while it answers a request. Elysia's ahead-of-time
  compile, which runs on Workers since 1.4.7, builds every handler once when
  the module loads, so no request ever asks the runtime for `new Function`.
- **The template's file stays the app.** `src/index.ts` is wrangler's `main`:
  the adapter and `.compile()` go on the template's own `new Elysia()`, and
  the bindings come from `cloudflare:workers` rather than a handler argument,
  so there is no separate entry file and nothing to keep in step with it.
- **The pin.** `latest` floats, and the adapter is young. A pinned 1.4.30
  makes an upgrade a pull request the e2e tier checks.

## 4. The repository

nitro-boilerplate's, which is express-api's, which is hono-api's: a pnpm
workspace (`apps/api`, `tests/api`, `tooling/*`), turbo, the shared
TypeScript and ESLint configuration, `intent.yaml` (dev verify-only, stage,
prod after stage, the placeholder workspace `ws_CHANGE_ME`), `ci.yml` byte
for byte turso-starter's as hono-api carries it, and the baseline's machinery
(`blueprint.yaml`, `repo-blueprint.yaml`, `.baseline/`, `tooling/rebrand`,
`tooling/bootstrap`, `hooks/`, `testing/`). The app's build is hono-api's:
wrangler's own dry run with `--outdir dist`, so turbo caches the bundle and the
lane's typecheck and dry run follow it.

Tests import the same app module the Worker runs — the adapter and
`.compile()` included — and call it with `app.handle(new Request(...))` on
Node. The one thing Node does not have, `cloudflare:workers`, is stood in for
by a module exporting an `env` the tests set (a vitest alias, and a
`paths` entry for the type check). The end-to-end tier serves the bundle under
workerd with `wrangler dev --env stage`.

## 5. Alternatives rejected

- **Keeping Bun.** Workers run workerd; Bun is not a runtime Orun deploys.
  Elysia's Cloudflare adapter is its own supported path to Workers.
- **A separate `worker.ts` entry passing the bindings in.** Elysia's
  Cloudflare adapter hands a route no Worker `env`; `cloudflare:workers`
  gives one module-wide, so the template's file can stay the entry.
- **Rewriting the route in another framework (hono-api's shape).** The
  starter is about Elysia: its chained routes, its validators and its
  adapter are the point.
- **`nodejs_compat`.** Nothing in Elysia's Cloudflare adapter or the template
  needs a Node.js API; the flag would only add to the Worker.
