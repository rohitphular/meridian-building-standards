# Story Implementation Instructions

The developer implements the approved story spec and produces a sample output for review. Do not begin implementation until the story spec is approved.

---

## Before Writing Any Code

1. Read the full story spec. Confirm the Pre-Implementation Gate in the spec is cleared. If any row in the gate table shows INCOMPLETE, return the spec to the desk listing the incomplete items. Do not write any code until a corrected spec is received.
2. Read the parent epic details for shared conventions — model parameters, OI convention, output format, NSE constants. These are inherited; do not redefine them.
3. If any section of the spec is ambiguous or incomplete, return it to the desk before starting. Do not make assumptions.

---

## Producing the Sample

The sample is a real output from your production writer — not a hand-assembled file.

1. Implement the writer against the story spec.
2. Run the writer against the scenario inputs from the story's Verified Example (Part A).
3. Save the direct output to the sample folder. Do not edit the file after writing it.

---

## Pre-Submission Checklist

Run this before every submission — round 1 and every subsequent round.

- [ ] **Acceptance criteria row-by-row** — read every criterion, find the corresponding row in the sample, verify it passes.
- [ ] **Oracle arithmetic dual-verified** — every numeric value in the response is computed by two independent methods (hand/REPL + code output). No value written without both agreeing.
- [ ] **session_summary present** — sample contains a session_summary row. If the slice ends before session close, the row carries mid-session state with a comment.
- [ ] **Temporal-flag trace produced** — for every flag with a temporal lifecycle, a bar-by-bar trace table was produced and the sample verified against it. Mark N/A if no temporal flags.
- [ ] **Test run log attached** — if this story includes unit tests, the literal pytest output ("N passed in Xs") is attached in this same submission. Mark N/A only if this story has zero unit tests.
- [ ] **Round-isolation verified** — sample file at round=N/sample/ is the direct output from the production writer, not assembled or copied from a prior round.

---

## If Any Checklist Item Is Blocked

If a gate item cannot be executed (Bash denied, test runner unavailable), raise it explicitly as a named process blocker in the submission. Do not substitute a workaround and continue silently.
