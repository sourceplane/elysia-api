# elysia-api — risks and open questions

## EL-A — No live bootstrap yet (OPEN)

Everything short of the platform is proven on every pull request: the card
against the build, the blueprint read by the pinned orun, the leak gate, the
offline bootstrap with the blueprint's own hooks, the app's own plan for the
01-product merge, and the produced app bundled by wrangler, dry-deployed,
served, smoke-tested and read route by route in the job tree orun stages. Not
proven: the repository created and linked, the token minted, a real `wrangler
deploy`, the smoke against workers.dev, and the routes read from the deployed
Worker.

## EL-B — `elysia` pinned to 1.4.30 (ACCEPTED)

The template asks for `latest`. A floating version would change the Worker in
an owner's build with no commit; 1.4.30 is what npm called `latest` on the day
of the port. Upgrading is a pull request that bumps the pin, regenerates the
lockfile and runs the tests and the e2e.

## EL-C — Elysia's Cloudflare adapter is young (WATCH)

`elysia/adapter/cloudflare-worker` and its ahead-of-time `.compile()` have
worked on Workers only since 1.4.7. A release that changes how the adapter
answers (or moves code generation to request time, which workerd refuses)
breaks the Worker. The e2e serves the bundle under workerd and reads every
route on every change, so a bump that breaks it fails before it merges.

## EL-D — Elysia's filesystem features do not run on Workers (ACCEPTED)

`Elysia.file` and the static plugin read files from disk, which a Worker has
not got. The template uses neither. An owner who wants static files adds an
`assets` directory to `wrangler.jsonc` (Workers Static Assets serves it
before the Worker runs), and an owner who wants Elysia's OpenAPI plugin checks
it bundles for workerd first.

## EL-E — `orun plan --changed` and the first landing (WATCH)

`orun plan --changed` plans nothing for a push with no `before`; the first
landing relies on merging into orun's seeded initial commit, as every
baseline of this family does. Confirmed by the first live bootstrap.

## EL-F — stack-tectonic 0.19.0 (ACCEPTED)

The family's one pin (`intent.yaml` explains why 0.20.0 is not taken). The
Worker is bundled by wrangler's esbuild, so 0.20.0's linked install would
work for it; the pin is kept for one convention.

## EL-G — The welcome page's logo is hot-linked (ACCEPTED)

Upstream's page loads Elysia's logo from Vercel's framework-logo CDN. The port
loads Elysia's own (`https://elysiajs.com/assets/elysia.svg`): the page is the
template's, and the logo is Elysia's to serve. An owner who wants no
third-party request adds the file to a Static Assets directory and points the
`<img>` at it.
