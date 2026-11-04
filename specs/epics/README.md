# Epics

Orun-style work programs. Each epic is a folder carrying `epic.yaml` (the
declaration the platform reads: slug, key prefix, milestones with exit
criteria and task keys) beside its doc set. The pull request that adds the
folder is the adoption; the merge creates the epic in the workspace.

| Epic | Cluster | Status | What it is |
|------|---------|--------|------------|
| [`elysia-api/`](./elysia-api/) | **EL** | In progress (EL0) | Vercel's Elysia on Vercel template, its route kept, on Elysia's own Cloudflare adapter in one Cloudflare Worker — and an Orun baseline that instantiates it in one landing (EL2) |

As-built ≠ intent: what shipped lives in each epic's `IMPLEMENTATION-STATUS.md`.
