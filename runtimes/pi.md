# RUNTIME ADAPTER: PI (`pi-subagents` & CINDY `cindy_subagent`)

Select Variant **P1** or **P2** from your declared tool names; read `<SKILL_DIR>` files in one parallel batch.

---

## Variant P1 — Native Pi (`subagent` / `subagents_enable`)

### 1. Activation & Role Mapping
- If `subagent` is absent and `subagents_enable` is declared, call `subagents_enable({})` first so `subagent` unlocks on the next turn.
- Role mapping:
  - `scout` -> `agent: "scout"`
  - `investigator` -> `agent: "oracle"` (or `"researcher"` when external web lookup is needed)
  - `implementer` / `integrator` -> `agent: "worker"`
  - `verifier` -> `agent: "worker"` when running test/typecheck/lint gates (with `profiles/verifier.md` read-only instructions), or `agent: "reviewer"` for pure diff review.

### 2. Encoding `jobs + needs` DAGs
Choose either interactive async wakes or a single declarative JS DAG workflow:
- **Programmatic `needs:` DAG (Recommended for Multi-Stage Parallel Graphs)**: Emit ONE ```` ```js workflow ```` block + call `subagent({ workflow: true })` in the same reply. Chain dependency promises per `needs:` edge using top-level `await`, `runs.run`, and `runs.all` (avoid nested `async function` or `async =>` for validator compatibility; leave `toolBudget`/`usageBudget` uncapped on `worker`):
  ```js
  const [sApi, sUi] = await runs.all([
    runs.run({ agent: "scout", task: scoutApiBrief }),
    runs.run({ agent: "scout", task: scoutUiBrief }),
  ]);
  const contract = await runs.run({ agent: "worker", task: buildContractBrief(sApi, sUi) });
  const [implApi, implUi] = await runs.all([
    runs.run({ agent: "worker", task: buildApiBrief(contract) }),
    runs.run({ agent: "worker", task: buildUiBrief(contract) }),
  ]);
  await runs.run({ agent: "worker", task: buildVerifyBrief(implApi, implUi) });
  ```
- **Flat Parallel / Interactive Ready-Queue**: Use `subagent({ workflow: "parallel", args: { tasks: [...] } })` or concurrent `subagent({ agent, task })` calls (`async: true` by default) and unlock ready `PENDING` jobs on each completion wake.

### 3. Warm Continuation & Worktree Policy
- `subagent({ action: "resume", id: latestRunId, message })` rotates `runId` on each pass—always track the newest `runId` (`action: "steer"` for live runs).
- `pi-subagents` accepts `worktree: true` only on a clean checkout (excluding `.pi/subagents/`). When dirty files exist, omit `worktree`, list dirty paths in `Protected Paths`, and run parallel jobs with disjoint `Owned Paths`.
- Scratch Manifest Path: `.pi/subagents/TASK_MANIFEST.md`.

---

## Variant P2 — Cindy Pi (`cindy_subagent`)

### 1. Role Mapping (`"scout" | "planner" | "reviewer" | "worker"`)
- `"scout"`, `"planner"`, `"reviewer"` have read-only file tools (`read,grep,find,ls`, no `bash`).
- Map `scout` -> `"scout"`, `investigator` -> `"planner"`, diff-only review -> `"reviewer"`, and shell-running `verifier` / `implementer` / `integrator` -> `"worker"` with `<role_profile>` prepended.

### 2. Parallel `needs:` Execution & Continuation
- **Ready-Batch Fan-Out (`<= 8` tasks, concurrency `4`)**: Dispatch all currently ready jobs (`needs` satisfied) via `cindy_subagent({ tasks: [{ agent, title: "[job-id] ...", task }, ...], async: true })` or parallel single calls; on completion wake, immediately dispatch newly unblocked `PENDING` jobs from `TASK_MANIFEST.md`.
- **Warm Continuation**: Completed run -> `cindy_subagent({ action: "resume", taskId: runId, childId, message })`; live run -> `action: "steer" | "follow_up"`.
- **Shared-Tree Disjoint Seams**: `cindy_subagent` runs in shared `cwd`; assign non-overlapping `Owned Paths`, defer shared bundling to a fan-in job, and store `TASK_MANIFEST.md` at `$TMPDIR/lo-manifest-<session>.md`.
