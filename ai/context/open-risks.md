# elysia-api — open risks

What is not yet proven or not yet decided. The full write-up of every risk,
including the accepted and resolved ones, is
`specs/epics/elysia-api/risks-and-open-questions.md`; this page lists only
what is still live, so it should be short.

| ID | Risk | State | What closes it |
|---|---|---|---|
| EL-A | No live bootstrap yet: the repository created and linked, the deploy token minted, a real `wrangler deploy` of the bundle, the smoke and the probes against workers.dev, and the routes read from the deployed Worker are all unproven | OPEN | One bootstrap from a tag against a workspace |
| EL-C | Elysia's Cloudflare adapter is young (its ahead-of-time compile has worked on Workers only since 1.4.7); a release that changes it could break the Worker | WATCH | The e2e (the lane's build, dry run and `wrangler dev` over the bundle) on every change, and a deliberate `elysia` bump |
| EL-E | `orun plan --changed` plans nothing for a push with no `before`; the first landing relies on merging into orun's seeded initial commit | WATCH | Confirmed by the first live bootstrap |

Accepted, and worth knowing before you change them: EL-B (`elysia` pinned
to 1.4.30, not the template's `latest`), EL-D (Elysia's static plugin and
`Elysia.file` read the filesystem and do not run on Workers; the template uses
neither), EL-F (stack-tectonic 0.19.0), EL-G (the welcome page's logo is
hot-linked from elysiajs.com, as upstream hot-links Vercel's).
