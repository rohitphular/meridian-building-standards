# Story Implementation Review Instructions

The desk reviews the developer's implementation sample against the story acceptance criteria. This is an output correctness check, not a code review.

---

## What to Check

1. **Acceptance criteria** — Go through every criterion in the story's Acceptance Criteria section. For each, find the corresponding row(s) in the sample and verify it passes.
2. **Row inventory** — The sample contains exactly the row types listed in the story's Row Inventory. No extra row types, no missing row types.
3. **session_summary present** — The sample includes a session_summary row. If the slice ends before session close, the row carries the mid-session state with a comment.
4. **Oracle arithmetic** — Part B values in the story spec match the sample output within stated tolerances.
5. **Null/fallback** — Null conditions are correctly applied. Zero is not used where None is required.
6. **Output format** — Compact NDJSON. Every line parseable independently. Correct rounding per field.
7. **Timestamps** — All timestamps are timezone-aware with UTC offset.
8. **Test run log** — If unit tests are required, the pytest output ("N passed") is attached in this same round.
9. **Round isolation** — The sample file is the direct output from the production writer, not hand-assembled or merged with another story's output.

---

## Reviewer Conduct Rules

**Rule 1 — No finding without an acceptance criterion reference.**
Every finding must cite the acceptance criterion or spec section being violated. A deviation from something not in the spec is a spec gap — raise it as a spec update for the next round, not as a finding against the implementation.

**Rule 2 — Prescriptions must cite a spec section.**
A prescription to change the implementation must name the acceptance criterion or formula it violates.

**Rule 3 — Confirmed clean items do not recur.**
Once an item is confirmed correct in a prior round, it is closed. Do not re-raise it.

---

## Review Round Format

Every round uses this exact three-section structure:

1. **Confirmed Clean** — items closed in prior rounds. Do not recur.
2. **New Findings** — items raised for the first time this round. Each must include an acceptance criterion or spec reference.
3. **Carry-Forward** — items from prior rounds not yet resolved.

---

10. **Part C signed** — Part B oracle values are present in the story spec (developer filled) and Part C desk plausibility sign-off is signed before APPROVED status is granted.

---

## Approval

The implementation is approved when a round produces zero new findings, zero carry-forward items, and the Reviewer Pre-Approval Gate in the template is fully checked. Mark overall status as `APPROVED`.
