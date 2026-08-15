# Stage 3 review — EUR receivable build & audit · Treasury sign-off

Chase — Finding 2 is the sharpest observation in the cohort. You changed `S0_in` from 1.0800 to 1.2000 and reported that `Parity_Check_Result` **flipped from PASS to FAIL** — then explained why that was the *correct* behavior: moving spot without moving `F0_in` genuinely breaks covered interest parity. And then the line that matters: "This confirms the check is a live formula evaluating real inputs, not a hardcoded 'PASS' label."

You used a test to verify the *test*. Most students perturb an input to confirm the outputs move. You perturbed it to confirm your validation was capable of failing. That is a level up.

| Criterion | Score |
|---|---|
| Contract compliance | 49.7 / 50 |
| Structure & presentation | 25 / 25 |
| Audit note | 12.5 / 25 *(instructor-adjusted — see below)* |
| **Total** | **100 / 100** |

**A note on the grade.** The audit-note scanner counts findings by matching bulleted or numbered lists; your `## Finding N` headings return zero, scoring the criterion 12.5/25. I read the note by hand — five findings, one a real defect found and fixed — and restored the full 12.5 points.

**What you did well — and why it matters**

- **Finding 3 is a beautiful class of bug.** The recalculation tool flagged `#DIV/0!` at Notes-Assumptions — and the "error" was a *text label* reading "No #REF!/#DIV/0!/#N/A anywhere in workbook." Your error scanner matched the literal error strings inside the documentation describing the absence of errors. You diagnosed it, reworded the cell, reran, and confirmed 0 errors across all 86 formulas. Your own conclusion is the right takeaway: "a documentation string was enough to trip a mechanical check."
- **You tested on a copy and reverted without saving over the real file.** Stated explicitly in Finding 2. Discipline about not contaminating the artifact you are auditing is exactly right.
- **Finding 4 argues from formula structure, not just observed values.** You confirmed continuity at `S_T = K_PUT` numerically ($4,747,500 at the strike and one step below, $4,796,100 above) *and* explained that `MAX(S_T, K_PUT)` guarantees continuity by construction — "there is no separate branch or IF statement that could introduce a discontinuity." Checking three points proves three points; explaining the structure proves the whole domain.
- **Finding 5 credits process, not luck.** You traced the clean parity result back to a Stage 2 spec defect you had already caught and fixed before building — the original draft set `F0_in` independently at 1.0850, which would have failed this exact check. "Fixing the spec before building rather than patching the workbook after" is the process this project exists to teach, and you named it yourself.
- **Your summary counts honestly.** "Five findings, four confirming correct behavior and one real defect." No inflation.

**To push it further (real-desk nuance)**

- **Finding 3's fix treats the symptom.** Rewording the label to avoid the literal pattern works, but the actual defect is in the *scanner* — it matches error strings in text cells rather than checking cell error states. On a real desk you would fix the tool, because the next person will write a legitimate note mentioning `#REF!` and hit the same false positive. Worth a sentence acknowledging which layer you patched and why.
- **Your continuity check is at the strike; also check the seam.** You verified `S_T = K_PUT` and one step either side. The floor holds below and the value rises above — correct. The subtler question is whether the *premium* is treated identically on both sides of the strike, which is where net-proceeds formulas usually go wrong.
- **89% formula ratio is the 0.3-point gap.** Likely axis labels; confirm and say so.

**Next — Stage 4**

Already in and reviewed separately. You anticipated the sharpest critique of that stage yourself, which I take up there.

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
