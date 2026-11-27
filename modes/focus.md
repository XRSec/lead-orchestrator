# MODE TREE: FOCUS (SINGLE-SEAM DEEP LOOP)

Use for single-module features, targeted bugfixes, or focused root-cause investigations where splitting across multiple writers would create unnecessary handoff overhead.
*(If the user's message bundles `>= 2` distinct items—such as a backend fix + UI polish + log refactor—switch to `modes/swarm.md` instead so independent seams run as a parallel `jobs + needs` DAG.)*

## 1. Execution Graph
```text
Coordinator (Minimal Triage)
 ├── [Investigation] -> [investigator] (fresh/fork) ──> Proven Root Cause / Design Verdict
 └── [Implementation] -> [implementer] (fresh, needs: []) ──> [verifier] (fresh, 0 edits, needs: [implementer])
                                ▲                                       │
                                └──────── Warm Delta (<= 3) ────────────┘ (on Verdict: BLOCK)
```

## 2. Execution Flow
1. **Immediate Dispatch**: Avoid reading source trees serially in the parent session. Compose a Cold-Start Brief and launch the worker right away (if target file paths are genuinely opaque, run one fast read-only `scout` first).
2. **Cold-Start Brief**:
   - `<role_profile>`: Body of `profiles/investigator.md` or `profiles/implementer.md`.
   - `User Request (Verbatim)`: Unedited user prompt.
   - `Prior Context`: Relevant facts from earlier turns when needed for disambiguation.
   - `Owned Paths` & `Protected Paths`: Target module boundary + any unrelated uncommitted files to leave untouched.
   - `Done Criteria`: Observable behavior + targeted test/typecheck/lint commands.
3. **Fresh Verification (`needs: [implementer]`)**:
   - Once `implementer` finishes, launch `profiles/verifier.md` in `fresh` context (`0` file edits) with the verbatim request, changed coordinates, and validation commands.
4. **Warm Repair (`<= 3` Rounds)**:
   - On `Verdict: BLOCK`, send a concise **Delta Brief** (`Completed So Far` + `P0/P1/P2 Findings with path:line` + `Validation Commands`) to the **same warm `implementer`**.
   - Re-verify after repair; if Round 3 still blocks, report the exact diagnosis and current diff state in `zh-CN` to the user.
