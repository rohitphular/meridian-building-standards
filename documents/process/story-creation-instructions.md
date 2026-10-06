# How to Write a Story Requirement: Instructions for the Trading Desk

This document explains every section of the task template and why it exists.
Read it once. The template itself is self-contained once you understand the
reasoning behind each section.

This guide is grounded in T0: eight tasks, nine feedback rounds. Every rule
below is traceable to a specific failure in those nine rounds. Generic
software-engineering advice is not here — only the failures that actually occurred.

---

## 1. What a Story Is

A story is one implementable unit: one Python module, one pull request, one
developer holding it in their head at once. It inherits all conventions from its
parent epic (the shared-conventions section of the epic spec). It does not
redefine those conventions — it references them and overrides only what genuinely
differs for this specific story.

The smallest coherent unit of work that produces something reviewable is the
right size for a story. T0-001 (Pydantic schemas) was one story. T0-002 (IV solver)
was one story. T0-007 (options snapshot writer) was one story that integrates the
prior six. If a story combines schema definition, numerical computation, and file
writing into one module, it is too large — that is three stories.

---

## 2. The Six Mandatory Sections

Every story spec must contain these six sections. The section headers cannot be
skipped. Each one prevents a specific category of failure from T0.

### Section 1: Input Contract (prevents G1 and G3)

**G1** was "units, scale, or sign not stated." The Greek engine initially computed
theta per lot instead of per index unit — a 75x error — because the input contract
said "returns theta" without stating the scale. The fix required one round.

**G3** was "reference quantity ambiguous." Moneyness was initially computed against
spot, then a reviewer noted it should use the forward (futures LTP). The fix was
applied inconsistently, requiring a third round.

The input contract must be a table with five columns:
- Field name
- Type (float / int / str / datetime / Literal["CE","PE"])
- Units and scale (the column most often missing in T0 specs)
- Valid range
- What to do on invalid value (raise ValueError / return None / skip record)

Never describe an input in prose without also giving a concrete example value.
"Use the risk-free rate" is not an input specification. "r = 0.065 (NSE 91-day
T-bill, annualised decimal; not the US Fed funds rate)" is.

### Section 2: Output Contract (prevents G1, G3, G5)

**G5** was "output format not stated." The options NDJSON file was serialised as
multi-line pretty-printed JSON in Round 8, breaking every standard NDJSON parser.
The fix was trivial; the round was not.

The output contract must be a table with five columns:
- Field name
- Type
- Units, scale, and sign convention (all three, not just type)
- Nullable? — and if yes, the exact conditions under which the field is None
- Example value, with the formula or source that produced it

Zero and None are different things. iv=0.0 is a data error (the Pydantic schema
rejects it). iv=None is correct for low-OI or DTE=0 cases. Do not conflate them.

### Section 3: Verified Example (prevents G2 — the single biggest source of rounds)

**G2** was "no example output with verified values." All eight T0 specs described
logic but provided no concrete numbers to check. The three-round debate in R4–R6
over whether the Greek engine had a rate bug would not have happened if the spec
had included one table of expected values.

The verified example section has three parts:

**Part A (desk fills before implementation):** Scenario inputs with specific values,
and acceptance bounds per output field. The bounds are domain-plausibility ranges
the desk is confident in from experience. The desk does not hand-compute the exact
numbers — that creates circular verification.

Example bounds (from T0 retrospective §G2):
- "CE delta should be near 0.50–0.51 for a near-ATM call at 15 DTE"
- "theta_CE should be more negative than theta_PE — call decays faster on NSE"
- "CE_delta + |PE_delta| should be approximately 0.9995, not 1.0"

**Part B (developer fills during implementation):** Exact values from an
independent oracle — QuantLib, Haug tables, or scipy. The developer runs the
oracle against the Part A inputs and records the results. These become the
acceptance benchmarks. The developer's implementation must match the oracle,
not match the desk's mental arithmetic.

**Key identities to verify:** The relationships between fields that a correct
implementation must satisfy. For Greeks, these are specific formulas with
numeric tolerances. See Section 4 for what identities to state.

### Section 4: Math Model Specification (prevents G4 and G7)

**G4** was "which variant of BSM was not specified." T0-002 used the BSM formula
with the Garman-Kohlhagen cost-of-carry form but never stated the variant name.
In Round 4, the reviewer tested against the Black-76 delta identity
(CE_delta + |PE_delta| = exp(-r*T)) and found it failed, and issued a correction.
The developer correctly rejected the correction, but only after Round 5 applied
it and broke a working engine, and Round 6 rescinded the Round-5 fix. Three full
rounds from one missing line.

**G7** was "parity identities absent." The spec described what to compute but not
what must hold between computed fields. The reviewer had to derive and apply the
identities during feedback. Each application broke something else.

For any task that involves a formula, state:

**Model variant name:** Use "BSM-Merton, b = r - q" for NSE equity index options.
Not "BSM" (too ambiguous). Not "GK" (that is the Garman-Kohlhagen FX variant,
b = r - r_f). Not "Black-76" (that is the futures variant, b = 0). Not
"Merton/GK" (mixes two distinct variants). The four variants are siblings under
the generalised BSM-with-carry framework; none is a special case of another.

**Parameter values with sources:** Not just names. "r = 0.065 (NSE 91-day T-bill)"
is a specification. "Use the risk-free rate" is not.

**Day-count:** act/365 (calendar days). T = DTE / 365. Never trading days / 252
for BSM inputs — that is a 44% error in T.

**Key identity with numeric example:** State the formula and substitute the scenario
values to produce a number. "CE_delta + |PE_delta| = exp(-q * T)" is a formula.
"At T = 15/365, q = 0.0125: exp(-0.0125 * 15/365) = 0.99949 — this is NOT 1.0
and NOT exp(-r*T) = 0.9973" is a specification.

**Exact vs approximate identities:** Identities that hold within a single leg
(where one sigma is used for all Greeks) are exact. Identities that hold across
CE and PE legs are approximate, because each leg carries its own IV solved from
its own market price. The tolerance for the cross-leg delta parity check is
±0.002, not zero. Failing this check with a zero-tolerance assertion will
produce false failures on any day with real skew — which is every trading day.

### Section 5: Acceptance Criteria (tied to G7)

Each criterion must be binary: pass or fail, checkable with a number. "Looks
correct" is not an acceptance criterion. "CE_delta + |PE_delta| = 0.9995 ±0.001"
is an acceptance criterion.

The acceptance criteria section must include the parity checks from the verified
example. The parity checks are not just documentation — they are tests.

### Section 6: Null/Fallback Behaviour

This section existed partially in T0 specs but not completely. The spec said
"iv is None when solver fails" without listing all conditions: provider returns
0.0, DTE < 1, OI below threshold, bid == 0 AND ask == 0 AND stale LTP. Each
missed condition triggered a separate round.

State every condition that produces None for every nullable field. If two
conditions both produce None for the same field, list both. If a field can be
None for three different reasons, list all three.

Zero is not None. If a field can be zero (e.g., oi_change = 0 means OI did not
change), state that zero is a valid value. If a field should never be zero (e.g.,
iv = 0.0 is always a data error), state that and state what to do instead (write
None, log a warning, etc.).

---

## 3. Rules for Numeric Specifications

Every field that is a number must state four metadata items. Without all four,
the specification is incomplete.

**Unit:** What is one unit of this number? INR, decimal fraction (0.15 = 15%),
integer contracts, index points. "decimal" alone is not enough — state what the
decimal represents.

**Scale:** Per lot or per index unit? Per calendar day or per year? Per 1%-point
or per unit of vol (i.e., per 0.01 change in decimal IV)? The 75x factor (per
lot vs per unit for NIFTY with lot_size=75) is not a rounding issue — it changes
the magnitude of risk sizing. The 365-vs-252 choice changes theta by 44%.

**Sign convention:** Can this field be negative? If yes, what does a negative
value mean? Theta is always negative for long options. Delta is positive for CE,
negative for PE. oi_change is negative on net selling.

**Bounds:** What is the valid range? "delta in (0, 1) for CE, (-1, 0) for PE"
is a bound. "anything is possible" is not a bound. What happens at the boundary
(delta exactly 0, delta exactly 1)?

---

## 4. Rules for Math Model Specifications

**Name the variant exactly.** BSM alone is not specific. The four variants used
in practice for NSE are:

| Variant | b value | Context |
|---|---|---|
| BSM-Merton | b = r - q | NSE equity index options (NIFTY, BANKNIFTY) |
| Garman-Kohlhagen (GK) | b = r - r_f | FX options — wrong for NSE equity |
| Black-76 | b = 0 | Futures options |
| Plain BSM | b = r | Equity, no dividends — wrong for NIFTY (q = 1.25%) |

**State parameter values, not names.** r = 0.065 is a specification.
"The NSE risk-free rate" is not. A developer new to NSE might use the US Fed
funds rate (0.0525) and produce results that look plausible but are wrong by
100bps across every Greek.

**State the day-count.** act/365 (T = DTE/365, calendar days). This is the T0
convention. Theta = annual_theta / 365, not / 252.

**State the key identity with a numeric substitution.** The identity is the
reviewer's check. Without the numeric substitution, the reviewer has to compute
it from the parameters — and that computation is where R4, R5, and R6 went wrong.

Example of complete math model specification (from the T0 retrospective):
```
Model: BSM-Merton, b = r - q
r = 0.065, q = 0.0125
T = DTE / 365 (calendar days, act/365)
Delta convention: spot-delta, dV/dS

Key identity:
CE_delta + |PE_delta| = exp(-q * T)
At T = 15/365, q = 0.0125: exp(-0.0125 * 15/365) = 0.99949
NOT 1.0 (that is BSM without dividends)
NOT exp(-r*T) = 0.9973 (that is Black-76 forward-delta)
Tolerance for cross-leg check: ±0.002 (approximate; breaks with skew)
```

---

## 5. Rules for Output Format Specifications

These rules apply to any task that writes files.

**Always include a literal example record.** Not a description of the format —
an actual line that a developer can copy, parse, and validate their output against.
The Round-8 NDJSON failure in T0 (pretty-printed multi-line JSON) would have been
caught before any code was written if the spec had included one literal line.

**State the serialisation rule explicitly.** "NDJSON" does not specify:
- Compact vs pretty-printed (no whitespace between tokens is compact)
- Whether blank lines are permitted (they are not — standard NDJSON)
- Whether there is a trailing newline after the last record (there should be)

**State the rounding policy per field.** Float fields written to JSON without
a rounding policy produce values like `basis: 65.65000000000146` (floating-point
noise from a subtraction). Specify:
- basis: 2 decimal places
- delta, gamma: 10 decimal places (precision is meaningful for hedging)
- theta, vega: 4 decimal places (sub-pip is not actionable)
- iv: full precision (stored as decimal fraction; IV rank calculations need it)
- change_frac: full precision (small numbers; 4dp loses signal)

**Field names must be valid Python identifiers.** `52w_high` is not valid
(leading digit). Use `high_52w`. This is a naming decision that belongs in the
spec, not a developer choice.

**State the Row Inventory for every row_type this story writes (T1 lesson).**

T1-003 had four spec rounds on output schema — all on a writer whose computation
was correct from round 1. The computation was specified; the schema was not.
Every file-writing task must include a row inventory stating, per row_type:

- **Count per session:** "exactly N rows" or "one per qualifying event." Do not
  write "emits opening_range rows" — write "emits exactly one opening_range row
  per OR variant per session" or "emits exactly one combined opening_range row."
- **Key fields:** Which fields distinguish this row_type from others in the same
  file? What fields are present on this row_type but absent on others?
- **Write trigger:** Immediate on the bar that completes the condition, or held.
  "Held" must state when the row is released: "held to session end (15:30)",
  "held until breakout event fires", etc. A row that is written after the bar
  that triggered it is a delayed write; the spec must say so.
- **Logical ts vs physical write order:** If a row's `ts` field contains the
  bar timestamp but the row is physically written later (session end), state this
  explicitly. A reviewer reading the file in physical order will see the row
  appear after rows with later timestamps — that is not an error, but it must be
  in the spec.

**Include the session_summary rows in every sample submission (T1 lesson).**

The story spec allows mid-session oracle slices. The session_summary row still
must appear in the initial sample submission, even if it represents the slice
state rather than a real 15:30 close. Include a comment in the sample explaining
this. Missing session_summary rows in round 1 consumed a round-1 response that
added nothing except the rows. The overhead is one row per writer; the saving is
one round.

---

## 6. Rules for Null/Fallback Behaviour

**List every condition exhaustively.** The partial list in T0 (iv is None when
solver fails) missed four other conditions. Each missed condition was one round.

The complete null condition list for iv from T0 (should have been in the spec):
- Provider returns iv = 0.0 (always a data error for NSE; convert to None)
- DTE = 0 (T = 0 causes division by zero; suppress IV on expiry day)
- DTE < 1 (extension of the above if using a DTE-based threshold rather than 0)
- OI <= oi_threshold (low-liquidity contract; IV is unreliable)
- IV solver returns None (convergence failure, no extrinsic value)
- bid == 0 AND ask == 0 AND LTP is stale (no tradeable price; IV is meaningless)

**Distinguish None from zero.** If a field can validly be zero, say so and give
an example. If zero is always a data error, say so and state the treatment (write
None, raise, log).

**Cascade None to dependent fields.** If iv is None, then delta, gamma, theta,
and vega are all None. This should be a Pydantic model-level validator, not an
ad-hoc check in the writer. The cascading rule belongs in the spec, not in the
developer's implementation choices.

---

## 7. How to Handle Ambiguity

**Decisions to make before handing off the spec.**

Some choices have no universally correct answer — they are design decisions the
desk must make. Do not hand off a spec with these unresolved:

- Which IV to use for same-strike CE/PE: solve independently from each leg's
  market price, or solve once from the CE and copy to the PE?
- Which reference price for moneyness: spot LTP or near-month futures LTP?
- What counts as "ATM": a percentage band (±0.5% of forward) or a fixed-point
  distance (±50 index points)?
- Is oi_change the EOD delta vs prior session close, or the intraday delta vs
  the prior 5-minute snapshot?

Each decision is one sentence in the spec. Leaving it implicit costs one round.

**Open questions to flag, not decide.**

Some questions you cannot answer without developer input. Flag them explicitly:

> "Open question: the IV solver may fail on deep OTM strikes where the market
> price equals intrinsic value. Developer should document the failure rate on a
> typical NIFTY chain and flag if it exceeds 5% of contracts."

Format: "Open question: [the question]. Developer should [recommend / flag /
decide] before implementation begins."

Never leave an open question implicit. The desk's uncertainty about something
is information the developer needs before they choose an implementation. A
conversation before implementation is one message. A conversation during review
is one round.

**When to escalate before implementation.**

Escalate to a domain discussion before any code is written when:
- Two valid interpretations produce results that differ by more than 10% for a
  key field (e.g., spot vs forward for moneyness changes which strikes are ATM)
- The choice affects a signal's direction (positive vs negative), not just
  magnitude
- A market structure constant has recently changed (NSE changed NIFTY weekly
  expiry from Thursday to Tuesday in 2024 — any spec that still said Thursday
  was structurally wrong)

---

## 8. Pre-Submission Checklist for Stories

Run through this before handing any story spec to the developer.

### Input and output contracts

- [ ] Every input field has: type, units/scale, valid range, on-invalid behaviour
- [ ] Every output field has: type, units/scale/sign, nullable condition, example value
- [ ] Zero and None are distinguished for every nullable field
- [ ] No field is described with just a name and a type

### Verified example

- [ ] Scenario uses specific numeric inputs, not illustrative ranges
- [ ] Acceptance bounds are numeric ranges, not "should look correct"
- [ ] Part B (oracle values) section is present for developer to fill
- [ ] Key identities are stated with numeric substitutions, not just as formulas
- [ ] Cross-leg checks have tolerance bands (±0.002 for delta parity, not zero)

### Math model (when applicable)

- [ ] Variant named exactly: "BSM-Merton, b = r - q" — not "GK", not "BSM"
- [ ] r and q stated with values: r = 0.065, q = 0.0125
- [ ] Day-count stated: act/365, T = DTE/365
- [ ] Delta convention stated: spot-delta (dV/dS)
- [ ] Key identity stated with numeric substitution

### Output format (when story writes files)

- [ ] Serialisation rule stated: compact JSON, no whitespace
- [ ] Literal example record provided (one line, all fields)
- [ ] Rounding policy stated per field
- [ ] All timestamps: timezone-aware with UTC offset
- [ ] Row inventory present: per row_type, states count per session, key fields,
      write trigger, held vs immediate (T1 lesson)
- [ ] Delayed-write rows identified with release condition stated
- [ ] session_summary row present with mid-session state comment if slice ends
      before 15:30 (T1 lesson — missing session_summary cost a round-1 response)
- [ ] Enum fields: all values, both directions of any symmetric axis, all
      boundary directions stated (T1 lesson — incomplete taxonomy = review round)
- [ ] Test suite run log attached in the same round as tests: if this task includes
      unit tests, the pytest output ("N passed in Xs") is attached to this round —
      "tests written" and "tests run and passing" are different claims; the reviewer
      cannot verify the latter without the log (T2 lesson — round 5→6 split caused
      solely by missing run log)
- [ ] Round-isolation verified: sample file at round=N/sample/ is the direct output
      from the production writer path — not hand-assembled, not copied from a prior
      round, not a merged multi-story file (T2 lesson — F1 rows in F2 sample persisted
      for two rounds because the claimed fix was not written to the reviewed path)

### Null behaviour

- [ ] All conditions that produce None are listed (not just "solver fails")
- [ ] Cascading None rules stated (iv=None implies all Greeks=None)
- [ ] DTE=0 case handled: Greeks and IV suppressed
- [ ] Mid-session carry-forward covered: zero-volume bars, gap bars, and
      accumulator state during dead intervals are addressed (T1 lesson)
- [ ] Session-summary denominators stated explicitly: every ratio field in
      session_summary identifies its denominator (T1 lesson)

### Acceptance criteria

- [ ] Every criterion is binary: pass or fail, checkable with a number
- [ ] Parity checks from verified example section are included
- [ ] OI divisibility check included for any task writing OI fields
- [ ] Example values in Output Contract come from the named Part A scenario bar,
      not copied across fields or invented separately (T1 lesson)
- [ ] Consumer tasks (tasks whose writer reads another writer's output): output file
      contains only this task's Row Inventory row types — stated as an explicit,
      grep-checkable acceptance criterion in the spec (T2 lesson — the "F2 output
      contains only F2 row types" rule lived as an epic principle but not as a task
      AC; it was not caught before round 1 review)

### Ambiguities

- [ ] Every design choice is resolved and written down
- [ ] Every genuine uncertainty is written as "Open question: ..."
- [ ] "Out of scope" section lists what this task does not cover

---

## 9. Handoff and Review Protocol

This section defines the process rules for the handoff from desk to developer, and
the required structure for every review round. These rules are the process fixes that
T0's nine rounds demonstrated were missing from the original workflow.

### Spec Completeness Gate (desk responsibility before handoff)

A story spec is not ready to hand off until three conditions are met:

1. **No [PLACEHOLDER] token remains in any section.** A spec with unreplaced
   placeholder text is returned without review. The desk is solely responsible for
   this — every placeholder token left in the spec represents an unresolved decision
   that will cost at least one review round to surface.

2. **Every box in the §8 pre-submission checklist is ticked.** Unchecked boxes mean
   the spec is still in draft status.

3. **Verified Example Part A contains specific numeric inputs, not illustrative
   ranges.** "NIFTY near 24000" fails this gate. "S = 24052.75, K = 24100, DTE = 15,
   CE LTP = 185.0" passes. Acceptance bounds must be concrete ranges: "0.50–0.51",
   not "near 0.5."

### Developer Intake Gate (developer responsibility before implementation begins)

Upon receiving a story spec, the developer runs through the §8 checklist before
writing any code. If any mandatory section fails the check:

- Return the spec to the desk with a specific list of missing items.
- Do not begin implementation until the desk has completed all flagged items.
- Do not make assumptions to fill gaps — assumptions that turn out wrong cost more
  to fix than a one-day delay on the handoff.

Rule: if the Verified Example Part A is missing, or uses illustrative ranges rather
than specific values, return the spec. If the math model section is absent for a story
that involves a formula, return the spec. These are not negotiable gates.

The developer is the last checkpoint before implementation starts. If a gap reaches
the developer, the developer catching it here is the cheapest possible fix.

### Developer Pre-Submission Self-Review (T1 lesson — run before every sample submission)

Two of T1's five rounds were caused by avoidable developer errors: a missing
`session_summary` row and a hand-arithmetic transcription error in the response
table. Both were visible from the acceptance criteria and the formula without
any desk input. Before submitting any sample, the developer runs this checklist:

**1. Acceptance criteria row-by-row.** Read every bullet in the Acceptance Criteria
section. For each criterion, find the corresponding row(s) in the sample and verify
the criterion passes. Check it off. Do not submit until every bullet is checked.
This is not a skim — read each criterion, find the relevant row, verify the value.
The T1 `or_provisional` omission (round 4) and missing `session_summary` (round 1)
were both in the acceptance criteria and would have been caught by this check.

**2. Oracle arithmetic verified by two independent methods.** Any numeric value
recorded in a response document (round-N arithmetic table, comment, explanation)
must be computed by two separate methods before being written down:
- Method 1: hand arithmetic (or REPL evaluation)
- Method 2: code output (what the implementation actually produces)
If the two disagree, find the error before writing the response. Never submit a
response with unverified intermediate arithmetic. The T1 `range_pct` transcription
error (`0.47967` stated, `0.47925` correct) propagated to the sample file because
the hand-computed value was recorded without being cross-checked against the code.

**3. session_summary present.** Verify the sample file contains a `session_summary`
row. If the oracle slice ends before 15:30, include the mid-session state with a
comment. Do not submit a sample missing session_summary on the grounds that the
slice doesn't reach session end.

**4. Bar-by-bar trace for any session-temporal flag.** For every flag that turns on
and off over the session (flags with a temporal lifecycle), produce a trace table
before submission: one row per bar, columns for the state-machine inputs (e.g.
`or_15_final`, `or_30_final`), and the expected flag presence at that bar. Verify
the sample matches the trace. The T1 `or_provisional` miss required this trace to be
produced as round-4 response work; it should have been produced before round 4.

**5. Test run log attached (T2 lesson).** If this story includes unit tests, attach
the literal pytest output — "N passed in Xs" — in the same round as the tests.
"Tests written" and "tests run and passing" are different claims. The reviewer cannot
verify the latter without the log. The T2 round 5→6 split happened for exactly this
reason: the developer described the tests and stated they passed, but the log was
absent; a full round was consumed solely by adding `pytest -v` output. Mark N/A if
this story has no unit tests.

**6. Round-isolation verified (T2 lesson).** Before submitting, confirm the sample
file at the round=N/sample/ path is the direct output from the production writer —
not hand-assembled, not copied from a prior-round path, not a merged file containing
other stories' row types. A one-line check is sufficient:
`head -1 round=N/sample/output.ndjson | python -c "import sys,json; d=json.loads(sys.stdin.read()); assert d['row_type'].startswith('expected_prefix')"`
The T2 F1-rows-in-F2-sample issue persisted for two rounds because the developer
applied arithmetic fixes in place but did not write the corrected file to the reviewed
path. The reviewer tested the same file twice.

**Environment blockers are process blockers.** If items 2, 5, or 6 cannot be
satisfied because of an environment constraint (Bash denied, test runner unavailable,
REPL blocked), the correct action is:
1. Raise it explicitly in the submission as a named process blocker with a request
   for resolution.
2. Do not submit a sample whose gate is satisfied by a substitute method (Taylor
   expansion, visual inspection, description-only) without explicitly flagging it.
A gate you substitute around is a deferred finding, not a closed gate. The T2
method-B gate was environment-blocked from round 1 through round 4; the developer
documented it and continued; the reviewer caught the error the gate would have caught.

### Reviewer Conduct Rules

**Rule 1 — No finding without a spec reference.**
A reviewer may not raise a finding against a numerical output unless the spec already
carries a verified example that defines the expected value or range for that field.

If the reviewer identifies a discrepancy but the spec has no verified example for
that field:
- The finding goes back to the spec as a spec update request, not to the developer.
- The spec is updated first (with the correct expected value and identity).
- The finding is raised in the following round against the updated spec.

This is the rule that R4 violated. The reviewer tested the delta sum against
exp(-r*T) — an identity that did not exist in the T0-002 spec. A reviewer cannot
flag something as wrong if there is no reference for what correct looks like.

**Rule 2 — Prescriptions must cite a spec section.**
A prescription to change the implementation must cite the spec section that defines
the expected behaviour being violated. A prescription that cannot cite such a section
is a spec gap. Spec gaps are fixed in the spec; they do not become implementation
tasks.

**Rule 3 — Developer may reject a prescription that breaks a working engine.**
If a review prescription causes a correct assertion to fail, the developer may reject
it by documenting:
- The specific assertion that fails (with the numeric evidence from the output)
- The mathematical proof that the original was correct (formula + substituted values)
- The identity the reviewer applied and why it is wrong for this specific model variant

The developer must write the proof before reverting. This is not optional — it is
what made the R5 revert credible when R6 confirmed the engine was correct. Stating
"the reviewer is wrong" is not sufficient; stating "here is exp(-q*T) = 0.99949 and
here is why that is not exp(-r*T) = 0.99733" is.

### Review Round Format (mandatory in every round)

Every feedback communication — from desk to developer or developer to desk — must
contain three sections in this order:

1. **Confirmed Clean:** All items confirmed correct in prior rounds. These are closed
   and do not recur. Both parties must agree a prior item is clean before it can be
   listed here.
2. **New Findings:** Items raised for the first time in this round, with the spec
   reference for each.
3. **Carry-Forward:** Items not yet resolved from prior rounds.

This structure was improvised in T0 and was the single most effective process tool
that emerged. It is mandatory from T1 onward.

### Version Control Per Round

Before the developer begins implementing any review prescription:

- Create a named git tag: `review-round-N` (e.g. `review-round-5`).
- A rejected prescription is reverted via `git revert` or branch switch, not manual
  reconstruction from memory.
- Any prescription that touches production engine code (not just sample files) must
  be on a named branch, not directly on the working tree.

The R5 manual revert in T0 was a single source of error; for production code it
would have been two.

### Round-Count Trigger for Scope Review

A story that requires more than 3 implementation review rounds without reaching APPROVED must trigger a scope review before round 4 begins. Flag to the scrum master after round 3. Do not continue to round 4 without scrum master sign-off.

The scrum master determines whether the root issue is:
- **Story too large** — split the story into smaller units
- **Spec ambiguous** — return to story spec review; reopen story details before resuming implementation
- **Acceptance criteria wrong** — revise ACs before any further implementation rounds

Three implementation rounds without closure is a process signal, not a content debate.

### Part B and Part C Are Approval Gates, Not Optional Documentation

Part B (developer's oracle values) must be filled and submitted to the desk before
the story is marked `approved`. The desk's Part C plausibility sign-off is required
before the status changes from `in-review` to `approved`. These are hard gates:

- A story with Part B empty is not complete, regardless of whether the code works.
- A story with Part C not signed off is not approved, regardless of Part B values.

The verified example is the audit trail that makes future maintenance safe. A story
without it is a time bomb for the next reviewer who cannot tell whether the current
output was ever validated.

---

## Appendix: T0 Field Conventions Reference

These conventions are fixed in the codebase. New tasks must be consistent with them.

| Field | Convention |
|---|---|
| iv | Decimal fraction (0.0939 = 9.39%). Never 0.0 — use None. |
| change_frac | Decimal fraction. Not a percentage. (ltp - prev_close) / prev_close |
| delta | Spot-delta (dV/dS). CE in (0,1), PE in (-1,0). CE + \|PE\| = exp(-q*T) ≈ 0.9995. |
| gamma | Per 1 INR move in spot. Per index unit, not per lot. Always positive. |
| theta | INR per calendar day per index unit. Always negative. Divide annual by 365, not 252. |
| vega | INR per 1%-point IV change per index unit. Always positive. BSM vega * 0.01. |
| oi | Integer, exchange units, always a multiple of lot_size. Not in lots. |
| oi_change | Integer, exchange units. EOD delta vs prior session close, not intraday. |
| oi_change_basis | String literal "prior_session_eod". Not tribal knowledge — it is in the record. |
| dte | Calendar days to expiry, not trading days. DTE=0 on expiry day — suppress Greeks and IV. |
| basis | Raw observed basis only: futures_ltp - spot_ltp, INR, 2dp. Fair-value basis is separate. |
| ts | Timezone-aware datetime with UTC offset. IST is +05:30. No naive datetimes. |
| high_52w | Not 52w_high (leading digit is not a valid Python identifier). |
| lot_size | From provider or instrument spec. Never hardcode. |
| moneyness | Forward-relative (uses futures LTP). ATM band ±0.5% — NIFTY-specific. |
| greeks_model | Inline dict: type = "bsm_merton", r, q, t_calendar_days, day_count, underlying_ref, vol_source |
