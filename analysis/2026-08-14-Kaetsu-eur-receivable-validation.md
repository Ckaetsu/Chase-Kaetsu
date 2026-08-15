---
title: Stage 5 Validation - EUR 4,500,000 Receivable Hedge
author: Chase Kaetsu
date: 2026-08-14
---

# Stage 5 Validation: EUR 4,500,000 Receivable Hedge

## Part 1 — Independent LLM Execution

A fresh LLM conversation (no history, no coaching, no workbook results shared) was given exactly two documents — the Stage 2 spec (`docs/specs/2026-07-30-Kaetsu-eur-receivable-spec.md`) and the Stage 4 market-data memo (`data/2026-08-08-Kaetsu-market-data.md`) — and asked to compute all hedge outcomes and recommend a strategy. The full raw output is saved as an appendix: `analysis/2026-08-14-Kaetsu-llm-output-appendix.md`.

**Headline result:** the LLM's forward, put, call, and unhedged figures all matched the workbook exactly. Its money-market hedge did not — it used T_DAYS/365 as the period fraction instead of the ACT/360 convention (T_DAYS/360) the spec specifies. This is Part 2's central finding.

## Part 2 — Comparison & Hand Verification

### Comparison table: LLM vs. workbook, at three S_T points

| Strategy | S_T = 1.0983 (−5%) | S_T = 1.1561 (spot) | S_T = 1.2139 (+5%) | Discrepancy? |
|---|---|---|---|---|
| Forward — LLM | $5,295,600 | $5,295,600 | $5,295,600 | None |
| Forward — Workbook | $5,295,600 | $5,295,600 | $5,295,600 | — |
| Money-Market — LLM | $5,294,542 | $5,294,542 | $5,294,542 | **$1,251 low, all points** |
| Money-Market — Workbook | $5,295,793 | $5,295,793 | $5,295,793 | — |
| Put — LLM | $5,089,950 | $5,089,950 | $5,350,050 | None |
| Put — Workbook | $5,089,950 | $5,089,950 | $5,350,050 | — |
| Call — LLM | $4,843,350 | $5,103,450 | $5,103,450 | None |
| Call — Workbook | $4,843,350 | $5,103,450 | $5,103,450 | — |
| Unhedged — LLM | $4,942,350 | $5,202,450 | $5,462,550 | None |
| Unhedged — Workbook | $4,942,350 | $5,202,450 | $5,462,550 | — |

### Diagnosis of the one discrepancy

**What's wrong:** the LLM's money-market hedge is off by $1,251 (a constant offset across every S_T, since the money-market hedge doesn't depend on S_T at all — only on S0_in, R_USD, R_FC, and T_DAYS).

**Root cause:** the LLM divided by 365 instead of 360 in all three interest-rate steps. Since T_DAYS = 365 exactly, dividing by 365 collapses the period fraction to exactly 1.0, silently converting the "annual rate × ACT/360 period fraction" calculation into a flat annual-rate calculation. This looks reasonable on its face — 365 days is a year — which is exactly why it's a dangerous, easy-to-miss error rather than an obvious one.

**Whose error is it — LLM, workbook, or spec?** This is genuinely a **spec ambiguity**, not a pure LLM error, and not a workbook error. The spec states the rate basis ("ACT/360") in Section 4 (Assumptions & Constraints) — one place, in prose, separate from the Section 2 inputs table where R_USD and R_FC actually live. The Stage 4 market-data memo, which is the more recent and directly relevant document for live values, restates each rate with a source and timestamp but does not repeat "ACT/360" on the R_USD/R_FC rows themselves. An independent reader working primarily off the memo (since it holds the actual numbers to plug in) has to remember to go back to the older spec document for the basis convention. The workbook is correct because I built the day-count into the formula directly when constructing it in Stage 3, not because the spec made it foolproof to a new reader.

**How big is this in dollars, and does it matter?** $1,251 on a $5.3M receivable is about 0.024% — small, but not nothing, and it would compound across a larger book of trades or a longer tenor. More importantly: **the parity check still returns PASS under the LLM's wrong basis** (F_implied comes out to 1.176565 against F0_in = 1.1768, a gap of 0.000235, inside the $0.001 tolerance). That means the validation check built into the workbook would not have caught this specific error — a real finding, addressed in the retrospective below.

### Hand-verification table (≥3 outcomes, arithmetic shown, no Excel)

**1. Forward hedge:**
```
Proceeds_Forward = FC_AMT × F0_in
                  = 4,500,000 × 1.1768
                  = $5,295,600.00
```
Matches workbook exactly. ✓

**2. Money-market hedge, all three steps (ACT/360, correct basis):**
```
Step 1 — Borrow FC:
  Borrow_FC = FC_AMT / (1 + R_FC × T_DAYS/360)
            = 4,500,000 / (1 + 0.0225 × 365/360)
            = 4,500,000 / (1 + 0.0225 × 1.013889)
            = 4,500,000 / (1 + 0.022813)
            = 4,500,000 / 1.022813
            = 4,399,633.36 EUR

Step 2 — Convert at spot:
  USD_Now = Borrow_FC × S0_in
          = 4,399,633.36 × 1.1561
          = $5,086,416.13

Step 3 — Invest to settlement:
  Proceeds_MM = USD_Now × (1 + R_USD × T_DAYS/360)
              = 5,086,416.13 × (1 + 0.0406 × 1.013889)
              = 5,086,416.13 × (1 + 0.041164)
              = 5,086,416.13 × 1.041164
              = $5,295,792.80
```
Matches workbook exactly ($5,295,793, rounding). ✓ Does **not** match the LLM's $5,294,542 — confirming the LLM's error, not a workbook error.

**3. Put option, one outcome (S_T = 1.0983, below the strike):**
```
Proceeds_Put = FC_AMT × max(S_T, K_PUT) − FC_AMT × PREM_PUT
             = 4,500,000 × max(1.0983, 1.1561) − 4,500,000 × 0.0250
             = 4,500,000 × 1.1561 − 112,500
             = 5,202,450 − 112,500
             = $5,089,950.00
```
Matches workbook and LLM output exactly. ✓ (Below the strike, the floor holds — the put pays the difference regardless of how low S_T falls.)

## Part 4 — Spec Retrospective

**What the LLM got wrong or had to guess:** exactly one thing — the interest-rate day-count basis for the money-market hedge. It wasn't a guess made without any grounding; the spec does state ACT/360, but only once, in the Assumptions section, disconnected from the inputs table and not repeated in the Stage 4 memo. The LLM defaulted to treating T_DAYS = 365 as a literal one-year period fraction rather than pairing it with the /360 divisor stated elsewhere.

**What this reveals about the spec:** the named-range inputs table (§2) is complete on *values*, *units*, and *sources*, but not on *conventions that affect how those values are used in a formula*. A reader (human or LLM) building the calculation from §5 (Calculation Flow) alone, without holding §4 in working memory at the same time, will reasonably reach for the most literal interpretation of T_DAYS — and "365 days ÷ 365 days per year = 1" is a defensible literal reading if you're not actively cross-referencing the assumptions section.

**What v2 of the spec would say differently:** the rate basis should be stated redundantly, not just once. Specifically:
- The R_USD and R_FC rows in the §2 inputs table should include the basis directly in the unit column — e.g., "Annual % (ACT/360 — divide by 360, not 365)" — rather than relying on a separate assumptions paragraph.
- Every formula in §5 that uses T_DAYS should spell out the divisor explicitly in-line (e.g., "T_DAYS/360, not T_DAYS/365") rather than assuming the reader already carries the basis forward from §4.
- The §7 validation rules should tighten the parity tolerance, or add a second, independent check (e.g., "recompute F_implied using both /360 and /365; if the two differ by more than $0.0002, confirm which basis was actually used upstream") — because this incident showed the existing tolerance is wide enough to pass a real, dollar-relevant error silently.
- Future market-data memos (Stage 4-style documents) should restate the basis alongside each rate they report, not just alongside the original spec's inputs table, since the memo is often the document an independent reader relies on most directly for live numbers.

## Retrospective Addendum — R_FC Tenor Mismatch (post-review)

A second, independent gap surfaced after Treasury's Stage 4 review, separate from the rate-basis issue found in Part 1: I sourced R_FC from the ECB deposit facility rate — an overnight policy rate — while T_DAYS was tenor-matched to 365 days and R_USD was correctly a 1-year Treasury yield. I optimized for the wrong criterion: "freshest available rate" (the deposit facility is quoted daily) instead of "tenor-matched rate," and applied that inconsistently — the right standard on the dollar leg, the wrong one on the euro leg.

This is a more expensive mistake than the T_DAYS/365 basis slip found earlier: it moved F0_in by $0.0028 (1.1768 → 1.1796) and shifted both Proceeds_Forward and Proceeds_MM by $12,629.06 on a $4.5M notional — real money, not a rounding artifact. It's also a subtler class of error than the basis slip: it doesn't produce an obviously wrong number, just a *plausible* one that happens to be built on a tenor mismatch nobody would catch without specifically checking each rate's maturity against the model's horizon.

**What v2 of the spec would say differently:** Section 2's inputs table should require, for every rate input, an explicit statement of the instrument's own maturity alongside T_DAYS — not just a source and a value. A rule like "confirm the rate's stated maturity matches T_DAYS before accepting it as a Stage 4 substitute" would have caught this before it reached the workbook. "Freshest available" and "tenor-matched" can conflict, and the spec should say which one wins when they do.

This is not a case of "the spec was perfect" — it produced a real, quantifiable, if modest, dollar error in an independent reader's hands, and the fix is specific and actionable rather than a vague call for "more detail."
