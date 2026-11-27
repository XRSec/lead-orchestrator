# RUNTIME ADAPTER: CLAUDE CODE (2.1+)

## 1. Role Mapping (`Agent` & `SendMessage`)
Claude Code `Agent` runs asynchronously in the background by default (`run_in_background: true`). Give every job a unique lowercase `name` matching its `job_id` so it remains addressable via `SendMessage`:

| Profile (`profiles/*.md`) | `subagent_type` | Dispatch Call |
|---|---|---|
| **`scout`** | `"Explore"` (pure search/read) or `"general-purpose"` (if read-only bash needed) | `Agent({ name: "<job-id>", description: "[<job-id>] ...", subagent_type: "Explore", prompt })` |
| **`investigator` (fresh)** | `"general-purpose"` | `Agent({ name: "<job-id>", description: "[<job-id>] ...", subagent_type: "general-purpose", prompt })` |
| **`investigator` (warm fork)** | `"fork"` | `Agent({ name: "<job-id>", description: "[<job-id>] ...", subagent_type: "fork", prompt })` *(omit `model` and `effort` on `"fork"`)* |
| **`implementer`** | `"general-purpose"` | `Agent({ name: "<job-id>", description: "[<job-id>] ...", subagent_type: "general-purpose", isolation?, prompt })` |
| **`verifier`** | `"general-purpose"` | `Agent({ name: "<job-id>", description: "[<job-id>] ...", subagent_type: "general-purpose", prompt })` |
| **`integrator`** | `"general-purpose"` | `Agent({ name: "<job-id>", description: "[<job-id>] ...", subagent_type: "general-purpose", prompt })` |

## 2. Reactive `needs:` Fan-Out & Warm Continuation
1. **Parallel Ready Batch**: Emit all currently ready `Agent(...)` calls (`needs: []` initially) in a **single assistant response**, then end your turn for background completion wakes. Keep `run_in_background: true` (default) so multiple jobs run concurrently.
2. **Wake-and-Unlock (`needs:` Resolution)**: Each background `Agent` notifies the parent independently upon completion. On every completion wake, mark that `job_id = DONE` in `TASK_MANIFEST.md` and immediately emit `Agent(...)` calls for any `PENDING` jobs whose `needs` are now all `DONE`—even while other background agents are still running.
3. **Warm Continuation (Repair / Steer)**: Route `Round 1..3` fixes via `SendMessage({ to: "<job-id>", message: "<DELTA BRIEF>" })` (or returned `agentId`) so the author retains full context. Omit `model` and `effort` unless requested by the user.

## 3. Worktree & Scratch Handling
1. **Worktree vs Disjoint Seams**:
   - `isolation: "worktree"` branches from `HEAD`, omitting uncommitted parent edits. Use it when `HEAD` is clean and parallel jobs would contend on shared files/builds or `[best-of-n]` candidates (instruct each author to commit locally on its worktree branch for `integrator`).
   - When the parent working copy has uncommitted edits—or when jobs touch non-overlapping directories—omit `isolation`, list uncommitted user files in `Protected Paths`, and run parallel jobs on shared `cwd` with disjoint `Owned Paths`.
2. **Scratch Manifest Path**: `$TMPDIR/lo-manifest-<repo-name>.md`.
