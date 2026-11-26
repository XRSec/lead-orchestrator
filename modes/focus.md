# MODE TREE: FOCUS (SINGLE-SEAM DEEP LOOP)

Use for single-module features, targeted bugfixes, or deep root-cause investigations where splitting across multiple writers would create artificial coordination overhead.

## 1. Execution Tree
```text
Coordinator (0 Pre-Work)
 ├── [Investigation Task] -> Investigator (fresh/fork) ──> Verified Root Cause / Design Verdict
 └── [Coding Task]        -> Implementer (fresh) ──> Verifier (fresh, 0 edits)
                                  ▲                         │
                                  └─── Warm Delta (<= 3) ───┘ (on Verdict: BLOCK)
```

## 2. Step-by-Step Protocol
1. **Zero Pre-Work Dispatch**: Do NOT grep/read repo files in parent thread first. Compose a Cold-Start Brief immediately.
2. **Cold-Start Brief Contract**:
   - `<role_profile>`: Body of `profiles/investigator.md` or `profiles/implementer.md`.
   - `User Request (Verbatim)`: Exact user message, unedited.
   - `Prior Context`: Essential facts from earlier turns ONLY if latest message is ambiguous alone (omit on Turn 1).
   - `Authorized Scope`: Target seam (or `DISCOVER_AND_CONTAIN` if single-seam path is obvious to worker; if highly uncertain, run a 1-shot `scout` first).
   - `Done Criteria`: Observable behavior + local test/typecheck/lint verification.
3. **Adversarial Gate (`verifier`)**:
   - For coding changes, dispatch `profiles/verifier.md` in **`fresh` context** with the verbatim user request, changed files, and test commands.
   - Spot-check critical requirements: err on spawning a continuation round if any edge case, test, or requirement is partial.
4. **Bounded Warm Repair (`MAX_REPAIR_ROUNDS = 3`)**:
   - On `Verdict: BLOCK`, send a **Delta Brief** (`Completed So Far` + `P0/P1/P2 Findings with path:line` + `Validation Commands`) to the **same warm `implementer`** via host continuation primitive.
   - Re-verify after fix. If Round 3 still fails, halt and escalate exact diagnosis and diff state in `zh-CN` to the user.
