# Implementation Review Response Instructions

The developer responds to each finding raised in the implementation review. Every finding gets a resolution. No finding is left unaddressed.

---

## For Each Finding

**Fixed** — describe what was changed in the code and what changed in the sample output. Be specific: field name, bar, old value, new value.

**Rejected** — state why the finding is incorrect. Cite the acceptance criterion or spec section that supports the current output. A rejection without a cited reason will be re-raised.

**Deferred** — state why it cannot be resolved now and when it will be. Deferred items become Carry-Forward in the next review round.

---

## Rules

- Address every New Finding and every Carry-Forward from the review feedback. Do not skip items.
- Fix the code and regenerate the sample before writing the response. The response describes changes already made.
- Re-run the full pre-submission checklist after making changes. Do not submit a response without re-checking all items.
- Submit an updated sample together with the response — do not submit the response without an updated sample if any finding required a code change. Save the updated sample to `review-implementation/round={NEXT_ROUND}/sample/` where `NEXT_ROUND = OPERATION_ROUND + 1`. This pre-creates the next review round's sample folder so the reviewer can run message 10 immediately. Do not overwrite the current round's original sample — each round has its own path.
- Do not respond with "will fix" — fix it first, then respond.

---

## Disputed Rejections

If the reviewer disagrees with a rejection in the subsequent round, they re-raise it in Carry-Forward as "Disputed — rejection not accepted" with a counter-argument. After two consecutive rounds of dispute on the same finding, the scrum master arbitrates. The arbitration decision is final for that story.
