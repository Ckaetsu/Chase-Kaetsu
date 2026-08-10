# Stage 4 review — EUR receivable market data & population · Treasury sign-off

Chase — you wrote the sentence I was preparing to write to you:

> "This is expected by construction, since `F0_in` was itself computed via CIP from the live `R_USD`, `R_FC`, and `S0_in` — the check is confirming internal consistency rather than an independent market observation, and that is stated here explicitly rather than presented as a coincidence."

Most of the cohort reported a passing parity check as if it validated something about the market. You identified that your check was circular, said so in the memo, and refused to let a tautology pass as evidence. That is the single most sophisticated line I have read this stage.

| Criterion | Score |
|---|---|
| Data quality & provenance | 50 / 50 |
| Model resolves cleanly | 33 / 33 |
| Lab cross-check | 17 / 17 |
| **Total** | **100 / 100** |

**What you did well — and why it matters**

- **You documented the searches that failed.** "Barchart, Investing.com forward pages required a paid feed or did not return a numeric 1-year figure." Recording what you *tried* and why it did not work is what makes a proxy defensible rather than lazy. A reader now knows the CIP-implied forward was a fallback, not a shortcut.
- **You decomposed the 6.65% forward gap correctly.** Spot moved 7.0% (1.0800 → 1.1561), while the rate differential *narrowed* from 2.20pp to 1.81pp, "which works against the forward's rise, but the spot move dominates." Identifying two effects pushing in opposite directions and ranking them is real analysis. Most memos would have said "the market moved."
- **You explained why no structural fixes were needed as a consequence of prior work.** "This is a direct result of the Stage 3 audit process: the build was already verified to recalculate correctly when inputs change... so the live-data population confirmed rather than discovered that behavior." That connects two stages into one argument, and it is true.
- **`ISFORMULA`, error scan, 86 formulas, 0 errors** — carried forward consistently from Stage 3, so the counts reconcile across your own documents.

**The one real gap — your euro leg is overnight, your horizon is a year**

`R_FC = 2.25%` is the ECB **deposit facility rate**. That is an overnight policy rate. `T_DAYS = 365`, and your `R_USD` is a one-year constant-maturity Treasury yield, correctly matched.

Your stated reason is that the deposit facility "is quoted daily, giving the freshest available EUR reference rate." Freshness is the wrong optimization here. A tenor-matched rate observed yesterday is strictly better than an overnight rate observed this morning, because the mismatch is a *structural* error while the one-day lag is a rounding error. You applied the right criterion on the dollar leg — "matches `T_DAYS`" — and a different one on the euro leg.

It is worth what it costs: euro-area one-year yields and the deposit facility rate commonly sit 30–50bp apart, and at this tenor a 25bp error in `R_FC` shifts the CIP forward by roughly 0.0028, or about **$12,600 on EUR 4.5M**. Since your `F0_in` *is* the CIP-implied number, that error flows straight into forward proceeds, money-market proceeds, and every sensitivity row.

The free, public, tenor-matched alternative is the ECB Data Portal euro-area one-year spot yield-curve series (`YC/B.U2.EUR.4F.G_N_C.SV_C_YM.SR_1Y`).

**One consequence you should draw**

Because your parity check is circular (your words), your `$193` forward-versus-money-market gap is *not* evidence that parity holds in the market — it is rounding on a relationship you imposed. Your memo says the gap confirms "the parity relationship holds with live data just as it did with the Stage 2 placeholders." Given your own analysis two paragraphs earlier, that claim is stronger than your evidence supports. Trim it to what it shows: the two legs are internally consistent.

**Next — Stage 5**

Hand the workbook and spec to an LLM, get its analysis, then break it. Recompute three outputs by hand with explicit arithmetic — forward proceeds, the put floor, and the crossover spot where the put overtakes the forward. Write the recommendation in a CFO's voice framed on risk tolerance. Your spec retrospective has two strong candidates already: the `F0_in = 1.0850` defect you caught pre-build, and the `R_FC` tenor choice.

— Treasury

---

### How to work this review — professional workflow

Treat this PR the way an analyst treats feedback from Treasury — a review is a proposal to engage with, not a checklist to rubber-stamp:

1. **Read it yourself first.** Understand each point and form your own view before changing anything. Disagreeing *with a documented reason* is a legitimate, senior response.
2. **Stress-test it with an LLM (pushback pass).** Paste this review and your spec into your AI assistant and ask it to (a) explain anything you're unsure of more deeply, and (b) argue the *other side* — where might the reviewer be wrong, and what would you give up by making each change. You're building judgment, not just executing edits.
3. **Decide, then draft the changes with the LLM.** For the points you accept, have the AI help implement them — you specify exactly what and why. Your spec is the prompt; precise in, correct out.
4. **Verify — non-negotiable.** Re-run your own checks (`scripts/recalc.py`, the parity tie-out, sensitivity continuity, no error cells) and confirm the numbers before you commit. An AI will hand you a confident wrong edit; verification is what makes the result *yours*.
5. **Close the loop on the PR.** Reply in the thread with what you changed, what you pushed back on and why, then commit and push. Writing down the reasoning is exactly how this works on a real team.

*This is the same human-in-the-loop discipline the whole project is built on: the LLM drafts, you edit and verify, and you own the result.*
