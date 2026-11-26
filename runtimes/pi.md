# RUNTIME ADAPTER: PI (`pi-subagents` & CINDY `cindy_subagent`)

Select Variant **P1** or **P2** from your declared tool names; keep total parent tool calls lean (`read` skill files in 1 parallel batch).

---

## Variant P1 — Native Pi (`subagent` / `subagents_enable`)

### 1. Lazy Activation Gate
If `subagent` is absent and `subagents_enable` is declared, call `subagents_enable({})` immediately; `subagent` unlocks on the next turn.

### 2. Role Mapping & Single Dispatch
- `scout` -> `agent: "scout"`
- `investigator` -> `agent: "oracle"` (or `"researcher"` when external web evidence via `pi-web-access` is required)
- `implementer` / `integrator` -> `agent: "worker"`
- `verifier` -> `agent: "worker"` when test/typecheck/lint shell gates must run (enforce `profiles/verifier.md` zero-edit rule in `task`), or `agent: "reviewer"` for pure read-only diff review.
- Launch: `subagent({ agent: "<role>", task: "<role_profile>\n...\n</role_profile>\n\n" + brief })` (`async: true` by default; yield turn for native wake—NEVER sleep/poll).

### 3. Multi-Tree Parallel & Staged Workflows
Legacy top-level `tasks` / `chain` are unsupported. Choose one:
- **Flat Parallel Fan-Out**: `subagent({ workflow: "parallel", args: { tasks: [{ agent, task, cwd?, model? }] } })`.
- **Multi-Lane / Staged Tree (`Swarm` / `Team`)**: Write ONE ```` ```js workflow ```` block + call `subagent({ workflow: true })` in the **same reply** using `runs.run`, `runs.all`, `runs.lanes`, `runs.steer`.
  - **Portability Redline**: Use top-level `await` and plain helper functions returning `runs.run(...)` ONLY—NEVER write nested `async function` or `async =>` (rejected by validator).
  - Do NOT set hard `toolBudget` or tight `usageBudget` on `worker`.

### 4. Warm Continuation & Worktree Gate
- **Resume Rotates ID**: `subagent({ action: "resume", id: latestRunId, message })` returns a **new `runId`** on every pass—always target the newest `runId`! Live steer uses `action: "steer"`.
- **`worktree: true` Cleanliness Gate**: `pi-subagents` rejects `worktree: true` if ANY tracked/untracked file is dirty (except `.pi/subagents/`). Use `worktree: true` ONLY on a clean checkout (for `~/.pi/agent/extensions` repos, place worktrees outside `extensions/`); if dirty, omit `worktree` and enforce disjoint file seams.
- **Scratch Manifest Path**: `.pi/subagents/TASK_MANIFEST.md` (exempt from dirty-worktree check) or child `output: "plans/manifest.md"`.

---

## Variant P2 — Cindy Pi (`cindy_subagent`)

### 1. Strict `agent` Enum & Tool Ceiling
`cindy_subagent` accepts ONLY `"scout" | "planner" | "reviewer" | "worker"`:
- `"scout"`, `"planner"`, `"reviewer"` are locked to `read,grep,find,ls` (no `bash`).
- Map `scout` -> `"scout"`, `investigator` -> `"planner"`, diff-only review -> `"reviewer"`, and **test-running `verifier`**, `implementer`, `integrator` -> `"worker"` with `<role_profile>` prepended.

### 2. Dispatch, Fan-Out & Continuation
- **Single**: `cindy_subagent({ agent, title, task, async: true, context: "fresh"|"fork" })`.
- **Parallel Batch (`<= 8` tasks, concurrency `4`)**: `cindy_subagent({ tasks: [{ agent, title, task }, ...], async: true })` (auto-collected at parent `agent_before_settle`).
- **Warm Continuation**: Terminal run -> `cindy_subagent({ action: "resume", taskId: runId, childId, message })`; live run -> `action: "steer" | "follow_up"`.
- **Shared-Tree Only**: `cindy_subagent` shares `cwd` (`isolation: "inherit"`). Enforce strictly disjoint `Authorized Scope` file seams and zero `.git/index` commands across parallel workers. Store scratch manifests in `$TMPDIR/lo-manifest-<session>.md`.
