# Capital Allocation: Cash or Portfolio Deployment

Use `$ARGUMENTS` to decide whether available capital should remain in cash or be allocated to one or more positions in an existing portfolio, and in what proportions.

The first valid option is always **keep the capital in cash**. Do not maximize yield, force deployment, or assume that investing 100% is desirable. This workflow supports research and decision discipline; it is not personalized investment advice or a guarantee of returns.

## Input

The minimal command is:

```text
/capital-allocation
```

Optional parameters:

```text
/capital-allocation --candidates "Itochu, Verizon, Coca-Cola"
/capital-allocation --mode income
/capital-allocation --external-capital 500
```

- `--candidates` limits the allocation universe to the comma-separated names, tickers, or `Cash` supplied.
- `--mode` accepts `long_term_growth`, `income`, or `balanced`; default: `balanced`.
- `--external-capital` is a new contribution not included in the portfolio report. Interpret it in the portfolio base currency unless the user explicitly states another currency.

Do not require the user to re-enter portfolio cash, weights, currency, or report path. Detect the latest `portfolio-review` automatically: prefer `reports/portfolio-latest.md` when it exists, otherwise find the most recent portfolio-review report in `reports/`. Extract its cash balance, portfolio value, position weights, target weights, limits, and base currency.

By default, the report's cash balance is the available capital and is already included in portfolio value. If `--external-capital` is supplied:

```text
available capital = reported portfolio cash + external capital
post-contribution portfolio value = reported portfolio value + external capital
```

Only ask for confirmation when:

- the user states a capital amount that differs from reported cash without using `--external-capital`;
- the user mentions an external contribution without a clear amount or currency;
- no usable portfolio-review can be found;
- the latest report is stale or does not expose cash, weights, portfolio value, or currency.

Do not invent missing prices, exchange rates, weights, dividends, yields, fees, tax treatment, target ranges, or report availability. Ask explicitly for missing decision-critical inputs. If they cannot be obtained after repository discovery, retain the capital in cash and list the exact missing evidence.

## Candidate Scope

If `--candidates` is provided, or the user names a candidate in a conversational reply, analyze only those named companies and `Cash`. Set the selection origin to `Explicit user choice`. Use every other holding solely for portfolio constraints; do not justify why it was excluded and do not assign it a status or rank.

If no candidate is explicit, derive the universe from the latest `portfolio-review` and set the selection origin to `Automatic`:

1. first include lines marked `Increase` or an unambiguous localized equivalent;
2. then include lines marked `Hold / Increase` or an unambiguous localized equivalent;
3. add companies with a documented trigger newer than the portfolio review;
4. always include `Cash` as an alternative;
5. never treat the entire portfolio as the allocation universe.

Before concluding that the review approves no candidate, scan report filenames and contents for newer `investment-checklist`, `thesis-tracker`, `earnings-review`, `investment-research`, or `investment-team` evidence. Add a company only when the report explicitly supports one of the documented triggers below; a recent file alone is not a trigger.

Eliminate a company from the eligible ranking when the portfolio review already proves that it is above its target, at its upper bound, would exceed the upper bound after buying one whole share, breaches a sector or income-pocket cap, or would breach minimum cash. A newer trigger never overrides these strategic constraints. Keep the company in the candidate-status table with principal status `ALLOCATION BLOCK`; any missing reports are secondary future work. Use a reliable dated price and exchange rate for the one-share test; if either is unavailable, do not invent the result. When no company remains after both portfolio and newer-trigger discovery, return exactly:

```text
Decision: HOLD CASH
Reason: no candidate approved by the portfolio-review.
```

## Repository Report Discovery

Before marking any report `missing`, search recursively from the repository root. Use the file capabilities available in the current client; with a shell, prefer:

```text
rg --files reports
rg -il --glob '*.md' '<candidate name or ticker>' reports
```

For each candidate:

1. build search keys from the supplied name, full legal name, ticker, portfolio label, known localized name, and obvious spelling variants;
2. compare case-insensitively after treating spaces, hyphens, underscores, punctuation, and corporate suffixes as non-distinguishing;
3. search paths first, then Markdown content for tickers or aliases not present in filenames;
4. include root-level reports and company subdirectories;
5. search for `portfolio-review`, `investment-research`, `investment-checklist`, `income-investment`, `thesis-tracker`, `quality-screen`, and, when present, `investment-team` and `earnings-review` evidence;
6. list every matched path before deciding what is missing;
7. if a short ticker or alias produces ambiguous matches, read the candidate headings and ask for confirmation rather than guessing.

Never claim that a report is absent merely because the user did not pass its path. A localized name cannot be inferred without a shared alias, ticker, portfolio label, or report content; mark such a match `ambiguous`, not `missing`, until checked.

## Position in the Workflow

Use prior work instead of reproducing it:

```text
portfolio-review
    ↓
investment-research
    ↓
income-investment, when relevant
    ↓
investment-checklist
    ↓
capital-allocation
    ↓
thesis-tracker after a purchase or reinforcement
```

| Need | Required source |
|---|---|
| Portfolio weights, cash target, concentration limits | `portfolio-review` |
| Business quality, moat, management, intrinsic value | `investment-research` |
| Dividend coverage, durability, debt and income role | `income-investment` when relevant |
| Pre-purchase quality, discipline and margin-of-safety gates | `investment-checklist` |
| Existing thesis health and red lines | `thesis-tracker` when required by the rules below |

Do not redo full fundamental research. Read the discovered reports, record their paths, dates and conclusions, detect conflicts or gaps, and distinguish missing documentation from a documented price or allocation block.

## Required Evidence by Candidate Type

For a new candidate, normally require current `investment-research`, `investment-checklist`, and portfolio context. Require `income-investment` only when the dividend is central to the thesis or role. Create `thesis-tracker` after a purchase; its absence alone does not block a new candidate.

For an existing position, recent research and checklist evidence may provisionally support a reinforcement decision. Require a current `thesis-tracker` after a new purchase, after a significant reinforcement, for a core position, or when the thesis is old, ambiguous, or disputed. Do not block a small allocation solely because no formal tracker exists yet.

For `income-satellite`, `cyclical-income`, high-yield, highly leveraged, or dividend-led candidates, require a current `income-investment` report. For other income candidates, mark it `not required` only when research and checklist evidence already cover dividend safety sufficiently.

## Freshness Rules

- `portfolio-review`: current only if it reflects the latest portfolio transaction and cash balance;
- price: current session or latest available market close, clearly timestamped;
- `investment-research`: remains usable until a major event, material thesis change, or excessive age makes its conclusions unreliable;
- `investment-checklist`: must be reconciled with the current price and its stated buy zone or valuation assumptions;
- `income-investment`: must incorporate the latest available results, dividend decision, debt and refinancing information;
- `thesis-tracker`: update after purchase, significant reinforcement, or a material fundamental change.

An existing but outdated document is `stale`, never `missing`. Display `STALE DATA`, state what must be refreshed, and do not silently reuse a stale price-sensitive conclusion.

## Documented Allocation Triggers

The `portfolio-review` remains the primary strategic source, but it is not the only allocation trigger. Accept exactly these documented triggers:

1. the portfolio review explicitly recommends an increase;
2. a more recent `investment-checklist` explicitly concludes `BUY`, purchasable, or adequate margin of safety at the current price;
3. a reliable current price is at or below a buy zone recorded in an existing report;
4. a new or updated thesis explicitly documents a material structural improvement, durable quality gain, major debt reduction, or improved capital allocation;
5. a recent results report explicitly documents a material improvement in results, sustainable FCF, dividend growth, or balance-sheet quality.

Compare report dates and show `Portfolio Review date`, `Latest analysis used`, and `Selected trigger`. Prefer the portfolio-review trigger when it remains current; a newer trigger may complement it or add a candidate. Never infer a trigger from recency, create a buy zone, or treat a positive event as sufficient by itself: every candidate still passes allocation, price, quality, and documentation gates. If sources conflict, retain the more conservative conclusion unless the newer report explicitly resolves the older blocker.

## Decision Principles

Apply this priority order:

1. capital preservation;
2. business quality;
3. margin of safety;
4. consistency with portfolio target weights;
5. long-term capital growth;
6. durable income;
7. dividend timing only as a secondary consideration.

Use Duan Yongping for business simplicity and excellence, Buffett for intrinsic value and margin of safety, Munger for inversion and vetoes, Li Lu for long-term economic certainty, and `income-investment` for dividend durability.

Dividend yield never compensates for a weak thesis, excessive debt, poor coverage, bad capital allocation, or insufficient margin of safety.

## Research and Calculation Discipline

1. Run `date` before any current-data research and put the data cutoff date in the report header.
2. Record the date of every price and source report. Flag stale data and explain its decision impact.
3. Separate **Verified fact**, **Estimate**, **Assumption**, and **Analytical judgment**.
4. Cross-check decision-critical financial data with at least two independent sources when new verification is needed.
5. Use `python3 tools/financial_rigor.py calc --expr '<expression>'` for capital, fees, income, cash and weight arithmetic. Use its valuation commands when valuation figures require verification.
6. Round proposed quantities down to whole shares unless the broker explicitly supports fractional shares and the user requests them.
7. Recalculate all final weights using the post-trade portfolio value and remaining cash. Include fees when known.
8. If no reliable price is available, do not calculate shares. Show `Not calculable`.
9. Avoid false precision: use ranges when inputs or intrinsic values are ranges.

## Roles and Sizing

Classify every current position and candidate as exactly one role:

- `core-compounder`
- `core-income`
- `cyclical-income`
- `income-satellite`
- `opportunistic`
- `watchlist`
- `exit-candidate`
- `cash`

Apply portfolio-review limits first. Within those limits:

- core convictions may receive the largest weights only with a current, clear thesis and strong evidence;
- `core-income` must pass strict coverage, debt, durability, and valuation checks;
- `cyclical-income`, `income-satellite`, and `opportunistic` positions must remain smaller than core convictions;
- `watchlist`, `exit-candidate`, and positions without a clear thesis receive no new capital;
- high yield alone can never promote a position to a core role.

Separate the portfolio into:

- **Conviction pocket**: long-term capital growth and durable or growing dividends;
- **Income pocket**: higher current income used mainly for reinvestment, with a configurable aggregate cap, per-position cap, strict coverage test, debt monitoring, explicit exit rule, and smaller sizing than core convictions.

When the user permits some dividends to be consumed, separate projected reinvestable and consumable income and enforce the stated consumption maximum. Do not assume that any dividend will be consumed.

If an income-pocket cap or line cap is required but unavailable, do not allocate to that pocket.

## Evidence Extraction and Normalized Statuses

Read every matched report and extract explicit evidence when present:

- research verdict and business-quality conclusion;
- checklist verdict and failed gate;
- dated current price, buy zone, intrinsic-value range, and maximum price;
- `buy`, `hold`, `do not add`, `watch`, `avoid`, `reduce`, or `sell` conclusion;
- current weight, target range, portfolio role, and concentration limits;
- main risks and invalidation conditions;
- dividend amount, yield, payout, FCF coverage, growth, debt, maturities, income-pocket cap, and exit rule.

Keep the source path and heading or short evidence excerpt for each extracted conclusion. If reports conflict, show the conflict and apply the more conservative conclusion until it is resolved.

Give every candidate one principal status, zero or more secondary statuses, and one next action. Determine the principal blocker in this order:

1. `ALLOCATION BLOCK` — weight, whole-share sizing, sector, factor, income-pocket, fee, or minimum-cash constraints prevent allocation. Use `DOCUMENTED — ALLOCATION BLOCK` when the evidence is otherwise sufficient. Missing reports are secondary `DOCUMENTATION MISSING` or future documentary work, never the principal blocker here.
2. `PRICE BLOCK` — price exceeds the buy zone or maximum price, margin of safety is inadequate, or the checklist fails on price. Use `DOCUMENTED — PRICE BLOCK` when the evidence is sufficient.
3. `QUALITY BLOCK` — business quality, thesis, management, balance sheet, debt, dividend safety, or capital-allocation discipline fails.
4. `DOCUMENTATION MISSING` — genuinely required evidence remains absent after repository discovery. Name each missing item: research, checklist, income report, current price, current weight, target weight, or thesis.

Use `HOLD — DO NOT ADD` when an existing position is acceptable to retain but reports explicitly say hold, do not reinforce, or keep small. Use `REDUCE / EXIT CANDIDATE` when the review or research explicitly says reduce, exit, sell, thesis-invalid, role-less, or inferior on opportunity cost. Use `ELIGIBLE` only when business, quality, valuation, portfolio weight, cash, role, debt, and applicable dividend requirements all pass.

`HOLD CASH` is the portfolio decision when no `ELIGIBLE` candidate deserves capital. A documented candidate can be a good business, a good dividend payer, or a reasonable existing holding without being eligible for reinforcement.

## Capital Deployment Status

Assign exactly one portfolio readiness state without replacing the existing decision or candidate statuses:

- `READY TO DEPLOY`: at least one candidate has a documented trigger and passes quality, price, allocation, and documentation gates.
- `WAITING FOR TRIGGER`: the portfolio is healthy and at least one credible candidate exists, but an observable condition such as price, refreshed checklist, results, thesis evidence, or weight capacity is not yet satisfied.
- `NOT READY`: deployable cash or documentation is insufficient, concentration is structurally excessive, or no credible candidate exists.

For `WAITING FOR TRIGGER` and `NOT READY`, produce short, concrete `Unlock Conditions` supported by existing evidence, such as a documented maximum price, named report refresh, specified result, or weight threshold. Never invent one. When none is documented, write exactly: `No documented trigger currently available.`

## Execution Workflow

### A. Establish Portfolio Context and Cash

Calculate:

```text
post-trade cash = reported portfolio cash + external capital - proposed purchases - fees
post-trade portfolio value = reported portfolio value + external capital
post-trade cash weight = post-trade cash / post-trade portfolio value
```

Compare both with the cash target or minimum from `portfolio-review`. If the minimum would be breached, stop: `HOLD CASH`.

### B. Select and Pre-filter Candidates

Apply explicit or automatic selection rules. In automatic mode, scan for newer documented triggers before deciding that the portfolio review offers no candidate. Then test allocation feasibility and record allocation failures as principal blockers.

### C. Discover and Read Candidate Evidence

For each in-scope candidate, run repository report discovery, read the matches, build the evidence map, and label each document `found`, `missing`, `not required`, or `stale`. Do this before any missing-document conclusion.

### D. Select the Trigger, Classify Candidates, and Measure Target Gaps

For each candidate, compare the portfolio-review date with the latest relevant evidence and name the selected trigger. No trigger means no allocation, even when the candidate remains worth monitoring.

For each eligible position:

```text
gap = relevant target weight - current weight
```

When the target is a range, do not automatically use its midpoint. Explain whether the proposed weight aims at the lower bound, midpoint, or upper bound based on evidence quality, conviction, cyclicality, downside risk, and portfolio concentration.

### E. Rank Eligible Uses of Capital

Rank only `ELIGIBLE` candidates. Compare each with every other eligible candidate and with holding cash. Evaluate:

- business quality and simplicity;
- long-term certainty;
- moat;
- management and capital allocation;
- valuation and margin of safety;
- balance sheet and permanent-loss risk;
- dividend contribution, coverage and growth;
- diversification benefit;
- opportunity cost;
- final position size;
- transaction fees.

Do not collapse these dimensions into a mechanically precise numeric score. Rank by evidence-backed judgment after gates.

### F. Apply Inversion

For every proposed allocation answer:

> Why could this allocation be a mistake?

Give at least three distinct reasons, one order-cancellation signal, a maximum advisable price when supported by a current valuation report, and explicit wait conditions. If no defensible maximum price exists, show `Not calculable` and do not place the order.

### G. Select One Decision

Return exactly one portfolio-level decision:

- `HOLD CASH`
- `ALLOCATE TO ONE POSITION`
- `ALLOCATE TO MULTIPLE POSITIONS`
- `STAGED ALLOCATION`
- `WAIT FOR PRICE`
- `REJECT ALL CANDIDATES`

Never force allocation of all available capital. Compare the selected decision explicitly with retaining 100% in cash.

## Dividend Calendar Rules

When reliable data exist, use the dividend calendar only to estimate annual income, map payment months, identify months with no income, and show the projected income change.

State explicitly:

> Buying a company only to fill a month without dividends is prohibited if quality or valuation fails the filters.

A nearby ex-dividend date never justifies a rushed purchase and is not a free gain.

## Required Report Format

Save the report to `reports/capital-allocation-{YYYYMMDD}.md`. Use these headings exactly once:

## Capital Deployment Status

```text
Status: READY TO DEPLOY | WAITING FOR TRIGGER | NOT READY
Portfolio Review date:
Latest analysis used:
Selected trigger:
Reason:
```

For `WAITING FOR TRIGGER` and `NOT READY`, follow it with `## Unlock Conditions` and a short evidence-backed list. Omit that heading for `READY TO DEPLOY`.

## 1. Executive decision

```text
Decision:
Existing cash:
External capital:
Total deployable cash:
Capital allocated:
Cash retained:
Confidence:
```

The external contribution is zero when absent. For example, `1,346.58 EUR + 200 EUR = 1,546.58 EUR` of total deployable cash. Also show the data cutoff date and constraints used: cash target, weights, sector/factor limits, income-pocket cap, fees, mode, assumptions, and stale-data warnings.

## 2. Candidate selection origin

Write `Mode: Automatic` or `Mode: Explicit user choice`.

## 3. Evidence map

| Candidate | Research | Checklist | Income | Thesis | Portfolio target |
|---|---|---|---|---|---|

Use only `Found`, `Missing`, `Not required`, or `Stale`, and list matched report paths directly below the table.

## 4. Candidate status

| Candidate | Principal status | Secondary blockers | Next action |
|---|---|---|---|

## 5. Eligible candidate ranking

| Rank | Candidate | Proposed amount | Estimated quantity | Final weight | Reason |
|---:|---|---:|---:|---:|---|

Show only `ELIGIBLE` candidates. If none qualify, write exactly:

```text
No eligible candidates.
HOLD CASH.
```

## 6. Documented but blocked candidates

| Candidate | Block type | Current conclusion | Unlock condition |
|---|---|---|---|

Include price, quality and allocation blocks, `HOLD — DO NOT ADD`, and `REDUCE / EXIT CANDIDATE`. When allocation is already impossible, list missing reports as `Future documentation work`, not as the primary refusal reason.

Add an `Income impact` subsection when at least one in-scope candidate is income-led. Extract available annual dividend, yield, payout, FCF coverage, debt, maturities, cut history, dividend growth, role, maximum weight, and exit rule. Show income before and after, incremental income, yield on allocated capital, dividend quality, payout safety, and income-pocket weight. Distinguish a sound dividend, a sound stock to reinforce, and a sound existing holding that should receive no new capital.

## 7. Cash comparison

Compare investing, retaining cash, and waiting for a stated price. Cash wins when margin of safety, documentation, allocation capacity, income safety, or opportunity cost is inadequate.

## 8. Execution plan

Only provide orders when allocation is selected. Give amount, whole-share estimate, indicative order type, price limit, non-execution conditions, and when to rerun `portfolio-review`. Order instructions are indicative, not guarantees.

Otherwise write:

```text
No order to place.
Re-run the skill when an unlock condition is met.
```

## 9. Journal entry

| Date | Capital available | Decision | Allocation | Cash retained | Reports used | Main justification | Next step |
|---|---:|---|---|---:|---|---|---|

## 10. Next analyses

List only genuinely missing or stale reports. Separately list `thesis-tracker` actions required after a purchase, significant reinforcement, core-position designation, or material thesis change.

End with a limitations statement and reminder that the report is decision support, not a return guarantee.

## Usage Examples

- `/capital-allocation`: use reported cash, default to `balanced`, select review candidates, then check newer documented triggers before returning `HOLD CASH`.
- `/capital-allocation --external-capital 500`: add 500 units to reported cash and evaluate the full mobilizable cash balance.
- `/capital-allocation --candidates "TotalEnergies"`: analyze only TotalEnergies and Cash, with explicit-user selection origin.

## Illustrative Output Example

```markdown
# Capital Allocation Report

## Capital Deployment Status
Status: WAITING FOR TRIGGER
Portfolio Review date: 2026-07-15
Latest analysis used: Itochu checklist dated 2026-07-18
Selected trigger: None; documented buy zone not reached
Reason: credible candidates exist, but none passes every gate today

## Unlock Conditions
- Itochu enters its documented buy zone.
- Verizon receives a newer documented add conclusion.

## 1. Executive decision
Decision: HOLD CASH
Existing cash: 1,346.58 EUR
External capital: 200.00 EUR
Total deployable cash: 1,546.58 EUR
Capital allocated: 0.00 EUR
Cash retained: 1,546.58 EUR
Confidence: High

## 2. Candidate selection origin
Mode: Explicit user choice

## 3. Evidence map
| Candidate | Research | Checklist | Income | Thesis | Portfolio target |
|---|---|---|---|---|---|
| Itochu | Found | Found | Not required | Not required | Found |
| Verizon | Found | Found | Found | Not required | Found |
| Coca-Cola | Missing | Missing | Not required | Not required | Found |

## 4. Candidate status
| Candidate | Principal status | Secondary blockers | Next action |
|---|---|---|---|
| Itochu | DOCUMENTED — PRICE BLOCK | None | Wait for the documented buy zone |
| Verizon | HOLD — DO NOT ADD | None | Monitor FCF and debt |
| Coca-Cola | DOCUMENTATION MISSING | Research, checklist | Complete both analyses |

## 5. Eligible candidate ranking
No eligible candidates.
HOLD CASH.

## 6. Documented but blocked candidates
| Candidate | Block type | Current conclusion | Unlock condition |
|---|---|---|---|
| Itochu | Price | High-quality business, price too high | Price enters the buy zone |
| Verizon | Hold | Covered dividend, do not add | New favorable documented conclusion |

## 7. Cash comparison
Cash preserves optionality; no candidate compensates for its risk at current prices and constraints.

## 8. Execution plan
No order to place.
Re-run the skill when an unlock condition is met.

## 9. Journal entry
| Date | Capital available | Decision | Allocation | Cash retained | Reports used | Main justification | Next step |
|---|---:|---|---|---:|---|---|---|
| 2026-07-19 | 1,546.58 EUR | HOLD CASH | None | 1,546.58 EUR | Portfolio and matched reports | No eligible candidate | Wait or complete analyses |

## 10. Next analyses
- Coca-Cola: investment-research and investment-checklist.
```

## Release Audit

After saving a real report:

```bash
python3 tools/report_audit.py extract --report reports/capital-allocation-{YYYYMMDD}.md
# Verify extracted items against reliable sources, then:
python3 tools/report_audit.py verdict --results '<verified JSON>' --report reports/capital-allocation-{YYYYMMDD}.md
```

Fix failed items and repeat the audit. Keep unresolved gaps visible and reduce confidence instead of filling them with assumptions.
