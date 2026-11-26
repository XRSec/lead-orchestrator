# MODE TREE: TEAM (STAGED SPEC-TO-DELIVERY DAG)

Use for ambiguous requirements, cross-cutting architectural changes, or multi-wave initiatives requiring upfront alignment and a shared manifest SSOT.

## 1. Staged Execution DAG
```text
Stage 1: Scout Recon + Clarify Gate (skip ask if [skip-interview] or unambiguous)
   │
Stage 2: Living Scratch Manifest + [Optional Investigator Oracle Stress-Test]
   │
Stage 3: Foundation Seam -> Parallel/Sequential Implementer Lanes
   │
Stage 4: Fresh Verifier Gate <──(Warm Repair Loop <= 3)──> Owning Implementer(s)
   │
Stage 5: Optional Integrator (Worktree Landing / Docs Sync / User-Authorized SCM)
```

## 2. Stage Gates & Contracts
1. **Recon Before Interview**: If repo topology is unknown, run `scout` FIRST so you NEVER ask the user questions already answered by code/tests/docs. Unless `[skip-interview]` is set, ask at most **1–3 high-impact product/architecture questions** (with `(Recommended)` defaults) and wait for reply.
2. **Scratch Manifest SSOT**: Write `TASK_MANIFEST.md` to the host's untracked-safe scratch path (from `runtimes/*.md`):
   - `Verbatim Goal`: Exact user request + confirmed interview decisions.
   - `Seam Ownership Graph`: Wave order, `path:line` coordinates, owned vs read-only shared files, isolation mode (`worktree` vs `disjoint-seam`).
   - `Acceptance Criteria (AC-01..N)`: Observable outcomes + exact test/typecheck/lint commands.
3. **Oracle Gate (High-Risk Only)**: When touching concurrency, auth/security, persistence schemas, or public protocols, run `investigator` in **Oracle Mode** against the manifest before Stage 3 edits.
4. **Verification & Selective Memory**:
   - Advance past Stage 4 ONLY on `verifier` `Verdict: PASS` across 100% of ACs (`<= 3` warm repair rounds per seam).
   - If Nowledge Mem MCP is available: query `memory_search(limit=5)` during Stage 1 when historical context matters; after Stage 4 `PASS`, persist ONLY verified reusable architectural decisions or non-obvious root causes (deduplicate via `memory_search` -> `memory_update`/`memory_add`).
5. **Authorized Integration**: Dispatch `integrator` ONLY when isolated worktrees must be landed, docs/specs require updates, or the user explicitly requested `git commit` / squash / PR.
