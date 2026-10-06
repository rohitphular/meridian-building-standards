# Epic Review — Round [N] — Epic [EPIC_NUMBER]

**Reviewer:** [role]
**Date:** [date]
**Epic file:** [path to epic-details.md]

---

## Checklist Sweep

> Work through all 10 checks from the review instructions before writing findings. Mark Pass or Gap for each. Every finding in the New Findings table must reference a check number from this sweep.

| # | Check | Pass / Gap found |
|---|-------|-----------------|
| 1 | No placeholders — no `[PLACEHOLDER]` token remains | |
| 2 | Formulas explicit — every formula written out, variants named and rejected | |
| 3 | Enum completeness — all values, both directions, all boundary conditions | |
| 4 | Temporal flags complete — set condition, exact clear condition, union vs per-variant | |
| 5 | Output schema present — row inventory, per-variant vs combined decided, delayed writes identified | |
| 6 | Constants table complete — every threshold in flag/classification is a named row | |
| 7 | End-to-end scenario — Part A has specific numeric inputs and concrete acceptance bounds | |
| 8 | Suppression paths covered — second scenario bar provided for every unreachable suppression path | |
| 9 | Oracle values code-computed — transcendental values have a captured code-run, not hand-approximated | |
| 10 | Pre-submission checklist — fully ticked in the epic creation instructions | |

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

- [ ] **APPROVED** — zero new findings, zero carry-forward. Epic is ready for story writing.
- [ ] **FINDINGS RAISED** — see above. Re-submit after addressing all findings.
