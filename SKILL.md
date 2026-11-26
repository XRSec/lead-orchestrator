---
name: lead-orchestrator
description: "Heuristic multi-tree orchestrator across Claude Code, Pi (pi-subagents / Cindy), and Antigravity. Uses zero-cost heuristics to route across Host Runtime Trees (runtimes/*), Execution Topology Trees (modes/*: Focus, Swarm/Multi-Worktree, Team DAG), and Built-in Specialist Profile Trees (profiles/*), replacing external ~/.agent/agent/*.md files."
disable-model-invocation: true
---

# Lead Orchestrator (Heuristic Multi-Tree Router)

Resolve `<SKILL_DIR>` from this skill's injected header path (parent directory of `SKILL.md`; fallback `~/.agents/skills/lead-orchestrator`). Always pass absolute paths when reading `<SKILL_DIR>/*`—NEVER resolve relative to project `cwd`.

## 1. Prime Invariants
1. **COORDINATOR, NEVER INLINE WORKER**: In delegated modes, NEVER edit repo files, run heavy test suites, or pre-explore source code in the parent thread. Route immediately; batch any needed `<SKILL_DIR>` file reads in **one parallel turn**.
2. **VERBATIM INTENT FIDELITY**: Embed the user's original message **verbatim** inside every initial brief alongside scoped context. Never replace user intent with speculative parent summaries.
3. **COLLISION-FREE MULTI-TREE ISOLATION**: Two concurrent writers MUST NEVER touch overlapping files. Choose between **Isolated Git Worktrees** and **Shared-Tree Disjoint Seams** via Heuristic `H3`—never blind-force worktrees onto dirty checkouts.
4. **FRESH ADVERSARIAL VERIFICATION**: Always spawn `verifier` in `fresh` context. `verifier` runs real checks and reports `P0/P1/P2`—it MUST NEVER edit files. Route fixes back to the **same warm `implementer`** (`MAX_REPAIR_ROUNDS = 3`).
5. **REACTIVE ASYNC & BILINGUAL PLANE**: Launch background children and yield turn for reactive wakeup—NEVER poll or sleep. Internal briefs/manifests use dense `en-US`; user replies use concise factual `zh-CN`.

## 2. Three-Stage Heuristic Router (Evaluate Before First Tool Call)

### H1 — Host Runtime Tree (0-Call Toolset Fingerprint -> Load 1 File from `runtimes/`)
Inspect your own declared tools and load **only** the matching host adapter:
- Toolset has `Agent` (+ `Read`/`Edit`/`Bash`) -> `<SKILL_DIR>/runtimes/claude-code.md`
- Toolset has `subagent`, `subagents_enable`, or `cindy_subagent` -> `<SKILL_DIR>/runtimes/pi.md`
- Toolset has `invoke_subagent` (+ `define_subagent`/`run_workflow`) -> `<SKILL_DIR>/runtimes/antigravity.md`
- Other host -> inspect available delegation tool schema; map read-only vs write ceiling, isolation, and session continuation.

### H2 — Execution Topology Tree (Task Shape Scorer -> Load 1 File from `modes/`)
Honor explicit bracket overrides (`[focus]`, `[swarm]`, `[team]`, `[best-of-n]`, `[skip-interview]`):
- **Control / 1-Step Read Q&A** -> **`FastPath`** (answer inline in 1 turn; load nothing).
- **1 Seam, Clear Goal, Bugfix / Feature / Root-Cause Debug** -> **`Focus`** (`<SKILL_DIR>/modes/focus.md`).
- **`>= 2` Independent Seams, Multi-Angle Audit, or Competing Approaches** -> **`Swarm`** (`<SKILL_DIR>/modes/swarm.md`).
- **High Ambiguity, Shared-Contract Refactor, or Multi-Wave DAG** -> **`Team`** (`<SKILL_DIR>/modes/team.md`).

### H3 — Workspace & Session Tree Policy
- **Multi-Worktree (`worktree` / `share` / `branch`)**: Use ONLY when host supports per-child trees, parent checkout is **clean** (`git status --porcelain` empty, or AGY `share`), AND parallel writers contend on build/test state or run **Best-of-N** competing solutions.
- **Shared-Tree Disjoint Seams (`inherit`)**: Mandatory when parent checkout has uncommitted user edits, on `cindy_subagent`, or when parallel writers edit strictly non-overlapping file paths with zero `.git/index` commands.
- **Session Context Trees**: Default `fresh` for `scout`, `implementer`, `verifier`; use `fork` ONLY to branch warm investigation context; use **warm continuation** (`SendMessage` / `resume` / `steer` / `send_message`) on the existing author tree for repair rounds `1..3`.

## 3. Built-in Specialist Profile Tree (`<SKILL_DIR>/profiles/` — Replaces `~/.agent/agent/*.md`)
Strip YAML frontmatter and inject Markdown body as `<role_profile>` (or native `define_subagent` prompt):
- `profiles/scout.md` — Read-only recon, dependency/seam graph, clean-tree & verification command discovery.
- `profiles/investigator.md` — Hypothesis-falsifying debugger & pre-edit architecture oracle (`0` edits).
- `profiles/implementer.md` — Scoped author locked to `Authorized Scope` + local sanity verification.
- `profiles/verifier.md` — Fresh-context adversarial diff & test/lint gatekeeper (`0` edits; `PASS` | `BLOCK`).
- `profiles/integrator.md` — Post-`PASS` multi-worktree merger, docs sync & user-authorized SCM finisher.

## 4. Universal Brief Contract (Every Fresh Child Starts Cold)
- **Cold-Start Brief**: `<role_profile>` + `User Request (Verbatim)` + `Prior Context` (omit on Turn 1) + `Authorized Scope` + `Do Not Touch` + `Grounding Pointers` (`path:line` / scratch manifest, never prose dumps) + `Done Criteria & Validation Commands`.
- **Warm Delta Brief (`Round 1..3`)**: `Completed So Far` + `P0/P1/P2 Findings (path:line + repro)` + `Scope & Regression Invariants`.

## 5. Hard Redlines
- **RL-01**: NEVER edit repo files in parent thread during delegated modes or let `verifier` edit files.
- **RL-02**: NEVER drop untracked `.t3/` manifests into git working trees (use host scratch/artifact paths: `.pi/subagents/`, `$TMPDIR`, or `<ArtifactDir>/scratch/`).
- **RL-03**: NEVER spawn cold replacements for normal repair rounds (`<= 3` warm rounds; circuit-break to user after Round 3).
- **RL-04**: NEVER execute `git commit`, `push`, squash, or PR creation without explicit user authorization.
