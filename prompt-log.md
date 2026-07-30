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
