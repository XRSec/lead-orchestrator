# lead-orchestrator

English | [简体中文](./README.zh-CN.md)

A **Heuristic + Multi-Tree** multi-agent orchestration skill that turns your active coding session into a strict coordinator across **Claude Code (2.1+)**, **Pi (`pi-subagents` & Cindy `cindy_subagent`)**, and **Antigravity (AGY)** — with **5 built-in specialist profiles** replacing scattered `~/.agent/agent/*.md` files.

---

## Why Multi-Tree + Heuristics?

Monolithic orchestration prompts waste context on irrelevant host syntax and force every task through a rigid waterfall. `lead-orchestrator` uses a **55-line root router (`SKILL.md`)** with three zero-cost heuristics (`H1`–`H3`) to load **only the active branch** across three orthogonal trees:

```text
lead-orchestrator/
├── SKILL.md                    # Heuristic Root Router (H1 Host × H2 Mode × H3 Isolation)
├── agents/
│   └── openai.yaml             # Explicit-invocation policy metadata
├── runtimes/                   # Tree A: Host Runtime Adapter (loads ONLY 1 active host)
│   ├── claude-code.md          # Claude Code 2.1+ (Agent + SendMessage + worktree gate)
│   ├── pi.md                   # Native Pi (pi-subagents / js workflow) & Cindy (cindy_subagent)
│   └── antigravity.md          # Antigravity (define_subagent / invoke_subagent / run_workflow)
├── modes/                      # Tree B: Execution Topology (loads ONLY 1 selected mode)
│   ├── focus.md                # Single-seam deep loop (verbatim -> author -> verifier -> warm delta)
│   ├── swarm.md                # Multi-tree sharding, Best-of-N worktrees & multi-angle audits
│   └── team.md                 # Staged DAG (recon -> clarify -> scratch manifest -> oracle -> lanes)
└── profiles/                   # Tree C: Built-in Specialist Personas (replaces ~/.agent/agent/*.md)
    ├── scout.md                # Read-only recon, dependency/seam graph & cleanliness check
    ├── investigator.md         # Hypothesis-falsifying debugger & pre-edit architecture oracle
    ├── implementer.md          # Scoped code author with strict seam/worktree boundary lock
    ├── verifier.md             # Fresh-context adversarial diff & test gatekeeper (0 file edits)
    └── integrator.md           # Post-PASS multi-worktree merger, docs sync & authorized SCM
```

## Core Design Principles

- **Zero Inline Code-Touch in Delegated Modes**: The parent session triages, clarifies, verifies, and synthesizes — never pre-exploring or patching source code inline.
- **Verbatim Intent Fidelity**: Every cold-start brief embeds the user's original prompt verbatim alongside scoped coordinates so intent never degrades across hops.
- **Collision-Free Multi-Tree Isolation (`H3`)**:
  - **Clean `HEAD` + build/test contention or Best-of-N**: Spawns isolated Git worktrees (`isolation: "worktree"`, `worktree: true`, `Workspace: "share"`).
  - **Dirty parent checkout or shared-cwd host**: Automatically falls back to **Shared-Tree Disjoint Seams** (`Authorized Scope` lock + zero `.git/index` mutation) instead of failing worktree preflight or dropping uncommitted user edits.
- **Fresh Adversarial Verification + Warm Repair**: `verifier` always runs in `fresh` context and **never edits files**; `BLOCK` findings route back to the **same warm `implementer`** session (`<= 3` bounded rounds).
- **Untracked-Safe Scratch Manifests**: Stores transient manifests in host-ignored scratch paths (`.pi/subagents/`, `$TMPDIR`, or `<ArtifactDir>/scratch/`) so git status stays clean.

---

## Installation

### Skills CLI (`~/.agents/skills` / Claude Code / Codex / Cursor / Pi)
```bash
npx skills add XRSec/lead-orchestrator
```

### Pi Coding Agent (`pi install`)
```bash
pi install git:github.com/XRSec/lead-orchestrator
```

### Manual Clone / Symlink
```bash
git clone https://github.com/XRSec/lead-orchestrator.git ~/.agents/skills/lead-orchestrator
ln -s ~/.agents/skills/lead-orchestrator ~/.claude/skills/lead-orchestrator
ln -s ~/.agents/skills/lead-orchestrator ~/.gemini/config/skills/lead-orchestrator
```

---

## Usage

Invoke explicitly via slash command (`/lead-orchestrator` or `/skill:lead-orchestrator` in Pi). Auto-triage picks the lightest valid topology, or pass bracket hints to force a mode:

| Invocation | Behavior |
| --- | --- |
| `/lead-orchestrator <task>` | Auto-scores task via `H1`–`H3` into `FastPath`, `Focus`, `Swarm`, or `Team` |
| `/lead-orchestrator [focus] <task>` | Force single-seam deep implementation/investigation + fresh verifier loop |
| `/lead-orchestrator [swarm] <task>` | Force parallel non-colliding seam trees + peer verifiers (`1` per `2–4` writers) |
| `/lead-orchestrator [best-of-n] <task>` | Run competing candidate implementations in isolated worktrees and land the winner |
| `/lead-orchestrator [team] <task>` | Full staged DAG: `scout` -> clarify gate -> scratch manifest -> oracle -> lanes |
| `/lead-orchestrator [skip-interview] <task>` | Bypass upfront clarification gate in `Team` mode and record explicit assumptions |

## License

[MIT](../LICENSE) © XRSec
