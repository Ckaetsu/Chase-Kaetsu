# Stage 5 review — EUR receivable LLM analysis & validation · Treasury sign-off

Chase — this is the best diagnostic work in the cohort, and one sentence is why:

> *"the parity check still returns PASS under the LLM's wrong basis (F_implied comes out to 1.176565 against F0_in = 1.1768, a gap of 0.000235, inside the $0.001 tolerance). That means the validation check built into the workbook would not have caught this specific error."*

Everyone else who found a discrepancy reported the discrepancy. You went one level further and asked whether your *control* would have caught it — and discovered it would not. A validation rule that passes a real, dollar-relevant error is worse than no rule, because it manufactures false confidence. Finding that your own check has a blind spot, quantifying the blind spot, and then writing the v2 rule that closes it (recompute `F_implied` under both /360 and /365 and flag a divergence above 0.0002) is exactly what a model-risk function does for a living.

The day-count finding itself is equally good. `T_DAYS = 365` with a /365 divisor collapses the period fraction to exactly 1.0 — and as you say, *"This looks reasonable on its face — 365 days is a year — which is exactly why it's a dangerous, easy-to-miss error rather than an obvious one."* Then you assigned the fault correctly: not to the LLM, but to a spec that states ACT/360 once, in §4 prose, disconnected from the §2 inputs table where the rates actually live. *"The workbook is correct because I built the day-count into the formula directly when constructing it in Stage 3, not because the spec made it foolproof to a new reader."* That is an unusually honest sentence.

| Criterion | Score |
|---|---|
| LLM execution & comparison | 25 / 25 |
| Hand verification | 25 / 25 |
| Recommendation & executive voice | 25 / 25 |
| Spec retrospective | 17 / 17 |
| Repo polish | 6.4 / 8 |
| **Total** | **99 / 100** |

**What you did well — and why it matters**

- **Every number reconciles.** I recomputed the money-market chain independently: `4,500,000 / 1.0228125 = €4,399,633.6` ✓ → `× 1.1561 = $5,086,416.13` ✓ → `× 1.041164 = $5,295,792.8` ✓. And in the memo: the $520,200 unhedged swing ✓, the $193 forward-versus-MM gap ✓, the $205,650 floor give-up ✓, and the $54,450 of retained upside at +5% ✓. All of them.
- **You closed the loop on the Stage 4 review, and the addendum is better than the original finding.** Discovering that you sourced `R_FC` from the **ECB deposit facility** — an overnight policy rate — while `T_DAYS` was 365 and `R_USD` was a 1-year Treasury yield, is a subtle and expensive class of error. Your own diagnosis of *why* is the valuable part: *"I optimized for the wrong criterion: 'freshest available rate' instead of 'tenor-matched rate,' and applied that inconsistently — the right standard on the dollar leg, the wrong one on the euro leg."* Inconsistent application of a standard across two legs of the same trade is precisely how real mispricings survive review.
- **You got the conceptual framing right where most of this cohort got it wrong.** Three times: *"the cost is embedded in the rate itself, via the interest-rate differential"*; *"the CIP-implied forward simply reflects the interest-rate differential between USD and EUR money markets, not a directional bet either way"*; and *"That is not 'free money' — it reflects paying nothing for a feature (upside participation) we are choosing not to buy."* Several classmates described the forward premium as a gain the hedge earns. You refused to, in the memo body, the recommendation, and the justification.
- **You named the cost of your own recommendation and put a trigger on it.** *"We are explicitly giving up upside participation. This is the one real cost of this recommendation, and it should be named as such rather than buried in the numbers — if management's house view is that the euro is likely to strengthen materially, the put becomes the better trade and this recommendation should be revisited."* That is a recommendation with a documented review condition, not an assertion.

**The one substantive correction — you found the tenor error and then did not propagate it**

Your addendum states, precisely and correctly:

> *"it moved F0_in by $0.0028 (1.1768 → 1.1796) and shifted both Proceeds_Forward and Proceeds_MM by $12,629.06 on a $4.5M notional — real money, not a rounding artifact."*

But the workbook, the comparison table, §B, §D and §E all still carry `F0_in = 1.1768` and proceeds of **$5,295,600**. On your own analysis the correct figure is `4,500,000 × 1.17960646 = $5,308,229`.

So the memo hands the CFO a number the analyst has already documented as wrong by $12,629 — and §E tells them *"we know today, to the dollar, what we'll collect in 365 days."* To the dollar is the one thing it is not.

Finding an error is half the loop. The other half is re-running the model, restating the affected figures, and reissuing the deliverable — or, if there is no time, saying so **at the top of the memo** rather than in an addendum to a separate document. A reader who opens only the recommendation, which is what a CFO does, has no way to know the headline number is superseded.

Concretely, three things would have closed this:
1. Repoint `R_FC` at a 1-year euro rate (the 1-year EUR OIS or the ECB 1-year yield, matching how `R_USD` was sourced) and re-run.
2. Restate `F0_in`, `Proceeds_Forward`, `Proceeds_MM`, and the put's relative gaps.
3. Put a one-line dated note at the head of the memo: *"Revised 2026-08-14: `R_FC` re-sourced to a tenor-matched 1-year rate; forward proceeds restated from $5,295,600 to $5,308,229."*

None of this diminishes the finding — it is the sharpest self-audit anyone submitted. It is specifically about what happens *after* the finding, and that step is the one that protects the decision-maker.

**One smaller item — the premium is the one place your time-value discipline lapses**

```
Proceeds_Put = FC_AMT × max(S_T, K_PUT) − FC_AMT × PREM_PUT
             = 5,202,450 − 112,500 = $5,089,950
```

The $112,500 is paid **today**; the $5,202,450 arrives in **365 days**. Carried forward at the same `R_USD` and ACT/360 basis you defended so carefully everywhere else, `112,500 × 1.041164 = $117,131`, so the like-for-like floor is **$5,085,319** and the put's give-up widens from $205,650 to about $210,281.

I raise it because of who you are in this cohort: you are the person who caught a /360-versus-/365 slip worth $1,251 and a tenor mismatch worth $12,629. Discounting consistency is the same family of discipline, and the premium is the one cash flow that escaped it.

**Repo polish — 1.6 points, one item**

`LICENSE` is the only open box. Add an MIT license at the repo root and this stage is a 100.

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
