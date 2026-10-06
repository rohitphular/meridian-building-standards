# [STORY-ID] — [Story Name]

> Replace [STORY-ID] with the epic-qualified ID, e.g. T1-003.
> Replace [Story Name] with a short noun phrase, e.g. "BSM IV Solver" or
> "Options Snapshot Writer." Do not use a verb phrase ("Build the solver") —
> the story ID conveys it is being built.

---

## Status

> Choose one: `draft` | `in-review` | `approved` | `completed`

`draft`

---

## What This Task Builds

> One paragraph. What is the output? What problem does it solve? What does it
> feed downstream? The developer uses this paragraph to understand the scope
> before reading the contracts.
>
> T0-007 example:
> "The capture function for the full options chain. It enforces the null-IV rule,
> applies the stale LTP rule, invokes the BSM IV solver, computes all four Greeks,
> determines moneyness, and applies seven flags. It is the integration point that
> brings T0-001 through T0-006 together. The output NDJSON files feed every
> Tier 1+ options signal."

[PLACEHOLDER — one paragraph]

---

## Depends On

> List task IDs and the specific symbols or functions imported from each.
> "Depends on T0-001" is not enough. "Depends on T0-001 (OptionsRecord,
> CaptureFlags)" is.
>
> T0-007 example:
> - T0-001: OptionsRecord, CaptureFlags
> - T0-002: bsm_iv
> - T0-003: bsm_greeks_or_none
> - T0-004: write_snapshot

[PLACEHOLDER]

---

## Epic Conventions Inherited

> Reference the parent epic's shared conventions section. State any overrides
> that apply only to this task. Do not copy the conventions here — reference them.
>
> Example:
> "Inherits all conventions from T1 §Shared Conventions. Override: this task
> uses q = 0.0100 for BANKNIFTY (different dividend profile); all other parameters
> as per T1."

Inherits: [EPIC-ID] §Shared Conventions

Overrides (if any): [NONE | PLACEHOLDER — state override and reason]

---

## Pre-Implementation Gate

> Developer fills this section within 24 hours of receiving the spec.
> Do not write a single line of implementation code until this gate is cleared.
> If any section is incomplete, return the spec to the desk with the missing items
> listed below and await a corrected version. See §9 of story-creation-instructions.md.

| Section | Status | Missing or incomplete items |
|---|---|---|
| Input Contract — every field has type, units/scale, valid range, on-invalid | COMPLETE / INCOMPLETE | |
| Output Contract — every field has type, units/scale/sign, nullable condition, example | COMPLETE / INCOMPLETE | |
| Verified Example Part A — specific numeric inputs (not illustrative ranges) | COMPLETE / INCOMPLETE | |
| Verified Example Part A — acceptance bounds are concrete ranges, not "looks correct" | COMPLETE / INCOMPLETE | |
| Math Model Specification — variant named, parameters stated, key identity with substitution | COMPLETE / INCOMPLETE / N/A | |
| Null / Fallback Behaviour — every nullable field, every condition exhaustively listed | COMPLETE / INCOMPLETE | |
| Acceptance Criteria — every criterion is binary and checkable with a number | COMPLETE / INCOMPLETE | |
| No [PLACEHOLDER] token remaining in any section | COMPLETE / INCOMPLETE | |

Gate result: [CLEARED — implementation may begin | RETURNED — spec sent back on DATE with gaps listed above]

---

## Input Contract

> Table with five columns. Fill every cell. The Units/Scale column is the one
> most often missing — it is also the one that caused the most T0 rework.
>
> On-invalid options: raise ValueError | return None | skip record and log |
> use default (only when the default is domain-meaningful, not just convenient)
>
> T0-002 example inputs:

| Field | Type | Units / Scale | Valid range | On invalid |
|---|---|---|---|---|
| S | float | INR (index points) | > 0 | raise ValueError |
| K | float | INR (index points) | > 0 | raise ValueError |
| T | float | years: DTE / 365 calendar days | > 0 | return None (suppresses Greeks) |
| r | float | annualised decimal (0.065 = 6.5%) | (0, 1) | raise ValueError |
| q | float | annualised decimal (0.0125 = 1.25%) | [0, 1) | raise ValueError |
| sigma | float | annualised decimal (0.15 = 15%) | (0, 5.0] | return None |
| option_type | Literal["CE","PE"] | exact string | "CE" or "PE" only | raise ValueError |

> Replace the T0-002 rows with the inputs for this task. Keep the column headers.

| Field | Type | Units / Scale | Valid range | On invalid |
|---|---|---|---|---|
| [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] |

---

## Output Contract

> Table with five columns. The Nullable column must state the exact condition
> under which the field is None — not just "yes" or "no." Zero and None are
> different: state which applies.
>
> T0-007 example outputs:

| Field | Type | Units / Scale / Sign | Nullable? (when None) | Example value |
|---|---|---|---|---|
| iv | float or None | decimal fraction; 0.0939 = 9.39%; never 0.0 | when: provider returns 0.0, DTE <= 0, OI <= threshold, solver fails, bid=0 AND ask=0 AND stale | 0.0939526529 |
| delta | float or None | spot-delta dV/dS; CE in (0,1); PE in (-1,0) | when: iv is None (any reason) | 0.5076 (CE), -0.4919 (PE) |
| gamma | float or None | delta-change per 1 INR spot move; per index unit; always positive | when: iv is None | 0.000870 |
| theta | float or None | INR per calendar day per index unit; always negative | when: iv is None | -7.811 (CE), -4.365 (PE) |
| vega | float or None | INR per 1%-point IV change per index unit; always positive | when: iv is None | 19.44 |
| moneyness | Literal["ITM","ATM","OTM"] | categorical | never None | "ATM" |
| oi | int | exchange units; multiple of lot_size | never None | 12525 |
| oi_change | int | exchange units; multiple of lot_size; negative = net selling | never None | -750 |

> Replace with this task's output fields. Keep all five columns.

| Field | Type | Units / Scale / Sign | Nullable? (when None) | Example value |
|---|---|---|---|---|
| [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] | [PLACEHOLDER] |

---

## Verified Example

> This section is the single highest-leverage part of any task spec. The three-round
> debate in T0 R4–R6 over whether the Greek engine had a rate bug would not have
> happened if the original spec had included this section.
>
> Three-role authorship rule: desk fills Part A before implementation; developer
> fills Part B from an independent oracle during implementation; desk signs off
> on Part C plausibility before marking the task approved.

### Part A — Scenario Inputs and Acceptance Bounds (Desk fills before implementation)

> Use specific values, not illustrative ranges. "NIFTY near 24000" is not a
> scenario. "S = 24052.75" is.
>
> Acceptance bounds are domain-plausibility ranges. The desk does not compute
> exact values (that is circular verification). The bounds are what the desk
> knows from experience: "near-ATM delta is always near 0.5."

Scenario inputs:
```
Symbol:       [PLACEHOLDER e.g. NIFTY]
Date/time:    [PLACEHOLDER e.g. 2026-07-14T09:35:00+05:30]
Spot LTP:     [PLACEHOLDER e.g. 24052.75]
Futures LTP:  [PLACEHOLDER e.g. 24118.40]
Expiry:       [PLACEHOLDER e.g. 2026-07-29]
DTE:          [PLACEHOLDER — integer calendar days]
Strike:       [PLACEHOLDER e.g. 24100]
option_type:  [PLACEHOLDER — CE or PE]
LTP:          [PLACEHOLDER e.g. 185.0]
bid / ask:    [PLACEHOLDER e.g. 183.0 / 187.0]
OI:           [PLACEHOLDER — must be a multiple of lot_size]
lot_size:     [PLACEHOLDER e.g. 75]
r:            0.065  (from epic §Shared Conventions; override if different)
q:            0.0125 (from epic §Shared Conventions; override if different)
```

Acceptance bounds (desk fills):

| Output field | Acceptance bounds | Domain reasoning |
|---|---|---|
| [PLACEHOLDER field] | [PLACEHOLDER range] | [PLACEHOLDER — one sentence] |

> T0 example rows (add rows for this task's outputs):
> | iv | 9.3% to 9.5% (decimal 0.093 to 0.095) | Near-ATM, 15 DTE; IV should be in the low single digits for NIFTY in normal conditions |
> | delta (CE) | 0.50 to 0.51 | Near-ATM call delta is always near 0.5 by definition of ATM |
> | theta (CE) | more negative than theta (PE) | Call decays faster than put on NSE always (r > q always) |
> | CE_delta + \|PE_delta\| | approximately 0.9995, not 1.0 | BSM-Merton: sum tracks exp(-q*T), not 1.0 |

### Part B — Oracle Values (Developer fills during implementation)

> Developer: run QuantLib or scipy against the Part A inputs. Record values here.
> These become the acceptance benchmarks. Your implementation must match these,
> not match the desk's mental arithmetic.
>
> Example oracle call (Python, scipy):
> ```python
> from scipy.stats import norm
> import math
> S, K, T, r, q, sigma = 24052.75, 24100, 15/365, 0.065, 0.0125, 0.0939
> b = r - q  # BSM-Merton
> d1 = (math.log(S/K) + (b + 0.5*sigma**2)*T) / (sigma * math.sqrt(T))
> d2 = d1 - sigma * math.sqrt(T)
> delta = math.exp((b-r)*T) * norm.cdf(d1)  # CE spot-delta
> ```

```
[DEVELOPER FILLS — field: oracle_value (source: QuantLib/scipy/Haug)]

iv:           [DEVELOPER FILLS]
delta (CE):   [DEVELOPER FILLS — e.g. 0.5076229027]
delta (PE):   [DEVELOPER FILLS — e.g. -0.4918756201]
gamma:        [DEVELOPER FILLS — e.g. 0.0008702202]
theta (CE):   [DEVELOPER FILLS — e.g. -7.8109573856]
theta (PE):   [DEVELOPER FILLS — e.g. -4.3647283124]
vega:         [DEVELOPER FILLS — e.g. 19.4386]
[add fields for this task]
```

### Part C — Desk Plausibility Sign-Off (Desk fills after reviewing Part B)

> Desk: review Part B oracle values before marking this task approved.
> Confirm: (a) signs are correct for every field, (b) magnitudes are in the
> domain-plausible range, (c) asymmetric fields (theta_CE vs theta_PE, IV skew
> direction) are in the expected direction. Do not recompute Part B — that is
> circular. Only assess domain plausibility.
>
> Sign off by filling the table and setting Status to `approved`.

| Field | Domain plausibility check | Pass / Fail | Notes |
|---|---|---|---|
| [PLACEHOLDER] | [PLACEHOLDER — e.g. "sign is negative for CE theta"] | [PASS/FAIL] | |

Desk sign-off: [DESK INITIALS / DATE — required before status → `approved`]

### Key Identities to Verify

> State the formula and substitute the scenario values to produce a number.
> Include tolerance for cross-leg checks (approximate) vs within-leg checks (exact).
>
> The T0 identities that a correct BSM-Merton Greek engine must satisfy:

| Identity | Formula | Numeric value at scenario inputs | Tolerance |
|---|---|---|---|
| CE/PE delta parity | CE_delta + \|PE_delta\| = exp(-q * T) | exp(-0.0125 * 15/365) = 0.99949 | ±0.002 (approximate; per-leg IVs differ) |
| Same-strike IV parity | \|IV_CE - IV_PE\| | < 5 basis points | < 0.0005 |
| Theta asymmetry direction | theta_CE < theta_PE | theta_CE - theta_PE ≈ -3.45 per day | exact direction; ±0.10 magnitude |
| Gamma parity | \|gamma_CE - gamma_PE\| / gamma_CE | < 1% | < 0.01 |
| OI divisibility | oi % lot_size == 0 | 12525 % 75 == 0 | exact |

> Add rows for this task's specific identities. Keep the BSM rows above for any
> task with Greek computation.

[PLACEHOLDER — add task-specific identities if any]

---

## Math Model Specification

> Fill this section for any task that involves a formula.
> Skip this section (write "N/A — no formula computation in this task") for
> tasks that are pure schema, file I/O, or data transformation without formulas.
>
> T0 example:

**Model variant:** BSM-Merton, b = r - q (continuous-dividend equity form)

Do not use:
- "GK" or "Garman-Kohlhagen" (that is the FX variant: b = r - r_f)
- "Black-76" (futures variant: b = 0)
- Plain "BSM" (no dividends: b = r)

**Parameters:**
```
r = 0.065   (NSE 91-day T-bill; annualised decimal; from epic §Shared Conventions)
q = 0.0125  (NIFTY trailing dividend yield; annualised decimal; from epic §Shared Conventions)
T = DTE / 365  (calendar days; act/365 day-count; NOT trading days / 252)
S = spot LTP (underlying_ltp field)
```

**Delta convention:** spot-delta, dV/dS
- CE spot-delta: delta = exp((b-r)*T) * N(d1), result in (0,1)
- PE spot-delta: delta = exp((b-r)*T) * (N(d1) - 1), result in (-1,0)
- NOT Black-76 forward-delta (uses S = F = futures price)
- NOT plain BSM delta (uses b = r)

**Key identity (with numeric example):**
```
CE_delta + |PE_delta| = exp(-q * T)
At T = 15/365, q = 0.0125:
  exp(-0.0125 * 15/365) = 0.99949
  This is NOT 1.0 (BSM without dividends)
  This is NOT exp(-r*T) = exp(-0.065 * 15/365) = 0.99733 (Black-76 forward-delta)
  A reviewer seeing 0.9995 should NOT flag it as a rounding error.
```

**DTE=0 landmine:** T = DTE/365 = 0 on expiry day. This causes division by zero
in d1/d2 and NaN Greeks. Suppress by returning None for all Greeks when DTE = 0.
Do not propagate NaN — Pydantic rejects it and the failure will be silent.

> Override the model specification above if this task uses a different variant.
> State the override, the b value, and the reason.

[CONFIRM — or OVERRIDE with variant name, b value, parameters, and reason]

---

## Output Format

> Fill this section for any task that writes files.
> Skip (write "N/A — this task does not write files") for computation-only tasks.

**Format:** `.ndjson`

**Serialisation rule:**
- Compact JSON: no spaces after colons, no spaces after commas, no indentation
- One JSON object per line; newline `\n` as the sole delimiter
- No enclosing array, no trailing comma
- File ends with a newline after the last record
- No blank lines

**Literal example record (one line — this is the specification, not illustration):**

```
[PLACEHOLDER — paste one complete output line here; all fields populated]
```

> T0-007 example line:
> ```
> {"symbol":"NIFTY","instrument_type":"option","exchange":"NSE","expiry":"2026-07-29","dte":15,"strike":24100.0,"option_type":"CE","ts":"2026-07-14T09:35:00+05:30","ltp":185.0,"bid":183.0,"ask":187.0,"prev_close":170.0,"volume":32100,"oi":12525,"oi_change":-750,"oi_change_basis":"prior_session_eod","iv":0.0939526529256709,"delta":0.5076229027,"gamma":0.0008702202308,"theta":-7.8109573856,"vega":19.4386,"underlying_ltp":24052.75,"moneyness":"ATM","lot_size":75,"option_token":"NFO:NIFTY29JUL2624100CE","greeks_model":{"type":"bsm_merton","r":0.065,"q":0.0125,"t_calendar_days":15,"day_count":"act/365","underlying_ref":"spot","vol_source":"ltp_implied"}}
> ```

**Rounding policy:**

| Field | Rounding |
|---|---|
| basis | 2 decimal places |
| delta, gamma | 10 decimal places |
| theta, vega | 4 decimal places |
| iv | full precision (decimal fraction; IV rank needs it) |
| change_frac | full precision (small number; 4dp loses signal) |
| [PLACEHOLDER] | [PLACEHOLDER] |

> Add rows for fields specific to this task. The T0 rows above are the baseline.

**Row Inventory (T1 lesson — required for every file-writing task):**

> One row per row_type this task writes. Completeness check: can you reconstruct
> the full daily NDJSON file structure from this table alone? If not, add rows.
> The T1-003 four-round schema churn is the justification for this table.

| row_type | Count per session | Key unique fields (not shared with other row_types) | Write trigger | Held / immediate | Logical ts vs write order |
|---|---|---|---|---|---|
| [PLACEHOLDER] | [e.g. one per bar = N bars per session] | [PLACEHOLDER] | [e.g. on each completed bar] | immediate | ts == bar timestamp, written in bar order |
| session_summary | exactly 1 | [fields present only on summary row] | session close (15:30) | held to 15:30 | ts = 15:30 bar; physically last row in file |

> Add a row for every row_type. Delete placeholder rows before submitting.
> Note: session_summary must appear here and must be present in every sample
> submission, even for mid-session oracle slices. See §5 of story-creation-instructions.md.

---

## Null / Fallback Behaviour

> List every condition that produces None for every nullable field.
> If a field can be None for three different reasons, list all three.
> Do not write "when the solver fails" — write every case that counts as failure.
>
> T0 example for iv (this is the complete list that was missing from T0 specs):

**iv → None when:**
1. Provider returns `iv = 0.0` (always a data error on NSE; convert before Pydantic validation)
2. `DTE = 0` (T = 0 causes division by zero; suppress on expiry day)
3. `DTE < 1` if using integer DTE and wanting intraday expiry-day suppression
4. `OI <= oi_threshold` (500 contracts; low-liquidity contract; IV unreliable)
5. IV solver returns None (convergence failure; price equals intrinsic or below)
6. `bid == 0 AND ask == 0 AND LTP is stale` (no tradeable price)

**delta, gamma, theta, vega → None when:**
- iv is None (any of the six conditions above)
- This cascade is enforced by the Pydantic model-level validator, not ad-hoc

> Fill this section for this task's nullable output fields. The T0 iv list is
> the baseline for any task that writes iv.

[PLACEHOLDER — null conditions for each nullable output field]

---

## Acceptance Criteria

> Binary pass/fail list. Each criterion is checkable with a number or a boolean
> assertion. "Looks correct" is not an acceptance criterion.
>
> Include the parity checks from the Verified Example section. They are not just
> documentation — they are tests.

- [ ] [PLACEHOLDER — one criterion per bullet; each checkable with a number]

> T0-007 example criteria (keep the ones relevant to this task):
> - [ ] Null-IV rule: input `iv=0.0` from provider → `OptionsRecord.iv is None`
> - [ ] DTE=0: all four Greeks are None when DTE=0
> - [ ] OI divisibility: `oi % lot_size == 0` for every written record
> - [ ] NDJSON format: every line is parseable by `json.loads()` independently
> - [ ] CE_delta + |PE_delta| = 0.9995 ± 0.001 for the verified scenario
> - [ ] theta_CE < theta_PE for the verified scenario (exact direction)
> - [ ] |IV_CE - IV_PE| < 5 basis points at same strike for the verified scenario
> - [ ] All timestamps in output: timezone-aware, UTC offset present
>
> Consumer task example — add this criterion if this task reads another task's output
> (T2 lesson: file-inventory check must be an explicit AC, not just an epic principle):
> - [ ] Output file contains only this task's Row Inventory row types: no [other_task]
>       row types appear in [this_task_output_path/] (grep-checkable before submission)

---

## Definition of Done

> A story is "done done" only when ALL of the following are true. The desk confirms this before the implementation review can grant APPROVED status. Partial completion is not done.

- [ ] All acceptance criteria pass — zero failing checks in the sample
- [ ] Part B oracle values filled by the developer
- [ ] Part C desk plausibility sign-off completed and signed
- [ ] Unit tests passing with literal pytest run log attached (or N/A documented with reason)
- [ ] Round-isolation verified — sample at current round=N/sample/ path is production writer output, not hand-assembled
- [ ] No open findings in the current review round — zero new findings, zero carry-forward

---

## Developer Pre-Submission Self-Review

> Developer fills this section before every sample submission — round 1 and every
> subsequent round. Do not submit a sample until all six items are checked.
> See §9 of story-creation-instructions.md for the rationale (T1 lessons).

- [ ] **Acceptance criteria row-by-row:** Every bullet in the Acceptance Criteria
      section above has been checked against the sample. Each criterion was found
      in the sample and verified to pass. Not a skim — each bullet was read and
      each corresponding sample row was found and checked.
- [ ] **Oracle arithmetic double-checked:** Every numeric value in any response
      document or round arithmetic table has been verified by two independent
      methods (hand/REPL + code output). No value was written without both methods
      agreeing. If there was a discrepancy, it was resolved before submission.
- [ ] **session_summary present:** The sample contains a session_summary row. If
      the oracle slice ends before 15:30, the row carries the mid-session state
      with a comment noting this.
- [ ] **Temporal-flag trace produced (if applicable):** For every flag with a
      temporal lifecycle, a bar-by-bar trace table (state inputs + expected flag
      presence at each bar) was produced and the sample was verified against it.
      Mark N/A if this story has no temporal-status flags.
- [ ] **Test run log attached:** If this story includes unit tests, the literal pytest output ("N passed in Xs") is attached in this same submission. "Tests written" is not the same as "tests run and passing." Mark N/A only if this story has zero unit tests.
- [ ] **Round-isolation verified:** The sample file at the round=N/sample/ path is
      the direct output from the production writer — not hand-assembled, not copied
      from a prior-round path, not a merged file containing other stories' row types.
      Verified before submitting this round.

> **Environment blockers:** If items 2, 5, or 6 above cannot be satisfied because
> of an environment constraint (Bash denied, test runner unavailable), raise it
> explicitly in this submission as a named process blocker with a request for
> resolution. Do not substitute a workaround and continue silently.

---

## Out of Scope

> List what this task explicitly does NOT cover. This prevents wrong assumptions
> and scope creep. Every task should have at least two entries.

- [PLACEHOLDER]
- Backtesting or historical data processing
- UI or visualisation of outputs
- [PLACEHOLDER — specific adjacent concerns this task does not handle]

---

## Open Questions

> Anything unresolved at time of writing. Flag these explicitly.
> Format: "Open question: [the question]. Developer should [recommend / flag /
> decide] before implementation begins."
>
> Example from T0:
> "Open question: the IV solver may fail on deep OTM strikes where the market
> price equals intrinsic value. Developer should document the failure rate on a
> typical NIFTY chain and flag if it exceeds 5% of contracts."

- [PLACEHOLDER — or "None — all decisions have been made"]
