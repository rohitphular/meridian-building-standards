# [EPIC-ID] — [Epic Name]

> Replace [EPIC-ID] with the next sequential ID (T0 = capture foundation, T1 = first
> signal epic, etc.). Replace [Epic Name] with a short capability phrase: "GEX Signal
> Engine", "IV Rank and Percentile", "Basis Signal."

---

## Status

> Choose one: `draft` | `in-review` | `approved` | `completed`

`draft`

---

## Purpose

> One paragraph. What trading capability does this epic deliver? What can the desk
> observe or act on after this epic is shipped that it could not before? Be concrete:
> "The desk will be able to read intraday GEX from NDJSON files and see which
> strike walls are accumulating dealer gamma exposure" is better than "GEX signals."

[PLACEHOLDER — describe the capability delivered]

---

## Business Context

> Why is this being built now? What decision or analysis is blocked without it?
> One to three sentences. This is not a feature justification — it is the context
> the developer needs to understand what "good enough" means.

[PLACEHOLDER — why this, why now]

---

## Depends On

> List prior epics this epic builds on. State what specifically is imported or
> assumed: schemas, computed fields, file formats.
>
> T0 example:
> - None (T0 is the foundation)
>
> T1 example:
> - T0 — OptionsRecord schema, NDJSON capture files at meridian/data/options/

[PLACEHOLDER — prior epic IDs and what is imported from them]

---

## Scope

### In Scope

> List what this epic covers. Be specific enough that the developer does not need
> to guess whether edge cases are in scope.

- [PLACEHOLDER]

### Out of Scope

> List what this epic explicitly does not cover. This prevents scope creep and
> wrong assumptions. "Backtesting and historical signal calculation" is out of
> scope for every capture/signal epic unless stated otherwise.

- [PLACEHOLDER]
- Backtesting and historical recalculation
- UI / visualisation layer

---

## Shared Conventions

> This section is inherited by every task in the epic without repetition. Get it
> right here and no task needs to repeat it. This section did not exist in T0 —
> its absence caused the same gaps to recur across eight tasks and nine rounds.
>
> Fill every subsection. Do not leave placeholders in this section when submitting
> for review.

### Model Parameters

> For any epic involving option pricing, IV, or Greeks. If this epic does not
> involve option math, write "N/A — no option pricing in this epic."

| Parameter | Value | Source | Notes |
|---|---|---|---|
| r | 0.065 | NSE 91-day T-bill, annualised decimal | Not US Fed funds (0.0525). Review annually. |
| q | 0.0125 | NIFTY trailing dividend yield, annualised decimal | BANKNIFTY: confirm separately |
| Model variant | BSM-Merton, b = r - q | Continuous-dividend equity form | NOT "GK" (that is FX). NOT Black-76 (b=0, futures). |
| Day-count | act/365 | Calendar days | T = DTE / 365. Never 252 trading days for BSM. |
| Delta convention | spot-delta, dV/dS | | CE in (0,1), PE in (-1,0) |

> T0 example: The parameters above are the T0 values. Override for a different
> index or a different rate environment. State the override and the reason.

[CONFIRM or OVERRIDE with reason]

### OI Convention

> NSE always reports OI in exchange units (number of contracts), never in lots.
> For NIFTY with lot_size=75: OI = 12,525 means 167 lots (12525 / 75).
> OI = 12,500 would be wrong — it is not divisible by 75, meaning the data was
> accidentally stored in lots rather than units.

- OI unit: exchange contracts (always a multiple of lot_size)
- oi_change basis: EOD delta vs prior session close — NOT intraday delta
- Validation rule: `oi % lot_size == 0` must be true for every record

[CONFIRM — changes to this require an explicit override with reason]

### Output Format

> Compact NDJSON: one JSON object per line, no whitespace between tokens, no
> pretty-printing. The Round-8 failure in T0 was caused by options files being
> serialised as multi-line pretty-printed JSON. A literal example below prevents
> this.

- File format: `.ndjson`
- Serialisation: compact JSON, no spaces after colons, no spaces after commas
- One record per line; file ends with a newline after the last record
- No enclosing array, no trailing comma

Literal example record (one line, all fields):

```
{"symbol":"NIFTY","instrument_type":"option","exchange":"NSE","expiry":"2026-07-29","dte":15,"strike":24100.0,"option_type":"CE","ts":"2026-07-14T09:35:00+05:30","ltp":185.0,"bid":183.0,"ask":187.0,"prev_close":170.0,"volume":32100,"oi":12525,"oi_change":-750,"oi_change_basis":"prior_session_eod","iv":0.0939526529256709,"delta":0.5076229027,"gamma":0.0008702202308,"theta":-7.8109573856,"vega":19.4386,"underlying_ltp":24052.75,"moneyness":"ATM","lot_size":75,"option_token":"NFO:NIFTY29JUL2624100CE","greeks_model":{"type":"bsm_merton","r":0.065,"q":0.0125,"t_calendar_days":15,"day_count":"act/365","underlying_ref":"spot","vol_source":"ltp_implied"}}
```

> Replace with a record appropriate to this epic's output type. This is not
> illustrative — it is the specification. A developer testing their output
> format checks against this line.

[PROVIDE literal example record for this epic's primary output type]

### Timestamps

- All timestamps: timezone-aware, UTC offset always present
- IST is +05:30. Example: `"2026-07-14T09:35:00+05:30"`
- No naive datetimes anywhere in the system
- File names use IST date and time (09:25 IST, not 03:55 UTC)
- Field type in Python: `datetime` with tzinfo; reject bare strings without offset

[CONFIRM]

### NSE Market Structure Constants

> State the constants that apply to this epic. Every threshold used in a flag or
> classification condition must appear here as a named row. In-prose-only thresholds
> are gaps — if a number is used in a flag condition, it lives in this table,
> referenced by name, not repeated inline. (T1 lesson: OR-width thresholds and
> gap-flat threshold needed a review round to add because they were prose-only.)

| Constant | Value | Notes |
|---|---|---|
| NIFTY lot_size | 75 | From NSE instrument spec; confirm before each series |
| BANKNIFTY lot_size | [PLACEHOLDER] | Confirm from provider |
| Weekly expiry weekday | Tuesday | Changed from Thursday in 2024 |
| Monthly expiry | Last Tuesday of month | Confirm for each contract month |
| ATM band (NIFTY) | ±0.5% of forward | NIFTY-specific; recalibrate for BANKNIFTY |
| OI threshold | 500 contracts | Below this: low_liquidity flag, iv=None |
| Gap tolerance | 7 min 30 sec | Interval > 7.5 min from prior snapshot triggers gap flag |
| [PLACEHOLDER — add every threshold used in any flag/classification in this epic] | | |

[CONFIRM values or OVERRIDE with reason; lot sizes must be confirmed from provider]

### Signal Computation Conventions

> Required for any epic with formulas, enums, or session-temporal flags.
> Write "N/A — no computation in this epic" only for pure schema/infrastructure epics.
>
> Three mandatory sub-sections:

#### Formulas

> Write every formula explicitly. Do not leave any formula inferable from an
> example arithmetic result alone. Name and reject non-obvious variants.
>
> Example format:
> ```
> σ = stddev(ltp_returns over trailing N bars) / sqrt(t)   [NOT / t]
> ROC = (ltp - ltp_N_bars_ago) / ltp_N_bars_ago * 100      [percentage, NOT decimal]
> VWAP = cumulative_pv / cumulative_volume                  [anchor resets at session open]
> ```

[PLACEHOLDER — list every formula used in this epic, with variant rejections]

#### Enum Taxonomies

> List ALL values for every categorical field. Both directions of any symmetric axis
> (up AND down, positive AND negative). All boundary conditions with explicit
> direction (>, >=, <, <=). No half-specified symmetric taxonomies.
>
> Example format:
> ```
> gap_type:
>   "full_gap_up"       open > prior_high          (no overlap at all)
>   "partial_gap_up"    open > prior_close AND      (gap but within prior range)
>                       open <= prior_high
>   "gap_flat"          |open - prior_close| / prior_close <= 0.15%
>   "partial_gap_down"  open < prior_close AND      (symmetric with partial_gap_up)
>                       open >= prior_low
>   "full_gap_down"     open < prior_low            (symmetric with full_gap_up)
> ```

[PLACEHOLDER — list every enum field with all values, boundaries, and directions]

#### Temporal-Status Flags

> For every flag that turns on and off over the session, state all three:
> (a) set condition, (b) exact clear timestamp or condition, (c) union vs
> per-variant if multiple variants exist.
>
> Example format:
> ```
> or_provisional:
>   Sets:   first bar of session (09:15) through the bar before both OR variants finalise
>   Clears: the bar on which the LAST variant finalises (union semantics — set if ANY
>           variant is still provisional; absent only when ALL variants have finalised)
>   Note:   NOT per-variant — one flag, fires if OR_15 or OR_30 is still open
> ```

[PLACEHOLDER — list every temporal-status flag with set/clear/union definition, or "None"]

---

### Output Schema Decisions

> Required for any epic with multiple writers or multiple row types per writer.
> These decisions belong here, at the epic level, before any task review begins.
> Deferring them to task review costs rounds.
>
> **Row inventory:** one row per (writer, row_type) pair.

| Writer | row_type | Count per session | Key unique fields | Write trigger | Per-variant or combined |
|---|---|---|---|---|---|
| [PLACEHOLDER writer] | [PLACEHOLDER row_type] | [e.g. exactly 1] | [fields that distinguish this row from others] | [immediate / held-to-session-end / held-until-EVENT] | [combined / one per variant] |

> Add a row for every (writer, row_type) pair in the epic. Completeness check:
> can you reconstruct the full NDJSON file structure from this table alone?
> If not, the table is incomplete.

[FILL COMPLETELY — no [PLACEHOLDER] rows when submitting for review]

---

## Task List

> Every task that makes up this epic. Fill in the table. Order rows by dependency
> (upstream tasks first). Every task must pass the one-line description test: its
> "What it builds" column should fit in one sentence.

| Task ID | Name | What it builds | Depends on |
|---|---|---|---|
| [EPIC-ID]-001 | [Task name] | [One sentence — what the output is] | None |
| [EPIC-ID]-002 | [Task name] | [One sentence] | [EPIC-ID]-001 |
| [EPIC-ID]-003 | [Task name] | [One sentence] | [EPIC-ID]-001, [EPIC-ID]-002 |

> T0 example:
> | T0-001 | Unified Pydantic Schemas | Pydantic v2 models for SpotRecord, FuturesRecord, OptionsRecord | None |
> | T0-002 | BSM IV Solver | bsm_iv() function solving implied volatility from option price | T0-001 |
> | T0-007 | Options Snapshot Writer | Full options chain capture integrating IV solver, Greeks, and file writer | T0-001 through T0-006 |

---

## End-to-End Scenario

> This section is the single highest-leverage addition compared to T0. The absence
> of a verified scenario caused at least five of the nine feedback rounds.
>
> Authorship split (mandatory — do not collapse these roles):
> - Part A: Desk fills inputs and acceptance bounds before implementation begins
> - Part B: Developer fills exact values from an independent oracle (QuantLib / scipy)
>           after implementation is complete, as part of the acceptance test
> - Part C: Desk validates domain plausibility of Part B values before marking approved

### Part A — Scenario Inputs (Desk fills before implementation)

> Specific values, not illustrative ranges. "NIFTY at approximately 24000" is not
> a scenario. Use a date that is a real trading day, a real DTE, specific prices.
>
> **Oracle authorship rule (T2 lesson):** Any formula result involving a transcendental
> (exp(), log()) must be produced by a captured code run, not hand-approximated. Record
> the unrounded value. A hand-approximated transcendental on a rounding boundary becomes
> an 11-location correction table if the rounding is wrong.

```
Symbol:       [PLACEHOLDER e.g. NIFTY]
Date/time:    [PLACEHOLDER e.g. 2026-07-14T09:35:00+05:30]
Spot LTP:     [PLACEHOLDER e.g. 24052.75]
Futures LTP:  [PLACEHOLDER e.g. 24118.40]  (near-month)
Expiry:       [PLACEHOLDER e.g. 2026-07-29]
DTE:          [PLACEHOLDER — calendar days from scenario date to expiry]
Strike:       [PLACEHOLDER e.g. 24100]     (nearest to futures LTP)
CE LTP:       [PLACEHOLDER e.g. 185.0]
PE LTP:       [PLACEHOLDER e.g. 168.5]
CE bid/ask:   [PLACEHOLDER e.g. 183.0 / 187.0]
PE bid/ask:   [PLACEHOLDER e.g. 167.0 / 170.0]
CE OI:        [PLACEHOLDER — must be a multiple of lot_size]
PE OI:        [PLACEHOLDER — must be a multiple of lot_size]
lot_size:     [PLACEHOLDER e.g. 75]
```

### Part A — Acceptance Bounds (Desk fills before implementation)

> These are the domain-plausibility bounds the desk knows from experience. They
> do not need to be exact. The oracle (Part B) provides exact values. These bounds
> are the qualitative sanity check.
>
> T0 example (from instruction-for-trader-requirement.md §G2):
> "ATM NIFTY, 15 DTE, IV ~9.4%:
>  - CE delta should be near 0.50–0.51
>  - CE/PE vega should be equal at the same strike (same-strike vega equality)
>  - theta_CE should be more negative than theta_PE (call decays faster on NSE)
>  - CE_delta + |PE_delta| should be approximately 0.9995, not 1.0"

Per task output:

| Task | Field | Acceptance bounds | Domain reasoning |
|---|---|---|---|
| [EPIC-ID]-00X | [field] | [e.g. 0.50–0.51] | [e.g. near-ATM call delta is always near 0.5] |
| [EPIC-ID]-00X | [field] | [e.g. always negative] | [e.g. call theta is always negative for long options] |

> Add rows for every output field that has a plausibility check. The bounds
> should be ranges the desk is confident in from experience — not "anything is
> possible." If you cannot state bounds for a field, that is itself a signal
> that the field definition may be ambiguous.

### Part C — Desk Plausibility Sign-Off (Desk fills after reviewing Part B)

> Desk: review Part B values before the epic is marked approved. Confirm direction
> and domain plausibility for the scenario. Do not recompute — assess plausibility.
> Sign off by filling the table below. This is a hard approval gate.

| Output | Domain plausibility check | Pass / Fail |
|---|---|---|
| [PLACEHOLDER] | [PLACEHOLDER — e.g. "put-call parity closes within 1 index point"] | [PASS/FAIL] |

Desk sign-off: [DESK INITIALS / DATE — required before epic status → `approved`]

### Part B — Oracle Values (Developer fills during implementation)

> Developer: run QuantLib or scipy against the Part A inputs and record exact
> values here. These become the acceptance benchmarks.

```
IV (CE):    [DEVELOPER FILLS — e.g. 0.0939526529]
IV (PE):    [DEVELOPER FILLS — e.g. 0.0941203714]
delta (CE): [DEVELOPER FILLS — e.g. 0.5076229027]
delta (PE): [DEVELOPER FILLS — e.g. -0.4918756201]
gamma:      [DEVELOPER FILLS — e.g. 0.0008702202]
theta (CE): [DEVELOPER FILLS — e.g. -7.8109573856]
theta (PE): [DEVELOPER FILLS — e.g. -4.3647283124]
vega:       [DEVELOPER FILLS — e.g. 19.4386]
```

### Second Scenario Bar — Suppression Paths (T2 lesson)

> If the headline Part A scenario cannot exercise a binding suppression rule
> (DTE=0 whole-row suppression, expiry-day afternoon cross-read suppression,
> low-liquidity suppression, etc.), provide a second named scenario bar here
> with desk-signed expected output. Unit tests alone are not a desk approval gate
> for suppression behaviour. Leave this section blank only if the headline scenario
> exercises ALL suppression paths.

[PLACEHOLDER — "None — headline scenario exercises all suppression paths" OR
provide: condition name, scenario inputs, and expected output (desk-signed)]

### Cross-Task Parity Checks

> Relationships between outputs from different tasks. These catch errors that
> individual task tests miss. Include tolerance bands — cross-leg identities
> are approximate, not exact, because CE and PE carry per-leg IVs.

| Check | Formula | Tolerance | Notes |
|---|---|---|---|
| CE/PE delta parity | CE_delta + \|PE_delta\| = exp(-q * T) | ±0.002 | NOT 1.0; NOT exp(-r*T). Approximate because per-leg IVs differ. |
| Same-strike IV parity | \|IV_CE - IV_PE\| | < 5 basis points | Same underlying, same expiry, same strike |
| Theta asymmetry direction | theta_CE < theta_PE | exact direction | Call decays faster always (r > q on NSE always) |
| Gamma parity | \|gamma_CE - gamma_PE\| / gamma_CE | < 1% | Approximately equal, not identical |
| OI divisibility | oi % lot_size == 0 | exact | Every record |

> Add rows for this epic's specific cross-task checks. The four rows above apply
> to any epic with Greek computation — keep them and add epic-specific rows below.

[PLACEHOLDER — add epic-specific cross-task parity checks]

---

## Epic-Level Acceptance Criteria

> Binary pass/fail conditions. Every criterion must be checkable with a number
> or a boolean assertion — "looks correct" is not an acceptance criterion.

- [ ] All tasks in the task list pass their individual acceptance criteria
- [ ] End-to-end scenario Part B values match oracle within stated tolerances
- [ ] Cross-task parity checks all pass
- [ ] All OI values in output files divisible by lot_size
- [ ] All timestamps in output files are timezone-aware (no naive datetimes)
- [ ] Output files are compact NDJSON (each line parseable by `json.loads()` independently)
- [ ] [PLACEHOLDER — epic-specific criteria]

---

## Known Constraints

> Technical or domain constraints the developer must respect. Examples: provider
> rate limits, NSE market hours, known data quality issues with a specific series.

- [PLACEHOLDER]

---

## Open Questions

> Anything that is genuinely unresolved at time of writing. Flag these explicitly
> rather than leaving them implicit. A conversation before implementation is one
> email; a conversation during review is one round of rework.
>
> Format: "Open question: [the question]. Developer should [recommend / flag /
> decide] before implementation begins."

- [PLACEHOLDER — or "None — all decisions have been made"]

---

## Epic Handoff Record

> Developer fills this section within 24 hours of receiving the epic spec.
> No task implementation begins until the epic-level gate is cleared.
> See §7 of requirement-epic-instructions.md for the full gate rules.

| Epic-level section | Status | Missing or incomplete items |
|---|---|---|
| Shared Conventions — all subsections filled, no [PLACEHOLDER] remaining | COMPLETE / INCOMPLETE | |
| End-to-End Scenario Part A — specific numeric inputs (not illustrative ranges) | COMPLETE / INCOMPLETE | |
| End-to-End Scenario Part A — acceptance bounds are concrete ranges per field | COMPLETE / INCOMPLETE | |
| End-to-End Scenario — all transcendental oracle values have a captured code-run backing (not hand-approximated) | COMPLETE / INCOMPLETE / N/A | |
| End-to-End Scenario — second scenario bar provided for every binding suppression path the headline cannot exercise | COMPLETE / INCOMPLETE / N/A | |
| Signal Computation Conventions — completeness sweep: every temporal flag has set+clear+per-variant; every session-summary field has a formula; every literal example record is consistent with Part A bounds | COMPLETE / INCOMPLETE | |
| Cross-Task Parity Checks — tolerance bands present | COMPLETE / INCOMPLETE | |
| All tasks pass one-line description test | COMPLETE / INCOMPLETE | |
| No [PLACEHOLDER] token remaining anywhere in the epic | COMPLETE / INCOMPLETE | |

Gate result: [CLEARED — task implementation may begin | RETURNED — sent back to desk on DATE with gaps listed above]
