# Epic Review Instructions

The developer reviews the epic spec for completeness, clarity, and implementability before any story work begins. This is a spec readiness check — not a technical debate.

---

## What to Check

1. **No placeholders** — No `[PLACEHOLDER]` token remains. Every section is filled.
2. **Formulas explicit** — Every formula written out. No formula left inferable from an example result alone. Non-obvious variants named and rejected.
3. **Enum completeness** — Every categorical field lists all values, both directions of any symmetric axis, all boundary conditions with explicit direction (`>`, `>=`, `<`, `<=`).
4. **Temporal flags complete** — Every session flag states: (a) set condition, (b) exact clear condition, (c) union vs per-variant if multiple variants exist.
5. **Output schema present** — Row inventory table present. Per-variant vs combined decided. Delayed writes identified with release condition.
6. **Constants table complete** — Every threshold used in a flag or classification is a named row in the constants table. No in-prose-only numbers.
7. **End-to-end scenario** — Part A has specific numeric inputs (not ranges). Acceptance bounds are concrete ranges per field.
8. **Suppression paths covered** — Every binding suppression path the headline scenario cannot exercise has a second named scenario bar.
9. **Oracle values code-computed** — Any transcendental oracle value (exp(), log()) has a captured code-run backing, not a hand approximation.
10. **Pre-submission checklist completed** — The checklist in the epic creation instructions is fully ticked.

---

## Reviewer Conduct Rules

**Rule 1 — No finding without a spec reference.**
Every finding must cite the section and rule being violated. If the deviation is from something not stated in the spec, that is a spec gap — raise it as a spec update request, not a finding against the implementation.

**Rule 2 — Prescriptions must cite a spec section.**
A prescription to change the spec must name the section that is incomplete or incorrect.

**Rule 3 — Reviewer may not add scope.**
A review round is not an opportunity to add new requirements. New features belong in a new epic.

---

## Review Round Format

Every round uses this exact three-section structure:

1. **Confirmed Clean** — items closed in prior rounds. Do not recur.
2. **New Findings** — items raised for the first time this round. Each must include a spec section reference.
3. **Carry-Forward** — items from prior rounds not yet resolved.

---

## Approval

The epic is approved when a round produces zero new findings and zero carry-forward items. Mark overall status as `APPROVED`.

---

## Round Limit

If an epic has not reached `APPROVED` status after 5 review rounds, the scrum master conducts an arbitration session before round 6 begins. Rounds beyond 5 indicate a structural gap in the epic spec or a process failure — not a content debate. The scrum master determines whether to continue, restructure the epic, or escalate.
