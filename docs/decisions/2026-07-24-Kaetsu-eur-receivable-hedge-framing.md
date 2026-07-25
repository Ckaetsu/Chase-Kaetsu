---
title: FX Hedging Framing - EUR 4,500,000 Receivable
author: Chase Kaetsu
date: 2026-07-24
---

# FX Hedging Framing: EUR 4,500,000 Receivable

**To:** CFO
**From:** Chase Kaetsu
**Date:** July 24, 2026
**Re:** Currency exposure on upcoming EUR receivable

## The Exposure

We're set to receive EUR 4,500,000 in one year. That euro figure is locked in — what it's worth in dollars is not. At EURUSD 1.10, we collect $4,950,000. At 1.00, it's $4,500,000. That's a $450,000 gap, and a 9% move like that isn't a stretch — it's a normal year for this currency pair. Right now, our USD budget for this receivable isn't really a number. It's a range, and a wide one.

## Why It's Risky

If the euro weakens before settlement, that loss comes straight off our proceeds. We have nothing elsewhere in the business offsetting this exposure, so whatever happens to EURUSD between now and settlement hits this line directly.

## Hedge Options

**Forward contract** — lock today's rate for delivery in a year. Certainty, full stop. The tradeoff: if the euro strengthens instead, we don't get to benefit from it.

**Money-market hedge** — borrow EUR now, convert at today's spot rate, invest the dollars until settlement. Gets us to the same place as a forward, just built out of our own borrowing capacity instead of a bank's forward book. Worth a look if forward pricing isn't attractive, though it does tie up credit lines we might want elsewhere.

**Put option** — pay a premium for the right, not the obligation, to sell EUR at a set rate. This is the only one of the three that keeps upside if the euro strengthens. The cost is the premium, paid whether we end up needing the protection or not.

## Next Steps

I want to see these three side by side before recommending one. My plan:
1. Spec out the model — exact inputs, exact calculation logic
2. Build it with AI assistance, then audit every formula myself
3. Swap in live, sourced market data for the placeholder rates
4. Run an independent validation pass and bring you a final recommendation

I'd rather hand you a number I've stress-tested than a guess dressed up as one.
