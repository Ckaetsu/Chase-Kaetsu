## 2026-07-24 — Bio draft
Prompt: Help me draft a 150-200 word professional bio for my GitHub profile 
README. Background: senior Finance major at UH Manoa, planning a financial 
advising career. Audience: recruiters and hiring managers in finance.
Result: Claude drafted a bio emphasizing advising, FX hedging coursework, 
and client-facing skills.
Edits: [I shortened sentences and varied their length instead of keeping all the sentences the same size. I also replaced generic wording with more specificity towards clientele.]

## 2026-07-24 — Resume draft
Prompt: I uploaded my existing resume (soccer referee and hotel bellman 
experience) and asked Claude to turn it into a quantified, recruiter-ready 
resume for finance, per the course's RESUME.md guidelines.
Result: Claude reframed my hospitality/referee experience around 
transferable skills (service under pressure, cross-team communication, 
real-time decision-making) and added concrete details like hotel size and 
years of experience instead of vague descriptions.
Edits: [Fed the AI my actual resume as source material, then adjusted the 
wording in the draft to sound more like how I'd actually phrase things 
rather than keeping it as generic AI phrasing.]

## 2026-07-24 — Phase 1 memo draft
Prompt: I asked Claude to help draft a 300-400 word executive memo on an 
FX receivable exposure, covering the exposure, risk, three hedge families, 
and next steps, using the course's example scenario since I couldn't 
locate my individually-assigned scenario. I then asked Claude to revise it 
so it read like my own executive judgment rather than a generic AI 
summary.
Result: Claude's first draft covered the required content but read as a 
neutral report rather than a decision. The revision cut the generic closer, 
added first-person ownership language ("I want to see these three side by 
side," "I'd rather hand you a number I've stress-tested"), and rewrote the 
hedge-option descriptions with more natural, decisive phrasing instead of 
a flat, evenly-weighted list.
Edits: [I reviewed both versions and picked the phrasing that actually 
sounded like something I'd say to a CFO. I revised the closing statement from a generic summary sentence to 
a specific statement of my own approach and reasoning, better reflecting 
executive-level judgment rather than a reported conclusion.]

## 2026-07-24 — Phase 1 memo expansion
Prompt: I asked Claude to expand the memo to meet the 300-400 word 
requirement and be more in-depth with explanations, since the initial 
draft was under the word count and light on business consequences and a 
clear ask.
Result: Claude added a sentence connecting the $450K exposure to actual 
budget impact, made the risk consequence more concrete by tying it to how 
it would show up in results rather than just stating the mechanic, and 
added a closing request for CFO sign-off so the memo ends with a decision 
point instead of trailing off after the process overview.
Edits: [I reviewed the additions for tone consistency with the rest of the 
memo and made sure the new closing ask read as a direct request rather 
than a vague suggestion.]

## 2026-07-30 — Phase 2 model spec draft
Prompt: I asked Claude to draft a full 8-section model specification for 
an FX hedging workbook, using the exact named ranges required (FC_AMT, 
S0_in, F0_in, R_USD, R_FC, K_PUT, K_CALL, PREM_PUT, PREM_CALL, T_DAYS), 
based on my Phase 1 memo's EUR 4,500,000 receivable scenario.
Result: Claude's first draft set F0_in independently of the interest rate 
inputs, which meant the parity check in Section 7 would fail — F_implied 
computed to roughly 1.1034 while the drafted F0_in was 1.0850, a gap far 
outside rounding tolerance.
Edits: [I caught this and had Claude recompute F0_in directly from the 
parity formula given R_USD, R_FC, S0_in, and T_DAYS, so the placeholder 
forward rate is internally consistent with the other placeholder inputs. 
A spec whose own numbers fail its own validation rule would produce a 
workbook that can't pass its own audit checklist before any real data is 
even added.]

## 2026-07-30 — Phase 2 spec: validation rules review
Prompt: I asked Claude to check the Section 7 validation rules against the 
Stage 2 instructions to confirm all four required check types were 
present (forward ≈ MM parity, continuous option proceeds, no error cells, 
every output a formula).
Result: Claude found that the draft only covered three of the four 
required checks and was missing a continuity check confirming the option 
payoff transitions smoothly at the strike price, with no jump between the 
floor payoff and the market-rate payoff.
Edits: [I added a continuity check confirming Proceeds_Put transitions 
smoothly through S_T = K_PUT, so the validation section now matches all 
four checks the instructions specify.]

## 2026-08-08 — Phase 3 model build and audit
Prompt: I asked Claude to generate the actual Excel workbook from my 
committed Stage 2 spec, following the build contract exactly: all ten 
named ranges attached to the right cells, formulas only (no pasted 
values), a cover page and Legend/Key tab with the color convention, all 
three hedge families, a formula-driven ±5% sensitivity table with chart, 
and the spec's validation checks computed live in the workbook.

Result: Claude built the workbook and ran it through a recalculation 
check. The first pass flagged a #DIV/0! error, which on inspection turned 
out to be a false positive — a text cell describing the "no error cells" 
validation rule literally contained the string "#DIV/0!" as an example, 
which tripped the mechanical error scanner even though no formula was 
actually broken.

Claude also ran a live test I asked for: changing S0_in in a test copy of 
the workbook to confirm the sensitivity table and parity check actually 
respond to input changes rather than being hardcoded. The Money Market 
proceeds, F_implied, every sensitivity row, and the put proceeds all 
updated correctly, and the Parity_Check_Result cell correctly flipped from 
PASS to FAIL once the inputs were made inconsistent — confirming the check 
is live, not decorative.

Edits: I had Claude reword the "no error cells" description to avoid the 
literal error-string pattern, then rerun recalculation, which came back 
clean at 0 errors across all 86 formulas. I reviewed the audit note 
findings against what was actually tested and confirmed each one traces 
to a real check I could verify myself rather than a generic pass/fail 
statement.

## 2026-08-08 — Phase 4 market data retrieval and population
Prompt: I asked Claude to retrieve live market data to replace the Stage 2 
placeholder inputs — EURUSD spot, 1-year USD and EUR interest rates, and a 
1-year forward rate — document the source and timestamp for each, then 
populate the Stage 3 workbook through the named-range input cells and 
confirm it still recalculates cleanly.

Result: Claude retrieved a live spot (Bloomberg, 1.1561), a live 1-year 
Treasury yield (Treasury.gov, 4.06%), and a live ECB deposit facility rate 
(2.25%). No live 1-year EURUSD forward quote was accessible through free 
sources, so Claude computed the CIP-implied forward per the Stage 4 
instructions (1.1768) and documented that explicitly rather than 
substituting an unsourced number. Claude also flagged that this forward 
is 6.65% higher than the Stage 2 placeholder (1.1034), and traced that 
gap mostly to the spot move rather than the interest-rate differential.

Result (continued): After populating the workbook, it recalculated clean 
(86 formulas, 0 errors) with no structural fixes needed — the Stage 3 
build's formula-only construction meant live data flowed through every 
tab correctly on the first population. The parity check passed, and the 
sensitivity table recalculated around the new live spot.

Edits: I noted that Claude's "FX Hedging Lab cross-check" was done by 
independently recomputing the Lab's documented formulas by hand with the 
live inputs, not by actually entering the numbers into the Lab's 
interactive web tool, since Claude can't click into it directly. Claude 
flagged this limitation itself rather than implying it had used the tool. 
I plan to also run the live inputs through the actual Lab page myself to 
confirm the match firsthand before treating the cross-check as fully 
complete.

## 2026-08-13 — Phase 5 independent validation and recommendation
Prompt: I asked Claude to simulate an independent "fresh LLM" execution of 
my Stage 2 spec and Stage 4 market-data memo — computing all hedge 
outcomes mechanically from only the values stated in those two documents, 
then compare that output against my real workbook and diagnose any 
discrepancies.

Result: The simulated run matched the workbook exactly on the forward, 
put, call, and unhedged outcomes. It diverged on the money-market hedge by 
about $1,251 (roughly 0.02%), tracing to a rate-basis interpretation gap: 
the spec states ACT/360 in its Assumptions section, but the value doesn't 
get restated in the inputs table row or in the Stage 4 memo's per-row 
source table, so an independent reader working primarily from the memo 
could reasonably apply T_DAYS/365 instead. Notably, the workbook's parity 
check would still return PASS even under that wrong basis, since the 
resulting gap is smaller than the check's tolerance — meaning this 
specific error wouldn't have been caught by the model's own validation 
rule.

Edits: I used this rate-basis gap as the basis for my Part 4 retrospective 
and the spec revisions I proposed there.

## 2026-08-14 — Stage 4 review: R_FC tenor mismatch correction
Prompt: My professor's Stage 4 review flagged that I'd sourced R_FC from 
the ECB deposit facility rate — an overnight policy rate — while T_DAYS 
was tenor-matched to 365 days and R_USD was correctly a 1-year Treasury 
yield. I asked Claude to find a tenor-matched euro-area 1-year rate, 
recompute F0_in and downstream proceeds, and quantify the dollar impact.

Result: Claude sourced the Eurostat/ECB euro area 1-year government bond 
yield (2.01%, most recent published reading, Dec 2025) as a tenor-matched 
replacement for the 2.25% overnight rate. Recomputed F0_in via CIP with 
the corrected rate (1.1768 → 1.1796) and re-ran the workbook: Proceeds_MM 
and Proceeds_Forward both increased by $12,629.06, which lined up closely 
with my professor's own estimate of ~$12,600 for a comparable basis 
error — a good independent confirmation the fix was right, not just 
plausible.

Edits: I also had Claude trim an overclaim in my original market-data 
memo, which said the passing parity check "confirms the parity 
relationship holds with live data." Since F0_in was computed via CIP from 
the same inputs the check evaluates, a pass only shows internal 
consistency, not an independently observed market relationship — my 
professor caught this and it's now corrected in an addendum to the memo.
