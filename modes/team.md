# MODE TREE: TEAM (SPEC-TO-DELIVERY `jobs + needs` DAG)

Use for ambiguous product requirements, cross-cutting architectural initiatives, or multi-subsystem projects that benefit from upfront alignment and an explicit `TASK_MANIFEST.md` single source of truth.

## 1. End-to-End Dependency Graph
```text
[scout-domain-1..k] (parallel, needs: [])
   │
   ▼
[Coordinator: Clarify Gate (if needed) + Compile TASK_MANIFEST.md DAG]
   │
   ├──> [Optional: oracle-review] (investigator, high-risk auth/schema/concurrency)
   ▼
[contract / foundation] (thin shared types/tokens, needs: [scouts / oracle])
   │
   ├──> [impl-lane-1] (needs: [contract]) ──┐
   ├──> [impl-lane-2] (needs: [contract]) ──┼──> [verify-subgraph] (fresh, 0 edits) <──(warm delta <= 3)──> [owning impl-lane]
   └──> [impl-lane-3] (needs: [contract]) ──┘              │
                                                           ▼ (on PASS)
                                                [integrate / deploy / docs] (needs: [verify-*])
```

## 2. Stage Contracts & Reactive Scheduling

1. **Parallel Recon Before Questions**:
   - When codebase coordinates are unknown, dispatch 1–3 parallel read-only `scout` jobs (`needs: []`) first so you never ask the user questions already answered by repo code, tests, or docs.
   - Unless `[skip-interview]` is present or requirements are already clear, ask at most **1–3 high-impact product/architecture questions** (with `(Recommended)` defaults) and pause for the user's answer.
2. **Compile `TASK_MANIFEST.md` (Untracked Scratch Path)**:
   - `Verbatim Goal`: Exact user prompt + confirmed decisions.
   - `Frozen Contracts`: Shared DTOs, route query params, DB invariants, or UI design tokens agreed upfront so parallel lanes don't block each other.
   - `Job DAG Table`: `job_id`, `role`, `needs: [...]`, `owned_paths`, `protected_paths`, `status`, `worker_ref`.
   - `Acceptance Criteria (AC-01..N)`: Observable outcomes + exact build/test/runtime verification commands.
3. **Optional Oracle Check (High-Risk Seams Only)**:
   - When changing auth/privilege models, persistence migrations, concurrency control, or public wire protocols, run a fast `investigator` (`Oracle Mode`) against the manifest before launching writers.
4. **Reactive `needs:` Execution & Verification**:
   - Drive all implementation, verification, and post-verification jobs with the **Ready-Queue Loop**: on each background completion wake, mark the job `DONE` and immediately launch every `PENDING` job whose `needs` are satisfied—independent subgraphs can verify or benchmark early while sibling subgraphs are still coding.
   - Advance to `integrator` / deployment / live benchmarking only after its upstream `verifier` reports `Verdict: PASS`.
   - If Nowledge Mem MCP is available, search relevant prior context during recon (`memory_search(limit=5)`) and persist reusable architectural lessons after `PASS`.
