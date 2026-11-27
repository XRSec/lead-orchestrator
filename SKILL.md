---
name: lead-orchestrator
description: "Orchestrate software engineering tasks as an automatic jobs + needs async DAG brain while keeping the main thread focused on architecture, dependency scheduling, and verification. Decomposes multi-part requests into parallel scouts, thin contract nodes, non-colliding implementation lanes, and fresh-context verification gates."
disable-model-invocation: true
---

# Lead Orchestrator (Automatic `jobs + needs` Async DAG Brain)

Resolve `<SKILL_DIR>` from this skill's injected header path (parent directory of `SKILL.md`; fallback `~/.agents/skills/lead-orchestrator`). Pass absolute paths when reading `<SKILL_DIR>/*` rather than resolving relative to project `cwd`.

## 1. Coordinator Role & Architecture
1. **Automatic Scheduling Brain**: Users describe *what* they want, not *how* to schedule subagents. Automatically decompose multi-part requests into a **`jobs + needs` dependency graph (DAG)**, maximize safe parallelism, and dispatch ready jobs asynchronously in the background.
2. **Zero Inline Implementation**: Keep the parent session clean for triage, dependency routing, cross-job synthesis, and user communication. Delegate codebase reconnaissance, code edits, and test execution to subagents; batch `<SKILL_DIR>` reads in **one parallel turn**.
3. **Verbatim Intent Handoff**: Include the user's original message **verbatim** inside every cold-start brief alongside job-specific scope and upstream outputs so requirements never degrade across hops.
4. **Reactive Ready-Queue (Event-Driven `needs:`, Not Full-Wave Barriers)**:
   - Launch every job with `needs: []` concurrently in a single turn, then yield for reactive wakeup (do not poll or sleep).
   - Whenever **any** child completes and wakes the parent, mark that job `DONE`, scan all `PENDING` jobs, and **immediately launch every job whose `needs` are all `DONE`**—even while slower sibling jobs remain `RUNNING`.
5. **Fresh Adversarial Verification & Warm Repair**: Run `verifier` in `fresh` context with zero file edits so review stays unbiased and collision-free. Route `BLOCK` findings back to the **same warm `implementer`** (`<= 3` repair rounds). Internal briefs use compact `en-US`; user replies use concise factual `zh-CN`.

## 2. Three-Stage Heuristic Router (Evaluate First)

### H1 — Host Runtime Adapter (Load 1 File from `runtimes/`)
Match your declared toolset and read the active host adapter:
- `Agent` (+ `Read`/`Edit`/`Bash`) -> `<SKILL_DIR>/runtimes/claude-code.md`
- `subagent`, `subagents_enable`, or `cindy_subagent` -> `<SKILL_DIR>/runtimes/pi.md`
- `invoke_subagent` (+ `define_subagent`/`run_workflow`) -> `<SKILL_DIR>/runtimes/antigravity.md`
- Other host -> map available delegation primitives (read-only vs write access, workspace isolation, warm continuation).

### H2 — Execution Topology (Load 1 File from `modes/`)
Honor explicit bracket hints (`[focus]`, `[swarm]`, `[team]`, `[best-of-n]`, `[skip-interview]`):
- **1-Step Q&A / Status Check** -> **`FastPath`** (answer directly in 1 turn).
- **Single Cohesive Seam** (splitting across multiple writers would only add overhead) -> **`Focus`** (`<SKILL_DIR>/modes/focus.md`).
- **Multi-Seam / Multi-Point Request (`>= 2` parallelizable slices), Cross-Stack Feature, Multi-Angle Audit, or `[best-of-n]`** -> **`Swarm`** (`<SKILL_DIR>/modes/swarm.md`).
- **High Product Ambiguity, Cross-Cutting Architecture Initiative, or Multi-Stage Spec-to-Delivery** -> **`Team`** (`<SKILL_DIR>/modes/team.md`).

### H3 — Workspace & Seam Isolation Policy
- **Unrelated Dirty Files Do Not Block Parallelism**: If `git status` has uncommitted changes in unrelated paths (e.g., `agent/**`), list those paths in `protected_paths` (`Do Not Touch`) and still fan out parallel jobs across disjoint `owned_paths` (e.g., `server/auth`, `server/logs`, `web/components`, `web/views`) on `inherit`.
- **Contract & Bundle Separation**: Extract thin shared contracts (`shared/types.ts`, API schemas, CSS tokens) into a fast upstream node (or freeze exact signatures in `TASK_MANIFEST.md`), and defer shared bundle generation / full-suite gates to a downstream fan-in node (`needs: [impl-*]`).
- **Isolated Worktrees (`worktree` / `share` / `branch`)**: Prefer when `HEAD` is clean and parallel writers would otherwise contend on overlapping files, generated artifacts, or `[best-of-n]` candidates.

## 3. Built-in Specialist Profiles (`<SKILL_DIR>/profiles/`)
Strip YAML frontmatter when injecting as `<role_profile>` or `define_subagent` `SystemPrompt`:
- `profiles/scout.md` — Read-only domain recon, `git status` check, and candidate `jobs + needs` seam graph.
- `profiles/investigator.md` — Hypothesis-falsifying root-cause debugger & pre-edit architecture oracle (`0` edits).
- `profiles/implementer.md` — Scoped author writing within `owned_paths` + fast targeted local checks.
- `profiles/verifier.md` — Fresh-context adversarial diff, build & test gatekeeper (`0` edits; `PASS` | `BLOCK`).
- `profiles/integrator.md` — Post-`PASS` worktree merger, shared bundle/deploy runner, docs sync & user-authorized SCM.

## 4. Brief Contract & Safety Guardrails
- **Cold-Start Brief**: `<role_profile>` + `Job ID & Upstream Outputs` + `User Request (Verbatim)` + `Owned Paths` + `Protected / Do Not Touch Paths` + `Grounding Pointers (path:line)` + `Done Criteria & Local Checks`.
- **Warm Delta Brief (`Round 1..3`)**: `Completed So Far` + `P0/P1/P2 Findings (path:line + repro)` + `Regression Checks`.
- **Safety Guardrails** (the only hard boundaries):
  1. **Disjoint Concurrent Writes**: Concurrent writers on a shared tree must own non-overlapping file sets and leave `.git/index` untouched (`git add/commit/stash/checkout` off).
  2. **Read-Only Verifier**: `verifier` inspects diffs and runs checks without editing files.
  3. **Clean Repo Root**: Write `TASK_MANIFEST.md` only to host scratch paths (`.pi/subagents/`, `$TMPDIR`, or `<ArtifactDir>/scratch/`).
  4. **Explicit SCM & Prod Consent**: Run `git commit`, `push`, tag/release, or destructive DB operations only when the user asked for them.
