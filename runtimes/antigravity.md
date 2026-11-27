# RUNTIME ADAPTER: ANTIGRAVITY (AGY)

## 1. Profile Binding & Parallel Dispatch
Bind `<SKILL_DIR>/profiles/<role>.md` via session-scoped typed subagents (reusable across DAG nodes) or built-in types:
1. **Session-Scoped Typed Agent (Recommended for Multi-Job DAGs)**:
   - Define needed roles in a single parallel batch once per conversation:
     `define_subagent(Name="lo-<role>", Description=..., SystemPrompt=<Markdown body of profiles/<role>.md>, EnableWriteTools=<false for scout/investigator, true for implementer/verifier/integrator>)`
   - Launch all currently ready jobs (`needs` satisfied) **in parallel in the same turn**:
     `invoke_subagent(TypeName="lo-<role>", Role="[<job_id>] <short job title>", Prompt=<brief>, Workspace="inherit"|"share"|"branch")`
2. **Built-in Fast Path**:
   - Read-only recon (`0` shell needed): `invoke_subagent(TypeName="research", Role="[<job_id>] ...", Prompt=...)`
   - Full-tool worker: `invoke_subagent(TypeName="self", Role="[<job_id>] ...", Prompt="<role_profile>...\n" + brief, Workspace=...)`
   - Live Web / UI check: `invoke_subagent(TypeName="browser", Model="flash", Role="[<job_id>] ...", Prompt=...)`

## 2. Reactive `needs:` Loop & Plan-Mode Discipline
1. **Plan Mode (`enter_plan_mode` / `exit_plan_mode`)**:
   - If the host starts in Plan Mode, avoid spending minutes reading dozens of source files serially in the parent thread. Perform only minimal orientation (`<SKILL_DIR>` reads + `git status -sb` / directory listing, or parallel read-only `research` / `lo-scout` subagents when available), then structure `<plan>` directly as a **`jobs + needs` DAG** (explicitly marking which jobs launch together in parallel vs which wait on `needs: [...]`) rather than a sequential 1-worker-per-step list.
2. **Interactive Reactive DAG (Default — Full UI Card Visibility + Warm Repair)**:
   - `invoke_subagent` runs asynchronously in the background and wakes the parent session **as soon as any individual child finishes**.
   - On every wakeup: mark that `job_id = DONE` in `TASK_MANIFEST.md` (and `update_steps`), check all `PENDING` jobs, and immediately issue `invoke_subagent` calls in parallel for every job whose `needs` are now all `DONE`—do not wait for slower sibling jobs to finish first.
   - Prefix `Role` with `[<job_id>]` (e.g., `[impl-auth] Admin & API Key Ownership`) so each background task card clearly shows its DAG node in the UI.
3. **Programmatic `asyncio` DAG (`run_workflow` — High-Fanout `>= 6` Homogeneous Slices)**:
   - When orchestrating many uniform tasks where parent intervention between nodes is unnecessary, load the `run-workflow` skill and encode `needs:` directly with Python `await` and `parallel()`. Prefer `invoke_subagent` for standard 2–6 job feature DAGs so each worker appears as an independent inspectable subagent card and supports warm `send_message` continuation.
4. **Warm Continuation & Isolation**:
   - Warm repair (`Round 1..3`): `send_message(Recipient=conversationId, Message="<DELTA BRIEF>")`.
   - Workspace mode: `Workspace="share"` (or `"branch"`) for isolated parallel trees on a clean `HEAD`; `Workspace="inherit"` with disjoint `owned_paths` when preserving uncommitted parent edits or editing non-overlapping modules.
   - Scratch Manifest Path: `<Artifact Directory Path>/scratch/TASK_MANIFEST.md`.
