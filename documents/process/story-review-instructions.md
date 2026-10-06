# Story Details Review Instructions

The developer reviews the story spec for completeness and implementability before writing any code. This is a spec readiness check — not a technical debate.

---

## What to Check

### INVEST Quality Gate (check before spec completeness)

1. **Independent** — This story can be implemented without waiting for another in-flight story. Any blocking dependency must name a completed story, not an in-progress one.
2. **Small** — The story is implementable in one sprint. If the row inventory contains more than 4 distinct row_types or the dependency chain spans more than 3 prior stories, flag for scope review.
3. **Testable** — Every acceptance criterion is binary (pass or fail, checkable with a number or boolean assertion). If any criterion uses "looks correct," "seems reasonable," or similar — the story is not testable and must be returned.

### Spec Completeness (check after INVEST)

4. **No placeholders** — No `[PLACEHOLDER]` token remains. Every section is filled.
5. **Input contract complete** — Every input field has: type, units/scale, valid range, on-invalid behaviour.
6. **Output contract complete** — Every output field has: type, units/scale/sign, nullable condition (exact), example value.
7. **Zero vs None distinguished** — For every nullable field, zero and None are explicitly distinguished.
8. **Verified example — Part A** — Specific numeric inputs (not illustrative ranges). Acceptance bounds are concrete ranges per field.
9. **Math model specified** — Variant named exactly. Parameters stated with values. Key identity stated with numeric substitution. Day-count stated.
10. **Null/fallback exhaustive** — Every condition that produces None is listed. Cascading None rules stated.
11. **Row inventory present** — Per row_type: count per session, key fields, write trigger, held vs immediate.
12. **Enum completeness** — Every categorical field lists all values, both directions, all boundary conditions.
13. **Acceptance criteria binary** — Every criterion is checkable with a number or boolean. No "looks correct."
14. **Consumer story check** — If this story reads another story's output, an explicit file-inventory acceptance criterion is present (grep-checkable row-type check).
15. **Pre-submission checklist completed** — The checklist in the story creation instructions is fully ticked.

---

## Reviewer Conduct Rules

**Rule 1 — No finding without a spec reference.**
Every finding must cite the section and rule being violated. Deviations from things not stated in the spec are spec gaps — raise as spec update requests, not implementation findings.

**Rule 2 — Prescriptions must cite a spec section.**
A prescription to change the spec must name the section that is incomplete or incorrect.

**Rule 3 — Reviewer may not add scope.**
New requirements belong in a new story or epic, not in a review round.

---

## Review Round Format

Every round uses this exact three-section structure:

1. **Confirmed Clean** — items closed in prior rounds. Do not recur.
2. **New Findings** — items raised for the first time this round. Each must include a spec section reference.
3. **Carry-Forward** — items from prior rounds not yet resolved.

---

## Approval

The story spec is approved when a round produces zero new findings and zero carry-forward items, and all 15 checks in the sweep are marked Pass. Mark overall status as `APPROVED`.
