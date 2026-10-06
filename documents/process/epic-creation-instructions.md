# How to Write an Epic Requirement: Instructions for the Trading Desk

This document explains every section of the epic template and why it exists.
Read it once before filling in your first epic. You do not need to reread it
for every epic — the template itself is self-contained once you understand the
purpose behind each section.

The rules here are grounded entirely in what went wrong during T0, where one
shippable capability (the capture foundation) was broken into eight tasks and
required nine rounds of feedback. The cost of those rounds was real: rework on
working code, a three-round debate over a single missing line in the math model
specification, and a round-8 format failure that should have been caught before
any code was written.

---

## 1. What an Epic Is and How It Differs from a Task

An **epic** is one shippable capability: a coherent bundle of functionality that
produces a result the desk can use. Examples: "capture foundation" (T0), "GEX
signal engine," "IV rank and percentile," "basis signal."

A **task** (or story) is one implementable unit within an epic: one Python module,
one pull request, one developer holding it in their head at once. Examples:
"unified Pydantic schemas" (T0-001), "BSM IV solver" (T0-002), "options snapshot
writer" (T0-007).

The distinction matters because an epic carries shared conventions that every
task in it inherits. If the BSM parameters are wrong in the epic, all eight tasks
are wrong. If they are stated correctly once in the epic's shared conventions
section, no task needs to repeat them, and there is one authoritative place to
update them.

T0 had no shared conventions section. The parameters r = 0.065 and q = 0.0125
were stated in T0-002 (the IV solver task) and silently assumed everywhere else.
When Round 4 questioned whether the correct rate was used, there was no single
document to point to. The three-round debate (R4, R5, R6) that followed would
have been a one-line reference check if the epic had carried the parameters
centrally.

**One-line description test for tasks.** When decomposing an epic into tasks,
apply this test to each task: can you describe what it builds in one sentence?
If not, the task is too large. "Build the options chain capture, including the
IV solver, the Greeks engine, the snapshot writer, the schema, and the file
writer" fails the test — that was the shape of T0 before decomposition, and it
produced exactly the ambiguities described here.

---

## 2. The Shared Conventions Section

This is the most important section in the epic template. It is the section
T0 was missing entirely.

**What belongs here:**

- Model parameters: r (risk-free rate), q (dividend yield), model variant name
- Day-count convention: act/365 (calendar days) vs act/252 (trading days) —
  never assume; state it
- OI convention: exchange units (always a multiple of lot_size), not lots
- Output format: compact NDJSON, one record per line, no whitespace
- Timezone rule: all timestamps IST +05:30, timezone-aware, no naive datetimes
- NSE market structure constants: expiry weekday, lot sizes, index ATM band
- Field naming conventions: snake_case, valid Python identifiers
- Signal computation conventions: all formulas, all enum taxonomies (complete),
  all temporal-flag definitions (T1 additions — see below)

**Signal Computation Conventions and Formula Explicitness (T1 lesson)**

T1 introduced a Signal Computation Conventions section. It was the single biggest
gap-preventer at the computation level. Three rules for what it must contain:

1. **Write every formula out explicitly.** Do not leave a formula inferable from
   an example arithmetic result. "σ = stdev(…) / √t" and "σ = stdev(…)" are
   different computations. The T1 σ formula pinned the `/t` double-divide as
   explicitly rejected — a note that cannot be inferred from a numeric example.
   Every formula that has a non-obvious variant must name the variant and reject
   the alternatives by name.

2. **Complete every enum taxonomy in the first draft.** Every categorical field
   must list all values, both directions of any symmetric axis, and every boundary
   condition. The T1 `gap_type` downside (`partial_gap_down` / `full_gap_down`)
   was missing from the first draft because the upside was specified first. There
   is no domain subtlety on the down side that the up side lacks — it is a literal
   mirror. Rule: if an enum has an up/positive direction, the down/negative
   direction must appear in the same paragraph. If it does not, the spec is not
   complete. Same for boundary conditions: `>`, `>=`, `<`, `<=` must all be stated.
   "boundary toward centre" is not a spec.

3. **Define temporal-status flags completely.** Any flag that turns on and off over
   a session must specify, in one place: (a) the condition that sets it, (b) the
   exact timestamp it clears, and (c) if multiple variants exist, whether the flag
   fires on the union (any variant) or per-variant. The T1 `or_provisional` flag
   had (a) specified but (b) and (c) were left implicit. The result was an
   implementation round for a flag that was otherwise correctly specified. Rule:
   a flag with a temporal lifecycle requires all three components. One missing
   component = one round.

**NSE Constants Table Completeness (T1 lesson)**

Every threshold used in a flag classification or suppression condition must appear
as a named row in the NSE Market Structure Constants table. In-prose-only numbers
are gaps. The T1 OR-width thresholds (`or_narrow` < 0.10%, `or_wide` > 0.60%) and
gap-flat threshold (0.15%) needed a review round to add to the table because they
appeared in the Signal Computation Conventions text but not in the constants table.
Rule: if a number is used in a flag condition, it is a constant. Constants live in
the table, referenced by name, not repeated inline.

**Why each of these specifically:**

The r and q parameters tripped up T0-002 through T0-007 because nothing stated
them centrally. Round 4 questioned the rate; Round 5 applied a wrong correction;
Round 6 had to rescind the Round-5 correction. Three rounds over two parameter
values that should have been a one-line reference.

The OI unit convention tripped Rounds 1 and 2. "NSE OI is in exchange units,
always a multiple of lot_size" is obvious to a practitioner but not obvious to
a developer. The developer used lot-level OI, which is 75x wrong for NIFTY.

The output format tripped Round 8. "NDJSON" does not specify compact vs
pretty-printed. The options file was serialised as multi-line pretty-printed
JSON, which broke every standard NDJSON parser. A one-line literal example in
the epic would have prevented this.

**What does not belong here:**

Task-specific parameters (a particular strike's OI threshold, a solver
convergence tolerance) belong in the individual task spec, not here. The
shared conventions section should contain only what is inherited by every
task in the epic without modification.

---

## 3. Task Decomposition Rules

**One PR per task.** A task boundary is also a code review boundary. If two
pieces of logic cannot be reviewed in one PR without the reviewer losing context,
they are two tasks.

**Dependency ordering.** List dependencies explicitly. T0-007 depended on
T0-001 (schemas), T0-002 (IV solver), T0-003 (Greeks engine), and T0-004
(file writer). Without explicit ordering, a developer might start T0-007 before
T0-002 is finished and have to redo work when the IV solver interface changes.

**One-line description test.** Every task in the task list table should have a
"What it builds" column entry that fits in one sentence. If you cannot write
that sentence, the task scope is not yet defined.

**Avoid the integration trap.** T0-007 was the integration point for all prior
tasks. Integration tasks are always the most complex and the most likely to surface
ambiguities that were invisible when the individual components were specified. Write
integration tasks last; review them most carefully; make sure the input/output
contracts of the components are fully nailed before the integration task is written.

---

## 3b. Output Schema and Write Model (T1 lesson — multi-writer epics)

T1-003 had four spec rounds entirely about output schema and write model, on top of
computation that was correct from round 1. The computation was specified well. The
schema — how many rows of each type per session, what each carries, when each is
written — was left to be discovered in review. That is a desk failure that the epic
template now explicitly prevents.

For every epic with more than one writer, the epic's Output Format section must
answer, per writer, per row type:

1. **Row count per session:** "E4 emits exactly N `opening_range` rows per session"
   is a spec. "E4 emits opening range rows" is not.

2. **Key field set per row type:** Which fields does each row type carry that are
   absent from other row types? This is the only way to catch schema collisions
   between row types before implementation.

3. **Write trigger:** Immediate (on the bar that completes the condition) or
   held (appended at session end, or held until some future event). The T1-003
   `failed_breakout` and `no_breakout` rows are held to session end — a delayed
   write. The spec must say "written at 15:30" or "appended at session close," not
   "emitted when condition fires" without specifying when that is.

4. **Per-variant vs combined:** If the epic has variants (two OR windows, two expiry
   series), state whether each row type is emitted once per variant or once combined.
   This is an epic-level schema decision. It must be in the epic Output Format section
   before the first task review round. The T1 `opening_range` row-count question
   (one combined row vs one per variant) surfaced in T1-003 task spec round 2 —
   two rounds in — because the epic had not resolved it. Per the T0 conclusion
   process rule, a decision that changes row count belongs in the epic, not in task
   review.

**The row inventory goes into the epic's Output Format section as a table.** One row
per (writer, row_type) pair, with columns: writer, row_type, count per session,
key unique fields, write trigger, per-variant or combined. If this table is absent
from the epic when it reaches the developer, send it back.

---

## 4. The End-to-End Scenario

The end-to-end scenario is the single change that would have reduced T0's nine
rounds by the most. It is the epic-level equivalent of the "verified example"
section in individual task specs.

**What it must contain:**

- Specific input values (not illustrative ranges). "NIFTY at approximately 24000"
  is not a scenario. "S = 24052.75, K = 24100, T = 15/365, futures LTP = 24118.40"
  is a scenario.
- Acceptance bounds per task, expressed as numeric ranges. "delta near 0.5" is not
  a bound. "delta: 0.507 ± 0.002" is a bound.
- Cross-task parity checks: relationships between outputs from different tasks that
  must hold simultaneously.

**The three-role authorship model (the G2 fix):**

The T0 retrospective identified the absence of verified example output as the single
biggest source of rework. But the solution is not for the desk to hand-compute
Merton/GK to nine decimal places — that would create circular verification, since
the developer would check their code against the desk's arithmetic, and neither would
catch a shared misunderstanding of the model.

The correct authorship split is:

1. **Desk writes the scenario and the acceptance bounds.** "ATM NIFTY, 15 DTE,
   IV approximately 9.4%: call delta should be near 0.50–0.51. CE/PE vega should
   be equal at the same strike. Theta should be more negative for the CE than the
   PE. CE_delta plus absolute PE_delta should be approximately 0.9995, not 1.0."
   The desk does not compute exact values — that is the developer's job.

2. **An independent oracle produces the exact digits.** QuantLib, Haug's option
   pricing tables, or scipy.stats are the source of truth. The developer runs the
   oracle against the scenario inputs and treats its output as the acceptance
   benchmark. Example: QuantLib gives delta = 0.5076229027 for the scenario above;
   the developer's engine must match this within tolerance.

3. **Desk validates domain plausibility of the oracle result.** Is the sign correct?
   Is the magnitude in the right range for a liquid ATM option? Does the theta
   asymmetry direction match the NSE rate environment (r > q means call decays
   faster — this is always true on NSE)? This is a qualitative check, not a
   recomputation.

This division prevents both failure modes: the desk hand-running the wrong formula
(which happened implicitly in R4–R6), and the developer testing against their own
output (which is circular).

**Oracle value authorship for transcendentals (T2 lesson):**

Any oracle value that involves a transcendental function — exp(), log(), or any
non-rational arithmetic — must be produced by a captured code run at epic drafting
time, not hand-approximated. Record the unrounded value so the rounding boundary
is visible. In T2, the `fair_value_basis` value was hand-approximated and landed on
the wrong side of a two-decimal rounding boundary, requiring an 11-location correction
table spread across rounds 3–5. The unrounded value (`54.189`) would have resolved the
rounding unambiguously from the first round.

Rule: if a Part A/B oracle value involves exp(), log(), or more than three arithmetic
steps, run it in code before writing it. Never approximate a transcendental.

**Second scenario bar for suppression paths (T2 lesson):**

If the epic's headline Part A scenario cannot exercise a binding suppression rule —
for example, a DTE=0 whole-row suppression or an expiry-day afternoon cross-read
suppression — a second named scenario bar is required. A suppression path exercised
only by unit tests has no desk-signed expected output. The desk cannot validate what
it has never seen, and a developer-only assertion is not a desk approval gate.

Format: "Scenario bar 2 — [condition name]: [inputs], expected output: [desk-signed
expected values]." Provide as many secondary bars as there are unreachable suppression
paths.

**Cross-task parity checks:**

These are the relationships between tasks that must hold simultaneously at the
end of the epic. They catch errors that individual task tests miss because they
require two tasks' outputs to be compared.

Example for a Greek-engine epic:
- CE_delta + |PE_delta| = exp(-q * T) at tolerance ±0.002 (cross-leg delta parity,
  approximate because CE and PE carry per-leg IVs from their own market prices)
- |IV_CE - IV_PE| at the same strike < 5 basis points (same-strike IV parity)
- theta_CE < theta_PE at the same strike, always (call decays faster on NSE,
  where r > q always holds)
- All OI values for every record divisible by lot_size (OI unit constraint)

The tolerance for the cross-leg delta check is approximately ±0.002, not zero.
When CE and PE carry different per-leg IVs (which they always do when solved from
their own market prices), the identity is approximate. The tolerance widens on
high-volatility days or for wing strikes. Do not fail this check with a zero-
tolerance assertion.

---

## 5. Additional Requirements for Aggregation Epics (G8–G11)

When the epic aggregates across strikes, expiries, or time (GEX, DAOI, PCR,
max pain, IV rank, basis signal), four additional specification requirements
apply that do not arise in per-record computation epics like T0.

**G8 — Aggregation scope (the aggregation equivalent of G3).**

Every aggregation indicator sums or weights across the chain. Two implementations
can both be correct per-task yet produce silently different numbers if the
aggregation scope is not specified. The epic must state:

- Which strikes are in scope: full chain, N strikes around ATM, moneyness band,
  strikes with OI above a threshold.
- How wing strikes are handled: wing strikes often have null IV, zero OI, or
  unreliable prices. Are they included, excluded, or weighted down?
- How near and next expiry are combined: weighted by DTE, separate outputs,
  near-only.

Example of an underspecified aggregation: "compute PCR across all strikes."
Does this include strikes with OI below the threshold? Does it include expiries
beyond the near month? What happens when 30% of strikes have null IV?

**G9 — Snapshot simultaneity and as-of semantics.**

Cross-instrument signals combine spot, futures, and options data that may not
share the same timestamp. The epic must state:

- How stale may an OI snapshot be before the signal is suppressed?
- NSE OI is EOD-authoritative: the exchange publishes final OI after market
  close, and the intraday OI visible during trading is an approximation. State
  which is used and when.
- For signals combining multiple instruments, state whether they must share the
  exact same timestamp or whether a defined staleness window is acceptable
  (for example, spot and futures within the same 5-minute bar is acceptable;
  options OI from a prior session is acceptable for signals that explicitly
  use EOD OI).

**G10 — Suppress-when condition.**

Every signal output must carry an explicit suppress-when condition. PCR extremes
are near-meaningless on weekly expiry day. Max pain is only a reliable attractor
in the final 90 minutes and only when OI is concentrated at a small number of
strikes. OI-quadrant signals are noise in range-bound markets.

The suppress-when condition is a domain decision: only the desk can say when a
signal is unreliable enough to exclude from downstream consumption. The developer
cannot default it. Every aggregation signal spec must include:

- The field name that carries the reliability flag
- The condition (as a boolean expression) under which the signal is suppressed
- The desk's reasoning (one sentence)

**G11 — Null propagation through aggregation.**

The per-record rule "null IV implies null Greeks" (from T0) is correct but
insufficient for aggregates. When 20 of 40 strikes have null IV, a PCR computed
over the remaining 20 is not the same signal as one computed over 40. The
partial-coverage aggregate looks valid but is computed on a different subset
each interval, making time-series comparison meaningless.

The epic must specify one of three treatments:

a. **Degrade gracefully with a coverage fraction field.** The aggregate is
   computed over available records, and a `coverage_fraction` field is written
   alongside it. Downstream consumers can filter on coverage_fraction.
b. **Suppress below a minimum-coverage threshold.** If fewer than N% of strikes
   have valid data, the aggregate is written as None. N is a domain decision.
c. **Flag partial coverage.** The aggregate is computed and a `partial_coverage`
   boolean is set when any input was null.

The choice between a, b, and c depends on how the desk uses the signal.
The default of "silently compute the aggregate over available records" is wrong —
it hides coverage variation that may be a signal itself (a sudden drop in IV
coverage often precedes a volatility event).

---

## 6. Pre-Submission Checklist for Epics

Run through this before handing the epic spec to the developer.

### Shared conventions section

- [ ] Model variant named exactly: "BSM-Merton, b = r - q" for equity index
      options. Not "BSM." Not "GK" (that is the FX variant). Not "Merton/GK."
- [ ] r stated with value and source: r = 0.065 (NSE 91-day T-bill)
- [ ] q stated with value and source: q = 0.0125 (NIFTY trailing dividend yield)
- [ ] Day-count stated: act/365, calendar days, T = DTE/365
- [ ] OI convention stated: exchange units, always a multiple of lot_size
- [ ] Output format stated: compact NDJSON, one record per line, no whitespace
- [ ] Timezone stated: IST +05:30, no naive datetimes
- [ ] NSE market structure constants present (expiry weekday, lot sizes, ATM band)

### Task decomposition

- [ ] Every task passes the one-line description test
- [ ] Dependency ordering is explicit (task IDs, not just task names)
- [ ] Integration tasks are listed last in the dependency chain
- [ ] No task combines schema definition + computation + file writing
      (those are three separate concerns requiring separate review)

### End-to-end scenario

- [ ] Scenario uses specific numeric inputs, not illustrative ranges
- [ ] Acceptance bounds are numeric ranges per task output field
- [ ] Cross-task parity checks are listed with tolerance bands
- [ ] The three-role split is followed: desk provides bounds, oracle provides
      exact digits, desk validates domain plausibility
- [ ] For same-strike CE/PE checks: tolerance is ±0.002 for the delta parity
      check, not zero (per-leg IVs cause approximate identities)
- [ ] All transcendental oracle values (any exp(), log(), or non-rational formula)
      produced by a captured code run at draft time; unrounded value recorded
      (T2 lesson — hand-approximated transcendentals on rounding boundaries require
      multi-location correction tables discovered mid-implementation)
- [ ] Second named scenario bar provided for every binding suppression path the
      headline Part A scenario cannot exercise (T2 lesson — unit tests alone are
      not a desk-signed approval gate for suppression behaviour)

### Aggregation epics only (G8–G11)

- [ ] Aggregation scope stated: which strikes, which expiries, how wings handled
- [ ] As-of semantics stated: staleness window, NSE EOD OI vs intraday OI
- [ ] Suppress-when condition stated per signal field
- [ ] Null propagation treatment chosen: degrade / suppress / flag

### Model naming consistency

- [ ] "BSM-Merton" (not "GK", not "Merton/GK", not plain "BSM") throughout
- [ ] greeks_model.type value is "bsm_merton" in all example records
- [ ] Delta identity stated as exp(-q*T), not 1.0 and not exp(-r*T)

### Signal computation conventions (T1 additions — required for every non-trivial computation epic)

- [ ] Every formula written out explicitly — no formula left inferable from an
      arithmetic example alone; non-obvious variants named and rejected
- [ ] Every enum has all values, both directions of any symmetric axis (up AND down,
      positive AND negative), and every boundary condition stated with direction
      (>, >=, <, <=) — no "toward centre" or half-specified symmetric taxonomies
- [ ] Every temporal-status flag states: (a) set condition, (b) exact clear
      timestamp, (c) union vs per-variant behaviour if multiple variants exist
- [ ] Every threshold used in a flag/classification condition appears as a named
      row in the NSE Market Structure Constants table — no in-prose-only numbers
- [ ] Epic pre-submission completeness sweep completed: (a) every temporal-status
      flag entry has all three components — set condition, exact clear timestamp,
      union vs per-variant; (b) every session-summary field has a formula or
      aggregation rule explicitly stated; (c) every literal example record in the
      Output Format section is consistent with the Part A acceptance bounds
      (T2 lesson — T2 rounds 3–6 each found one gap in one of these three categories;
      a structured sweep before submission would have prevented all four)

### Output schema and write model (T1 additions — required for any multi-writer epic)

- [ ] Row inventory table present: per (writer, row_type), states count per session,
      key unique fields, write trigger, and per-variant vs combined
- [ ] Per-variant vs combined schema decision made and stated in the epic Output
      Format section (not deferred to task review)
- [ ] Held/delayed writes identified: every row type that is written after the
      triggering bar is marked as "held" with its release condition stated
- [ ] Logical timestamp vs physical write order stated for any row that is written
      out of the order its ts field would imply

---

## 7. Handoff and Review Protocol

This section defines the process rules that govern the handoff from the desk to the
developer, and the conduct of each review round. These rules are not optional — they
are the process fixes that T0's nine rounds demonstrated were missing.

### Spec Completeness Gate (desk responsibility before handoff)

A spec is not ready to hand off until:

1. **No [PLACEHOLDER] token remains.** Every section must be filled. A spec with
   any unreplaced [PLACEHOLDER] is returned without review. The desk is responsible
   for this — handing off a spec with placeholder text transfers the desk's ambiguity
   directly into the developer's implementation choices.

2. **The pre-submission checklist is fully checked.** Every box in §6 must be ticked.
   Unchecked boxes mean the spec is in draft, not ready for implementation.

3. **The End-to-End Scenario Part A contains specific values, not ranges.** "NIFTY
   near 24000" is not an accepted input. "S = 24052.75, K = 24100, DTE = 15" is.
   If Part A uses illustrative ranges rather than specific values, the spec is
   returned.

### Developer Intake Gate (developer responsibility before implementation begins)

Upon receiving a spec, the developer runs through the pre-submission checklist in §6
before writing any code. If any mandatory section is incomplete:

- The spec is returned to the desk with a specific list of missing items.
- Implementation does not begin until the desk has completed all flagged items.
- Starting implementation on an incomplete spec is not permitted — doing so transfers
  the desk's unresolved ambiguities into the codebase, where they cost more to fix.

Rule: the developer is the last checkpoint before implementation. If the verified
example Part A is missing, or has illustrative ranges rather than specific values,
the developer returns the spec rather than making assumptions.

### Reviewer Conduct Rules

**Rule 1 — No finding without a spec reference.**
A reviewer may not raise a finding against a numerical output unless the spec already
carries the verified example that the finding is measured against. If the identity or
expectation is not in the spec:
- Write it into the spec first (as a spec update — next round)
- Then raise the finding against the updated spec in the subsequent round
- Do not raise a finding against a number that has no reference value in the spec

This is the rule that R4 violated. The reviewer tested the delta sum against
exp(-r*T) — an identity that was not in the spec. Had this rule been in place,
R4 would have produced a spec update ("add the delta identity and expected value"),
not a prescription to change the engine.

**Rule 2 — Prescriptions must be grounded in spec-stated expectations.**
A reviewer prescription to change a computation must cite the spec section that
defines the expected behaviour being violated. A prescription that cannot cite a
spec section is a spec gap, not an implementation bug.

**Rule 3 — Developer may reject a prescription that breaks a working engine.**
If a review prescription would cause a previously-correct assertion to fail, the
developer may reject it by documenting:
- Which parity check breaks (with the numeric evidence)
- The mathematical proof that the original was correct
- The specific identity the reviewer applied and why it is wrong for this model

The developer must document the proof before reverting, not after. The burden of
proof is on the rejection, not on the review. Holding a correct position under review
pressure is explicitly permitted and expected.

### Review Round Format (mandatory structure for every feedback round)

Every feedback round, whether from the desk to the developer or from the developer
back to the desk, must contain three sections in this order:

1. **Confirmed Clean:** A table of all items that have been confirmed correct in
   prior rounds. These items are closed — they do not recur.
2. **New Findings:** Items raised for the first time in this round.
3. **Carry-Forward:** Items from prior rounds that are not yet resolved.

This structure is not optional. It was improvised during T0 and proved effective —
it is now mandatory from T1 onward.

### Version Control Per Round

Before beginning implementation of any review prescription:

- Create a named git tag: `review-round-N` where N is the round number.
- A rejected prescription is reverted via `git revert` or branch switch, not
  by manual reconstruction.
- If a prescription touches production engine code (not just sample files), it must
  be on a named branch, not directly on the working tree.

This rule exists because T0's R5 revert was a manual undo with no isolation. If
the prescription had touched production code instead of sample files, a manual
undo would itself have been a second source of error.

### Environment Blockers Are Process Blockers (T2 lesson)

If any pre-submission gate item cannot be executed because of an environment
constraint — Bash denied, REPL unavailable, test runner blocked — the correct
action is to escalate it immediately as a named process blocker before submitting,
not to document it as a known gap and continue.

The T2 method-B gate was environment-blocked from round 1 through round 4. The
developer documented the block, substituted a Taylor-expansion workaround, and
submitted. The reviewer caught a numeric error (a rounding bug) that the method-B
gate would have caught before submission. The gate existed; it was not enforced.

Rule: if a gate item is blocked, raise it explicitly in the submission with a
request for resolution — do not submit a sample whose gate is satisfied only by
a substitute method. A gate you substitute around is a deferred finding, not a
closed gate. The workaround may be technically sound for one round; it is not
acceptable across multiple rounds.

This rule applies identically to test-suite gates: if pytest cannot run, escalate
as a process blocker. Do not submit tests without a run log and call them passing.

### Template-Copy Risk in Consumer Tasks (T2 lesson)

When authoring a consumer task (a task whose writer reads a producer's output),
every suppression rule must be re-derived from the consumer's temporal semantics —
not copied from the producer's spec. In T2, the F2 row-count asymmetry (epic round 4)
was caused by copying the F1 suppression rule ("exclude suppressed-classification
intervals") into F2 without adjusting for the fact that F2's basis is a level read
(valid at 09:15) while F1's cross-read is a delta read (null at 09:15). The copied
rule was internally consistent and survived spec review — it failed only when the
row inventory count was checked against the interval enumeration.

Rule: for any field in a consumer task that reads or copies a flag or classification
from a producer output, re-read the producer's temporal semantics and ask: "does this
suppression rule apply to the consumer with exactly the same semantics, or does the
consumer's different read type change the answer?"

---

## Model Naming Reference

This came up across three rounds (R4, R5, R6) and is worth stating clearly once.

Four formula variants are commonly encountered. All are siblings under the
generalised BSM-with-carry framework — none is a "special case" of another.
They differ only in what fills the cost-of-carry parameter b:

| Variant | b value | Used for |
|---|---|---|
| BSM-Merton | b = r - q | Equity with continuous dividend yield |
| Garman-Kohlhagen (GK) | b = r - r_f | FX options (foreign interest rate) |
| Black-76 | b = 0 | Futures options |
| Plain BSM | b = r | Equity with no dividends |

For NIFTY and BANKNIFTY equity index options: **BSM-Merton, b = r - q**.

The label "GK" (Garman-Kohlhagen) appeared in the T0 review comments and in
intermediate spec versions. It is not wrong for FX — but it is wrong for NSE
equity index options, where the dividend yield replaces the foreign rate.
Using "GK" in an NSE equity spec is technically incorrect and causes confusion
when a developer cross-references a textbook and finds a foreign-rate parameter
where they expected a dividend yield.

The shipped data carries `greeks_model.type = "bsm_merton"`. All specs must
use the same name to prevent the G4 failure they describe.

---

## DTE=0 Landmine

T = DTE/365 means T = 0 on expiry day. Division by zero and NaN Greeks follow.
Every task that computes Greeks or IV must handle DTE=0 explicitly:

- Suppress Greeks: write delta, gamma, theta, vega as None when DTE = 0
- Suppress IV: write iv as None when DTE = 0 (or DTE < 1, depending on the
  intraday expiry-day behaviour the desk wants)
- Never silently propagate NaN — the Pydantic schema rejects NaN on float fields,
  but the computation layer must catch it before it reaches the schema

This is not a Tier 2 concern. On every weekly expiry Tuesday, all NIFTY weekly
series contracts hit DTE=0. If this is not handled, the expiry-day capture fails
for the most active series.

---

## Basis Naming

`basis = futures_ltp - spot_ltp` is the raw observed basis. It is what the
market shows; it requires no model.

Fair-value basis = S * (exp(b * T) - 1) is a separate model-derived quantity.
It requires r, q, and T.

These two quantities converge near expiry but are not identical during the series.
Never write "basis" when you mean fair-value basis, and never use the raw basis
formula when you mean fair-value. If a schema ever needs to carry both, label
them `raw_basis` and `fair_value_basis`. Using `basis` alone in a spec is
ambiguous — always specify which one.
