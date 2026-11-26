# MODE TREE: SWARM (HEURISTIC MULTI-TREE PARALLELISM)

Use when work splits into `>= 2` non-colliding seams, multi-angle parallel audits, or **Best-of-N** competing candidate trees.

## 1. Choose Sub-Tree Shape via Heuristics

### Shape S1 — Partitioned Seam Trees (Independent Modules / Files)
```text
[Optional Scout: Map Seams & Check Cleanliness]
 ├── Lane Tree A: Implementer-A (Seam A) ──┐
 ├── Lane Tree B: Implementer-B (Seam B) ──┼──> Fresh Verifier (1 per 2–4 lanes) ──> [Warm Repair <=3] ──> Integrator (if multi-worktree)
 └── Lane Tree C: Implementer-C (Seam C) ──┘
```
- **Sizing Heuristic**: Count pieces that genuinely proceed without shared-file edits. Never crowd a single function refactor with multiple writers.
- **Shared Interface Rule**: If seams share types/contracts, run a tiny **Wave 0** (`1 implementer` updates shared types) BEFORE fanning out Wave 1 parallel seam trees.

### Shape S2 — Tournament / Best-of-N Worktrees (`[best-of-n]` or High Design Uncertainty)
```text
Clean HEAD ──┬── Worktree 1: Implementer (Approach A) ──┐
             └── Worktree 2: Implementer (Approach B) ──┴──> Fresh Verifier (Comparative Benchmark & Audit) ──> Integrator Lands Winner & Cleans Loser
```
- Require **isolated worktrees** (`isolation: "worktree"` / `worktree: true` / `Workspace: "share"`). Never run competing edits on a shared tree.

### Shape S3 — Parallel Audit Trees (Read-Only Multi-Angle Review / Research)
- Spawn 2–4 `fresh` read-only specialists (`verifier` or `investigator`) on `inherit` workspace with **orthogonal lenses** (e.g., `correctness & races`, `security & boundaries`, `test adequacy & perf`), then synthesize deduplicated `P0/P1/P2` findings.

## 2. Multi-Tree Isolation & Brief Discipline
1. **Isolation Gate**: Clean `HEAD` + build/test contention (or Shape S2) -> **Isolated Worktrees**; Dirty parent checkout or `cindy_subagent` -> **Shared-Tree Disjoint Seams** (`Authorized Scope` lists exact owned paths + `Do Not Touch` lists sibling seams + forbid `git add/commit/stash/checkout`).
2. **Pointer Addressing**: Name every worker by job (`parser-impl`, `ui-impl`, `core-verify`). Pass manifest/artifact paths or `path:line` coordinates—NEVER copy-paste one worker's full prose report into another's brief.
3. **Verifier Ratio**: Spawn `1` fresh `verifier` per `2–4` writers. `verifier` attributes each `BLOCK` finding to its owning seam (`@<seam>-impl`); only affected warm authors run repair rounds.
