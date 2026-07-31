# Stage 2 review — EUR receivable · Treasury sign-off

Chase — I read your specification the way Treasury actually reads one: the spec is the contract the build must honor, so I checked whether an analyst could hand this to a modeler (or an AI) and get back the workbook you intended, with the exposure pointed the right way. It holds up. You are receiving EUR 4,500,000 in 365 days, so the firm is hurt if the euro *weakens* against the dollar — and every hedge in your flow defends that direction correctly.

| Criterion | Score |
|---|---|
| Named-range contract & tab architecture | 30 / 30 |
| Calculation flow | 30 / 30 |
| Validation & sensitivity plan | 20 / 20 |
| Reproducibility & prompt log | 20 / 20 |
| **Total** | **100 / 100** |

**What you did well — and why it matters**

- **The named-range contract is complete and honest about provenance.** All ten ranges carry a unit and a Stage-4 source — `R_USD` to the Fed H.15 release, `R_FC` to the ECB deposit facility, spot and forward to a live provider. That "indicative → replaced at Stage 4" flag is exactly how a desk separates a placeholder from an executable rate. It prevents the classic error of a model quietly shipping with stale assumptions nobody reviewed.
- **The money-market hedge is built in the correct direction for a receivable.** Borrow EUR now, convert at spot, invest USD to settlement — that monetizes the euro you're owed *today* and removes the FX path entirely. Getting the borrow-side currency right is where most students invert the trade; you didn't.
- **Your parity check is a real control, not decoration.** Requiring `|F_implied − F0_in|` within $0.001 means the forward and money-market lines must agree or the workbook is telling you there's a formula bug. That is how covered interest parity earns its keep on a live sheet.
- **The put is specified as a true floor with strike continuity enforced.** `max(S_T, K_PUT) − premium`, plus a check that proceeds never fall below `FC_AMT × K_PUT − premium`, guarantees the insurance actually pays when the euro drops.

**To push it further (real-desk nuance)**

- **Live forward points will not exactly equal your parity forward.** `F_implied` is the theoretical no-arbitrage rate; the *quoted* forward embeds a cross-currency basis and a dealer spread. At Stage 4, expect a small gap and treat it as basis, not error — your $0.001 tolerance is for internal consistency, not for the market quote. But if that gap is *large*, don't smooth it away: it means the forward and money-market hedges lock materially different USD amounts, so flag the advantaged leg (or a genuine arbitrage) and pick it — that mispricing is real money.
- **Your `K_PUT` is at-the-money (1.0800), the most expensive floor to buy.** An out-of-the-money put struck below spot costs far less premium but accepts a band of unhedged downside first. Frame that as a deliberate risk-tolerance choice — how much floor is worth how much premium — rather than defaulting to ATM.
- **Don't let hindsight pick the winner.** In Stage 5, whichever hedge "won" depends entirely on where EUR actually settled, which you can't know when you trade. Anchor the recommendation on certainty and risk appetite (forward/MM lock a known number; the put buys upside for a premium), not on the ex-post outcome.

**Next — Stage 3**

Hand this spec to the AI to build, then audit what comes back against it — every calculated cell a formula referencing your named ranges, all three families present, the sensitivity table and chart, and a build-audit note with at least three findings. A spec this tight means few surprises, but remember: any weakness left in the spec becomes a defect in the workbook. Audit like you expect to find one.

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
