# Story Details Review — Round [N] — Epic [EPIC_NUMBER] / Story [STORY_NUMBER]

**Reviewer:** [role]
**Date:** [date]
**Story file:** [path to story-details.md]

---

## Checklist Sweep

> Work through all 15 checks from the review instructions before writing findings. Mark Pass or Gap for each. Every finding in the New Findings table must reference a check number from this sweep.

| # | Check | Pass / Gap found |
|---|-------|-----------------|
| **INVEST** | | |
| 1 | Independent — no blocking dependency on an in-flight story | |
| 2 | Small — implementable in one sprint; ≤4 row_types, ≤3 dependency hops | |
| 3 | Testable — every acceptance criterion is binary (number or boolean) | |
| **Spec Completeness** | | |
| 4 | No placeholders — no `[PLACEHOLDER]` token remains | |
| 5 | Input contract complete — every field has type, units/scale, valid range, on-invalid behaviour | |
| 6 | Output contract complete — every field has type, units/scale/sign, exact nullable condition, example value | |
| 7 | Zero vs None distinguished — explicitly distinguished for every nullable field | |
| 8 | Verified example Part A — specific numeric inputs, concrete acceptance bounds per field | |
| 9 | Math model specified — variant named exactly, parameters stated, key identity with numeric substitution, day-count stated | |
| 10 | Null/fallback exhaustive — every condition producing None listed, cascading None rules stated | |
| 11 | Row inventory present — per row_type: count per session, key fields, write trigger, held vs immediate | |
| 12 | Enum completeness — all values, both directions, all boundary conditions | |
| 13 | Acceptance criteria binary — every criterion checkable with a number or boolean | |
| 14 | Consumer story check — if this story reads another story's output, file-inventory AC is present (grep-checkable) | |
| 15 | Pre-submission checklist — fully ticked in the story creation instructions | |

---

## Confirmed Clean

> Items confirmed correct in prior rounds. Closed — do not recur. Leave blank on round 1.

| # | Item | Round confirmed |
|---|------|----------------|
| | | |

---

## New Findings

> Items raised for the first time this round. Each must cite the spec section being violated.
> Write "None" if nothing is wrong.

| # | Finding | Spec section reference | Severity |
|---|---------|----------------------|----------|
| | | | blocking / advisory |

---

## Carry-Forward

> Items from prior rounds not yet resolved. Leave blank on round 1.

| # | Item | First raised (round) | Current status |
|---|------|---------------------|---------------|
| | | | |

---

## Overall Status

- [ ] **APPROVED** — zero new findings, zero carry-forward. Story is ready for implementation.
- [ ] **FINDINGS RAISED** — see above. Re-submit after addressing all findings.
