# Handover — GLM-4.7 on Intel Gaudi 3 (preview)

**Branch:** `gaudi3-preview` · **Owner handing off:** jm@farmgpu.com · **Date:** 2026-06-08

## What this is

An **isolated preview** of GLM-4.7 benchmarked on Intel Gaudi 3 HL-325L, to share with Intel
ahead of more benchmarking. The curves will be used to extrapolate GLM-5.1 / DSv4 / Kimi /
MiniMax behavior on Gaudi 3.

> ⚠️ **Keep this isolated.** Do **not** merge `gaudi3-preview` into `master`, and do not point it
> at the production inferencex.com Neon DB. It has its own Supabase DB and should get its own
> Vercel project. Data is preliminary.

## Dataset

`glm4.7 / gaudi3 / vllm-gaudi / BF16 / TP=8 / ISL=OSL=1024`, run date `2026-03-14`,
8 concurrencies `[1, 2, 4, 8, 16, 32, 64, 128]`. Source JSON in `glm-4.7/results/`.

## Status

### Done & verified

- **Code** committed on `gaudi3-preview`:
  - `806f001` — registry + supplemental data (feat)
  - `b51cfd0` — ingest developer guide (docs)
  - Touched: `packages/constants/src/{gpu-keys,models,framework-aliases}.ts`,
    `packages/db/src/etl/normalizers.ts`, `packages/db/data/supplemental-bmk.json`,
    `packages/app/src/lib/data-mappings.ts` (adds the `GLM-4.7 (Gaudi 3 preview)` dropdown
    entry, category `experimental`).
- **Supabase** project `inferencex-glm47-gaudi3-preview`
  (ref **`gtvtkgpwuhpsfiugezbe`**, org `xigwilkpopfmmvflwwjj`, us-west-1, Postgres 17) —
  ACTIVE_HEALTHY, migrations applied, **8 benchmark rows ingested**. Confirmed present.
- **Upstream sync (2026-06-08):** merged 112 commits from `origin/master` (SemiAnalysisAI)
  into `gaudi3-preview` — merge commit `24e3f39`. One conflict in `data-mappings.ts` resolved
  (kept upstream's param-count labels _and_ the GLM-4.7 entry). `pnpm typecheck` / lint / fmt
  all pass.

### Open items for the next engineer

1. **Push the merge** — `gaudi3-preview` is ahead of `farmgpu/gaudi3-preview` by the merge +
   112 upstream commits. Not yet pushed: `git push farmgpu gaudi3-preview`.
2. **Vercel preview project** — a dedicated Vercel project pointing at the Supabase DB above was
   **not found** under the personal team `jm-farmgpucoms-projects`. Either it lives under another
   team or it still needs creating. Set its `DATABASE_READONLY_URL` / `DATABASE_WRITE_URL` to the
   Supabase **direct (port 5432)** Postgres URL with `DATABASE_DRIVER=postgres` and
   `DATABASE_SSL=true`, deploy, verify the GLM-4.7 / Gaudi 3 curves render in
   throughput-vs-interactivity and throughput-vs-latency, then share the preview URL with Intel.
3. **Security:** Supabase **RLS is disabled** on all 9 public tables — anyone with the project's
   anon key can read/write. Acceptable while the app connects via the direct Postgres URL and the
   anon key stays private; enable RLS (with read policies) before exposing the anon key anywhere.

## Remotes

- `origin` → `github.com/SemiAnalysisAI/InferenceX-app` (upstream — fork source)
- `farmgpu` → `github.com/FarmGPU/InferenceX-app` (our fork; `gaudi3-preview` lives here)

## Re-ingest / refresh data

With the Supabase direct Postgres URL in `.env` (`DATABASE_WRITE_URL` + `DATABASE_READONLY_URL`,
`DATABASE_DRIVER=postgres`, `DATABASE_SSL=true`):

```bash
pnpm install
pnpm admin:db:migrate
pnpm admin:db:ingest:supplemental -y
pnpm dev   # verify curves render
```

See `glm-4.7/guide.md` for the full ingest walkthrough.
