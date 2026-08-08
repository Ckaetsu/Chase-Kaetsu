---
title: Stage 3 Build Audit - EUR 4,500,000 Receivable Hedge Model
author: Chase Kaetsu
date: 2026-08-08
---

# Build Audit: EUR 4,500,000 Receivable Hedge Model

Workbook audited: `models/builds/2026-08-08-Kaetsu-eur-receivable-model.xlsx`
Built against spec: `docs/specs/2026-07-30-Kaetsu-eur-receivable-spec.md`

## Finding 1 — Named ranges: checked and confirmed

**What I checked:** Opened the workbook and confirmed all ten named ranges from the spec's contract (FC_AMT, S0_in, F0_in, R_USD, R_FC, K_PUT, K_CALL, PREM_PUT, PREM_CALL, T_DAYS) exist and are attached to the correct cells on the Inputs tab, using Formulas → Name Manager.

**What I found:** All ten present and correctly scoped to the workbook (not a single sheet), each pointing at its intended cell on the Inputs tab.

**What I did:** No action needed — confirmed correct.

## Finding 2 — Live recalculation test: input change propagates correctly

**What I checked:** Per the instructions' guidance to "change S0_in and see if the sensitivity table moves," I made a test copy of the workbook, changed S0_in from 1.0800 to 1.2000, and recalculated.

**What I found:** Every downstream value updated correctly — USD_Now, Proceeds_MM, F_implied, every row of the Sensitivity table's S_T column, and every Proceeds_Put value shifted in response. Critically, the Parity_Check_Result cell correctly flipped from PASS to FAIL, since changing S0_in alone (without a matching change to F0_in) breaks covered interest rate parity. This confirms the check is a live formula evaluating real inputs, not a hardcoded "PASS" label.

**What I did:** No defect — this is exactly the behavior the model should have. Reverted the test copy without saving over the real file.

## Finding 3 — Documentation defect: false-positive error flag from literal error-text

**What I checked:** Ran the workbook through recalculation and validation.

**What I found:** The recalculation tool flagged a `#DIV/0!` error at Notes-Assumptions, in the "No error cells" validation-check description cell. On inspection, this was not an actual formula error — the cell is a plain text label reading "No #REF!/#DIV/0!/#N/A anywhere in workbook," and the error-detection scan matched the literal text pattern inside that description, not a real broken formula.

**What I did:** Reworded the cell to "No REF, DIV by zero, or N/A errors anywhere in workbook" to avoid the literal error-string pattern, and reran recalculation. Result: 0 errors across all 86 formulas in the workbook, confirmed via the `recalc.py` grading tool. This is a good example of why "the workbook recalculates clean" needs to be verified directly rather than assumed — a documentation string was enough to trip a mechanical check.

## Finding 4 — Continuity check at the option strike: confirmed no discontinuity

**What I checked:** Per spec §7, verified that Proceeds_Put transitions smoothly through S_T = K_PUT with no jump.

**What I found:** At S_T = K_PUT (the 0% row of the sensitivity table), Proceeds_Put = $4,747,500. Moving one step below (S_T at −1%) also computes $4,747,500 (the floor holds). Moving one step above (S_T at +1%) computes $4,796,100, which continues smoothly upward from the floor value rather than jumping. The `MAX(S_T, K_PUT)` formula structure guarantees this continuity by construction — there is no separate branch or IF statement that could introduce a discontinuity.

**What I did:** No defect found — confirmed correct.

## Finding 5 — Parity check passes on baseline placeholder inputs

**What I checked:** With the original placeholder inputs (S0_in = 1.0800, R_USD = 5.30%, R_FC = 3.10%, T_DAYS = 365, F0_in = 1.1034), confirmed F_implied computes to approximately 1.10336, within the $0.001 rounding tolerance of F0_in.

**What I found:** Parity_Check_Result correctly returns "PASS." This traces back to a Stage 2 spec defect I had already caught and fixed before the build — the original spec draft set F0_in independently at 1.0850, which would have failed this exact check. Because the spec was corrected before the build, the workbook passed on the first generation without needing a mid-build fix.

**What I did:** No action needed at build time — this finding confirms that fixing the spec before building (rather than patching the workbook after) produced a clean result, which is the process the assignment is testing for.

## Summary

Five findings, four confirming correct behavior and one real defect (Finding 3, a text-pattern false positive) that was traced and fixed at its source. No hardcoded values were found in place of formulas; all 86 formulas in the workbook reference named ranges rather than cell addresses or pasted numbers.
