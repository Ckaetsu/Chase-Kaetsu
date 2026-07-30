---
title: FX Hedging Model Specification - EUR 4,500,000 Receivable
author: Chase Kaetsu
date: 2026-07-30
---

# Model Specification: EUR 4,500,000 Receivable Hedge

## 1. Problem Statement

The firm holds a EUR 4,500,000 receivable settling in 365 days. The euro amount is contractually fixed; the USD value at settlement is not, since it depends on the EURUSD exchange rate at that future date. At the current spot rate (S0_in = 1.0800), the receivable is worth approximately $4,860,000. A 5% adverse move in EURUSD (spot falling to 1.0260) would reduce proceeds by roughly $243,000. This model exists to compare the forward, money-market, and put-option hedges against this exposure so the firm can select a strategy with a known, bounded outcome rather than an open position.

## 2. Inputs — Named-Range Contract

| Named Range | Placeholder Value | Unit | Stage-4 Data Source |
|---|---|---|---|
| FC_AMT | 4,500,000 | EUR | Confirmed contract amount (fixed, not market-sourced) |
| S0_in | 1.0800 | USD per EUR | Indicative — replaced with live spot from a market data provider (e.g. OANDA, Bloomberg) at Stage 4 |
| F0_in | 1.1034 | USD per EUR | Indicative — replaced with a quoted 1-year forward rate at Stage 4 |
| R_USD | 5.30% annual, ACT/360 | Annual % | Indicative — replaced with Federal Reserve H.15 release (1-year rate) at Stage 4 |
| R_FC | 3.10% annual, ACT/360 | Annual % | Indicative — replaced with ECB deposit facility rate at Stage 4 |
| K_PUT | 1.0800 | USD per EUR | Indicative — set at-the-money to spot; refined against live option quotes at Stage 4 |
| K_CALL | 1.0800 | USD per EUR | Indicative — set at-the-money to spot; refined against live option quotes at Stage 4 |
| PREM_PUT | 0.0250 | USD per unit FC | Indicative — replaced with a quoted put premium at Stage 4 |
| PREM_CALL | 0.0220 | USD per unit FC | Indicative — replaced with a quoted call premium at Stage 4 |
| T_DAYS | 365 | Days | Fixed by contract settlement date, not market-sourced |

All rate- and price-based inputs above are flagged **indicative — replaced with live market data at Stage 4**.

## 3. Tab Architecture

| Tab | Purpose |
|---|---|
| Cover | Scenario summary, firm name, exposure description, date prepared |
| Legend/Key | Color-coding key for input cells vs. formula cells; named-range glossary |
| Inputs | All ten named ranges, with values, units, and Stage-4 source notes |
| Forward | Forward hedge calculation (single-line proceeds) |
| Money Market | Three-step money-market hedge calculation, plus parity check |
| Options | Put and call payoff calculations across settlement spot scenarios |
| Sensitivity | S_T sensitivity table (0.95×S0_in to 1.05×S0_in in 1% steps) and comparison chart |
| Notes & Assumptions | Rate basis, transaction cost treatment, parity expectation, premium treatment |

## 4. Assumptions & Constraints

- **Rate basis:** All interest calculations use ACT/360 day-count convention.
- **Transaction costs:** No transaction costs or bid-ask spreads are modeled at this stage; all rates are treated as executable mid-market rates. This simplification will be revisited if Stage 4 data shows material spread impact.
- **Parity expectation:** The forward rate (F0_in) and the money-market hedge are expected to produce approximately equal USD proceeds, per covered interest rate parity. Any difference beyond rounding indicates a formula error, not a market inefficiency.
- **Premium treatment:** Option premiums (PREM_PUT, PREM_CALL) are treated as paid upfront and are not discounted or financed; they reduce net proceeds dollar-for-dollar regardless of the settlement outcome.

## 5. Calculation Flow

**Forward hedge:**

```
Proceeds_Forward = FC_AMT × F0_in
```

**Money-market hedge (three steps):**

```
Step 1 — Borrow FC today:
  Borrow_FC = FC_AMT / (1 + R_FC × T_DAYS/360)

Step 2 — Convert to USD at spot:
  USD_Now = Borrow_FC × S0_in

Step 3 — Invest USD to settlement:
  Proceeds_MM = USD_Now × (1 + R_USD × T_DAYS/360)
```

**Parity check:**

```
F_implied = S0_in × (1 + R_USD × T_DAYS/360) / (1 + R_FC × T_DAYS/360)
```

F_implied should approximately equal F0_in; a gap beyond rounding signals a formula defect.

**Put option hedge:**

```
Proceeds_Put = FC_AMT × max(S_T, K_PUT) − FC_AMT × PREM_PUT
```

Below K_PUT, the floor holds and the put pays the difference. Above K_PUT, the option expires worthless and the firm sells at the market rate, net of the premium already paid.

**Call option (for comparison/reference):**

```
Proceeds_Call = FC_AMT × min(S_T, K_CALL) − FC_AMT × PREM_CALL
```

## 6. Sensitivity Plan

S_T is varied from 0.95×S0_in to 1.05×S0_in in 1% increments (11 points total, including the base case). For each S_T value, the model computes Proceeds_Forward, Proceeds_MM, and Proceeds_Put side by side. One comparison chart plots all three strategies' proceeds against S_T, letting the CFO see visually where each hedge outperforms or underperforms the others, and confirming the put's floor and the forward/MM's flat, certain line.

## 7. Validation Rules (Check Figures)

- **Parity check:** |F_implied − F0_in| must be within $0.001 (rounding tolerance); a larger gap indicates a formula error in the money-market steps.
- **Forward proceeds** must be a flat, constant value across every row of the sensitivity table (it does not depend on S_T).
- **Continuity at the strike:** Proceeds_Put must transition smoothly through S_T = K_PUT with no jump or gap — at S_T exactly equal to K_PUT, the floor payoff and the market-rate payoff must produce the same value before the premium is subtracted.
- **Put proceeds** must never fall below FC_AMT × K_PUT − FC_AMT × PREM_PUT (the floor must hold at every S_T value).
- **No error cells** (#REF!, #DIV/0!, #N/A) anywhere in the workbook.
- **Every output cell must be a formula**, not a hardcoded number — outputs should update automatically if any input changes.

## 8. Outputs

| Output Name | Description |
|---|---|
| Proceeds_Forward | Final USD proceeds under the forward hedge |
| Proceeds_MM | Final USD proceeds under the money-market hedge |
| Proceeds_Put | Final USD proceeds under the put option hedge, by S_T scenario |
| Parity_Check_Result | Pass/fail flag comparing F_implied to F0_in |
| Sensitivity_Table | Full table of proceeds by strategy across the S_T range |
| Sensitivity_Chart | Comparison chart of all three strategies |
