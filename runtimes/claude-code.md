# RUNTIME ADAPTER: CLAUDE CODE (2.1+)

## 1. Primitive Mapping (`Agent` & `SendMessage`)
Claude Code `Agent` runs in the background by default (`run_in_background` defaults to `true`) and has **no `resume` field on `Agent`**. Always assign a unique lowercase `name` so the child session stays addressable via `SendMessage`:

| Profile (`profiles/*.md`) | `subagent_type` | Dispatch Call |
|---|---|---|
| **`scout`** | `"Explore"` (pure search/read) or `"general-purpose"` (if read-only bash needed) | `Agent({ name: "<seam>-scout", description: "...", subagent_type: "Explore", prompt })` |
| **`investigator` (fresh)** | `"general-purpose"` | `Agent({ name: "<topic>-oracle", description: "...", subagent_type: "general-purpose", prompt })` |
| **`investigator` (warm fork)** | `"fork"` | `Agent({ name: "<topic>-fork", description: "...", subagent_type: "fork", prompt })` *(omit `model` and `effort` on `"fork"`)* |
| **`implementer`** | `"general-purpose"` | `Agent({ name: "<seam>-impl", description: "...", subagent_type: "general-purpose", isolation?, prompt })` |
| **`verifier`** | `"general-purpose"` | `Agent({ name: "<seam>-verify", description: "...", subagent_type: "general-purpose", prompt })` |
| **`integrator`** | `"general-purpose"` | `Agent({ name: "release-integrator", description: "...", subagent_type: "general-purpose", prompt })` |

## 2. Async Fan-Out & Warm Continuation
1. **Parallel Sub-Trees**: Emit all independent `Agent(...)` calls in a **single assistant response**, then end your turn and wait for background completion wakes. Set `run_in_background: false` ONLY when nothing else can proceed and immediate blocking is required.
2. **Warm Continuation (Repair / Steer)**: Send delta feedback via `SendMessage({ to: "<name>", message: "<DELTA BRIEF>" })` (or returned `agentId`). Do NOT spawn a fresh `Agent` for normal Round 1–3 fixes unless `SendMessage` is unavailable in a stripped SDK host.
3. **Model & Effort**: Omit `model` and `effort` unless the user explicitly requested them.

## 3. Worktree & Scratch Rules on Claude Code
1. **`isolation: "worktree"` Gate**: Claude Code branches temporary worktrees from `HEAD`, **excluding uncommitted edits** in the parent working copy.
   - Clean parent checkout + build/test contention or Best-of-N -> pass `isolation: "worktree"` and instruct `implementer` to commit locally on its worktree branch so `integrator` can merge/cherry-pick it.
   - Dirty parent checkout -> OMIT `isolation` and enforce strictly disjoint `Authorized Scope` file lists in shared `cwd`.
2. **Scratch Manifest Path**: Store `TASK_MANIFEST.md` at `$TMPDIR/lo-manifest-<repo-name>.md`—NEVER write untracked `.t3/` files inside the repository.
