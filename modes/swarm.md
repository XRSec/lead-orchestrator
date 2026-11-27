# MODE TREE: SWARM (AUTOMATIC `jobs + needs` ASYNC DAG)

Use whenever a request spans `>= 2` parallelizable slices, cross-stack work (e.g., backend + UI + CLI/ops), multi-angle audits, or `[best-of-n]` candidate implementations.

## 1. Automatic Graph-Shaping Patterns

Design the DAG so mid-stage width is `>= 2` whenever independent boundaries exist:

### Pattern S1 — Contract-Split Parallel DAG (Default for Multi-Point / Cross-Stack Tasks)
```text
[scout-backend] (needs: []) ──┐
                              ├──> [freeze-contract] (needs: [scouts]) ──┬──> [impl-seam-a] (needs: [contract]) ──┐
[scout-web-ui]  (needs: []) ──┘    (or inline contract in manifest)      ├──> [impl-seam-b] (needs: [contract]) ──┼──> [verify-and-bundle] ──> [deploy/bench]
                                                                         └──> [impl-seam-c] (needs: [contract]) ──┘    (needs: [impl-*])       (needs: [verify])
```
- **Parallel Domain Recon**: When coordinates span multiple areas (backend, DB, frontend, external skills), launch 2–3 read-only `scout` jobs with `needs: []` in Turn 1 rather than reading source files serially in the parent thread.
- **Thin Upstream Contracts**: Do not bundle a 5-line shared contract (`shared/types.ts`, route DTOs, CSS custom properties) together with a 5-minute implementation. Either:
  1. Freeze exact TypeScript/JSON/CSS contract definitions inside `TASK_MANIFEST.md` and hand them to parallel `implementer` jobs (`impl-backend` owns `shared/types.ts` matching the frozen spec while `impl-web-*` code against the same spec in parallel), or
  2. Run a tiny `< 1 min` `contract` job touching *only* the shared contract/token files so all `impl-*` jobs unlock concurrently right after.
- **Defer Shared Artifacts to Fan-In**: If building writes shared outputs (e.g., `ui-bundle.ts`, lockfiles, full integration DB), keep `impl-*` jobs focused on source edits + fast per-seam `tsc`/unit checks, and run shared bundling + full test gates inside `verify-and-bundle` (`needs: [impl-*]`).

### Pattern S2 — Tournament / Best-of-N Worktrees (`[best-of-n]` or Competing Designs)
```text
Clean HEAD ──┬── [impl-candidate-a] (isolated worktree, needs: []) ──┐
             └── [impl-candidate-b] (isolated worktree, needs: []) ──┴──> [verify-compare] (needs: [a, b]) ──> [integrate-winner]
```
- Run candidates in isolated worktrees (`isolation: "worktree"` / `worktree: true` / `Workspace: "share"`), benchmark objectively in `verify-compare`, and land the winner via `integrator`.

### Pattern S3 — Parallel Orthogonal Audits (Read-Only Fan-Out)
- Launch 2–4 `fresh` read-only jobs (`verifier` or `investigator`, `needs: []`) with orthogonal lenses (`correctness & state`, `security & auth`, `UI/UX & edge cases`, `test & perf`), then deduplicate `P0/P1/P2` findings in the coordinator.

## 2. DAG Manifest & Reactive Ready-Queue Loop

1. **Declare the Job Table** (save to scratch `TASK_MANIFEST.md` when `>= 3` jobs):
   ```markdown
   | job_id | role | needs | owned_paths (write scope) | status | worker_ref |
   |---|---|---|---|---|---|
   | scout-api | scout | [] | (read-only) | DONE | ... |
   | contract | implementer | [scout-api, scout-ui] | shared/types.ts, web/src/tokens.css | DONE | ... |
   | impl-auth | implementer | [contract] | server/src/routes/auth.ts, server/src/db/migrate.ts | RUNNING | ... |
   | impl-logs | implementer | [contract] | server/src/routes/commands.ts, server/src/routes/logs.ts | DONE | ... |
   | impl-ui | implementer | [contract] | web/src/views/**, web/src/components/** | RUNNING | ... |
   | verify | verifier | [impl-auth, impl-logs, impl-ui] | (read-only + build/test gates) | PENDING | - |
   ```
2. **Wake-and-Unlock Dispatch Algorithm**:
   - **Initial Launch**: Dispatch every job whose `needs` list is empty in **one parallel tool batch**, then end turn.
   - **On Every Child Wakeup**:
     1. Record the completed job's `status` (`DONE` or `BLOCKED`) and key handback coordinates.
     2. Find every `PENDING` job where **all** IDs in `needs` are now `DONE`.
     3. Immediately dispatch those newly ready jobs in parallel in the current turn (do **not** wait for unrelated `RUNNING` jobs to finish first).
   - **Targeted Repair**: Spawn `1` fresh `verifier` per `2–4` closely related writers (or per independent subgraph). If `verifier` returns `BLOCK`, send a warm delta brief only to the specific `worker_ref` owning the defective file seam, then re-verify.
