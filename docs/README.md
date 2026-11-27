# lead-orchestrator

English | [简体中文](./README.zh-CN.md)

An **Automatic `jobs + needs` Async DAG** multi-agent orchestration skill that turns your active coding session into an event-driven scheduling brain across **Claude Code (2.1+)**, **Pi (`pi-subagents` & Cindy `cindy_subagent`)**, and **Antigravity (AGY)** — with **5 built-in specialist profiles** replacing scattered `~/.agent/agent/*.md` files.

---

## Why Automatic `jobs + needs` DAG + Heuristics?

Users should describe *what* they want, not micromanage how subagents are scheduled. Traditional orchestration prompts either dump every platform's syntax into one massive file or serialize work behind coarse wave barriers. `lead-orchestrator` pairs a **compact root router (`SKILL.md`)** (`H1`–`H3`) with an **event-driven Ready-Queue scheduler** (modeled on GitHub Actions `jobs.<id>.needs`):

```text
lead-orchestrator/
├── SKILL.md                    # Heuristic Root Router + Reactive needs: Scheduler
├── agents/
│   └── openai.yaml             # Explicit-invocation policy metadata
├── runtimes/                   # Tree A: Host Runtime Adapter (loads active host)
│   ├── claude-code.md          # Claude Code 2.1+ (async Agent + SendMessage + worktree)
│   ├── pi.md                   # Native Pi (js workflow DAG) & Cindy (cindy_subagent)
│   └── antigravity.md          # Antigravity (define_subagent / invoke_subagent / run_workflow)
├── modes/                      # Tree B: Execution Topology (loads selected mode)
│   ├── focus.md                # Single-seam deep loop (author -> fresh verifier -> warm delta)
│   ├── swarm.md                # Automatic jobs + needs parallel DAG, Best-of-N & audits
│   └── team.md                 # Spec-to-delivery DAG (parallel scouts -> manifest -> lanes)
└── profiles/                   # Tree C: Built-in Specialist Personas
    ├── scout.md                # Read-only domain recon & candidate jobs + needs seam mapper
    ├── investigator.md         # Hypothesis-falsifying debugger & architecture oracle
    ├── implementer.md          # Scoped DAG job author within owned_paths / worktree
    ├── verifier.md             # Fresh-context diff & build/test gatekeeper (0 source edits)
    └── integrator.md           # Post-PASS worktree merger, bundle/deploy & consent-gated SCM
```

## Core Design Principles

- **Reactive `needs:` Ready-Queue (No Full-Wave Stalls)**: All jobs with `needs: []` launch concurrently; each time any child finishes and wakes the parent, newly unblocked `PENDING` jobs launch immediately without waiting on slower sibling jobs.
- **Contract & Bundle Separation**: Thin shared contracts (`shared/types.ts`, route schemas, CSS tokens) split into fast upstream nodes (or frozen manifest specs), while shared bundle outputs and full test suites fan in downstream (`needs: [impl-*]`), keeping mid-stage implementation wide and parallel.
- **Pragmatic Isolation (`H3`)**: Unrelated uncommitted files enter `Protected Paths` instead of blocking parallel disjoint-seam workers; clean `HEAD` tasks with overlapping build contention or `[best-of-n]` use isolated Git worktrees.
- **Fresh Verification + Warm Repair**: `verifier` runs in `fresh` context without editing source files; `BLOCK` findings route back to the owning warm `implementer` (`<= 3` rounds).
- **High-Signal Declarative Prompts**: Uses clear state tables and causal rationale rather than noisy all-caps prohibitions, avoiding defensive single-worker collapse on modern reasoning models.

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

Invoke explicitly via `/lead-orchestrator` (or `/skill:lead-orchestrator` in Pi). Describe your requirements naturally—the coordinator designs the `jobs + needs` graph automatically—or pass optional bracket hints:

| Invocation | Behavior |
| --- | --- |
| `/lead-orchestrator <task>` | Auto-routes via `H1`–`H3` into `FastPath`, `Focus`, `Swarm` (`jobs + needs` DAG), or `Team` |
| `/lead-orchestrator [focus] <task>` | Single-seam implementation/investigation + fresh `verifier` loop |
| `/lead-orchestrator [swarm] <task>` | Parallel contract-split `jobs + needs` DAG + fresh `verifier` gates |
| `/lead-orchestrator [best-of-n] <task>` | Competing candidate implementations in isolated worktrees, landing the winner |
| `/lead-orchestrator [team] <task>` | Parallel `scout` recon -> clarify gate -> `TASK_MANIFEST.md` DAG -> delivery |
| `/lead-orchestrator [skip-interview] <task>` | Skip upfront clarification in `Team` mode with explicit recorded assumptions |

## Support

- **Afdian (爱发电 · WeChat / Alipay One-Time Tip)**: [https://ifdian.net/a/xrsec](https://ifdian.net/a/xrsec)

## License

[MIT](../LICENSE) © XRSec
