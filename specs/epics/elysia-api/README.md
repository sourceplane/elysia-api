# elysia-api — Vercel's Elysia template on Cloudflare Workers

**The template is an Elysia server with one route that answers with a
welcome page, run on Bun by Vercel; Elysia ships an adapter for Workers, so
the port keeps the app as it is: Elysia's own Cloudflare adapter and
`.compile()` on the template's app, bundled by wrangler into one Cloudflare
Worker, a `/health` route beside it, deployed by Orun to stage and prod on
every merge — and the baseline that makes a copy of it somebody else's app
in one landing.**

| | |
|---|---|
| Status | ✅ Shipped (`baseline-v1`) — proven offline in Baseline CI; not yet live (risk EL-A) |
| Upstream | [vercel/vercel `examples/elysia`](https://github.com/vercel/vercel/tree/main/examples/elysia) (Apache-2.0), at `84878e5` (2026-10-27; the example last changed in `b998cf7`) |
| Reference | [nitro-boilerplate](https://github.com/sourceplane/nitro-boilerplate) — the same plain Worker, scaffolding, tests and bootstrap; [hono-api](https://github.com/sourceplane/hono-api) — the same wrangler-bundled app module |
| Key | `EL` |

## Read in this order

1. [design.md](design.md) — the starter, how each Vercel piece maps to
   Cloudflare, what the spike proved, the repository and the build, and the
   alternatives rejected.
2. [implementation-plan.md](implementation-plan.md) — the milestones and
   what each one lands.
3. [risks-and-open-questions.md](risks-and-open-questions.md) — what is not
   proven, and what was accepted.
4. [IMPLEMENTATION-STATUS.md](IMPLEMENTATION-STATUS.md) — what shipped, and
   every departure from the upstream starter.

## Milestones

| Milestone | What it lands | Task |
|---|---|---|
| EL0 — the spec | the repository, its intent, CI, tooling and spec index; this doc set | EL-1 |
| EL1 — Elysia on Workers | the template in `apps/api`, on Elysia's Cloudflare adapter, its tests, the README | EL-2 |
| EL2 — a baseline | the console card, the build document, the context pack and the baseline's tests | EL-3 |
