# elysia-api — decisions

The choices that shape this repository, one line each, with where the full
reasoning lives. Superseded decisions stay listed, marked, so the history
reads in order.

| # | Decision | Why | Source |
|---|---|---|---|
| 1 | The template's Elysia app, its route unchanged but for the platform's name and links, on Elysia's own Cloudflare adapter (`elysia/adapter/cloudflare-worker`) and `.compile()`, in a plain Worker (`cloudflare-worker-turbo`) | Elysia ships a Workers adapter; no Bun, no rewrite into another framework, and the template's file stays the app module | `specs/epics/elysia-api/design.md` |
| 2 | `elysia` pinned to `1.4.30` (the template asks for `latest`) | A floating dependency would change the Worker under a build unannounced; an upgrade is a deliberate pull request | design.md §3, risk EL-B |
| 3 | `wrangler.jsonc` is committed, `main: src/index.ts`, and the package's build is wrangler's dry run | One config a deploy reads; wrangler bundles the app module itself, so there is no framework build to order before the deploy | `apps/api/wrangler.jsonc` |
| 4 | No assets and no `nodejs_compat` | The template serves its page from code and links its logo; Elysia's Cloudflare adapter is web-standard and needs no Node.js API | design.md |
| 5 | `GET /health` chained onto the template's app, reading `env` from `cloudflare:workers`, and one name seam (`apps/api/src/identity.ts`) the welcome page reads | The deploy smoke and the bootstrap's probes read `/health`; the rename writes the name in one place | IMPLEMENTATION-STATUS |
| 6 | The tests import the same app module and call it with `app.handle(new Request(...))` on Node, `cloudflare:workers` stood in for by a stub `env` | The module that ships is the module tested (the adapter and `.compile()` included); the e2e serves the bundled Worker under workerd | `tests/api/src/app.test.ts` |
| 7 | One landing (`01-product`) then the record (`02-docs`) | A landing has a fixed cost; the plan, not the phases, carries deploy order (cirrus-baseline since baseline-v22) | `repo-blueprint.yaml` |
| 8 | `ci.yml` plans nothing while `intent.yaml` names `ws_CHANGE_ME` | A bootstrap cannot set a repository variable | hono-api's design |
| 9 | stack-tectonic 0.19.0 | The family's one pin (risk EL-F) | risks-and-open-questions.md |
| 10 | `subdomain` is read from the Cloudflare connection, never asked; the console asks for no domain | It belongs to the account; a domain is an owner's step after the bootstrap | `blueprint.yaml` |
