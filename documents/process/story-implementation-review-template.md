# Implementation Review — Round [N] — Epic [EPIC_NUMBER] / Story [STORY_NUMBER]

**Reviewer:** [role]
**Date:** [date]
**Story file:** [path to story-details.md]
**Sample path:** [path to review-implementation/round=N/sample/]

---

## Confirmed Clean

> Items confirmed correct in prior rounds. Closed — do not recur. Leave blank on round 1.

| # | Item | Round confirmed |
|---|------|----------------|
| | | |

---

## New Findings

> Items raised for the first time this round. Each must cite the acceptance criterion or spec section being violated.
> Write "None" if nothing is wrong.

| # | Finding | AC / Spec reference | Severity |
|---|---------|-------------------|----------|
| | | | blocking / advisory |

---

## Carry-Forward

> Items from prior rounds not yet resolved. Leave blank on round 1.

| # | Item | First raised (round) | Current status |
|---|------|---------------------|---------------|
| | | | |

---

## Overall Status

- [ ] **APPROVED** — zero new findings, zero carry-forward. Story implementation is complete.
- [ ] **FINDINGS RAISED** — see above. Re-submit after addressing all findings.

---

## Reviewer Pre-Approval Gate

> Complete before marking APPROVED. All items must be confirmed.

- [ ] All 9 checks from the review instructions explicitly worked through this round
- [ ] Round isolation confirmed — sample at `review-implementation/round=[N]/sample/` is direct production writer output, not hand-assembled or copied from a prior round
- [ ] Part B oracle values present in the story spec (developer filled)
- [ ] Part C desk plausibility sign-off present in the story spec (desk signed)
