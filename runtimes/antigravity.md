# RUNTIME ADAPTER: ANTIGRAVITY (AGY)

## 1. Profile Binding & Dispatch
Bind `<SKILL_DIR>/profiles/<role>.md` either dynamically for reuse across the conversation or inline:
1. **Session-Scoped Typed Agent (Preferred for Multi-Round / Swarm)**:
   - `define_subagent(Name="lo-<role>", Description=..., SystemPrompt=<Markdown body of profiles/<role>.md>, EnableWriteTools=<false for scout/investigator, true for implementer/verifier/integrator>)`
   - `invoke_subagent(TypeName="lo-<role>", Role="<seam-specific title>", Prompt=<brief>, Workspace="inherit"|"share"|"branch")`
2. **Built-in Fast Path**:
   - Read-only without shell: `invoke_subagent(TypeName="research", Role=..., Prompt="<role_profile>...\n" + brief)`
   - Full-tool leaf: `invoke_subagent(TypeName="self", Role=..., Prompt="<role_profile>...\n" + brief, Workspace=...)`
   - Live Web / UI Verification: `invoke_subagent(TypeName="browser", Model="flash", Role=..., Prompt=...)`

## 2. Multi-Tree Execution & Programmatic Pipelines
- **Isolated Parallel Author Trees**: Launch concurrent `implementer` agents with `Workspace="share"` (independent working tree sharing repo objects) or `"branch"`. Instruct each isolated author to commit locally in its workspace before completion; `integrator` merges/cherry-picks into parent workspace BEFORE any subagent workspace is cleaned up. Keep readers and `verifier` on `Workspace="inherit"`.
- **High-Fanout / Staged Pipelines (`>= 4` slices)**: Use `run_workflow` (load `run-workflow` skill first) with `phase()`, `agent()`, `parallel()`, and `pipeline()`.
- **Warm Continuation**: `send_message(Recipient=conversationId, Message="<DELTA BRIEF>")` to keep the author's warm context across repair rounds.
- **Scratch Manifest Path**: `<Artifact Directory Path>/scratch/TASK_MANIFEST.md`.
