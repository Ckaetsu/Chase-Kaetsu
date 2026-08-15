---
title: Hedge Recommendation - EUR 4,500,000 Receivable
author: Chase Kaetsu
date: 2026-08-14
---

# Hedge Recommendation: EUR 4,500,000 Receivable

**To:** CFO
**From:** Chase Kaetsu
**Date:** August 14, 2026
**Re:** Final hedge recommendation, based on live market data and independent model validation

## A. Exposure Summary

We hold a EUR 4,500,000 receivable settling in 365 days. As of live market data retrieved August 8, 2026 (spot EURUSD = 1.1561), this receivable is worth approximately $5.20 million today — but the dollar amount at settlement depends entirely on where EURUSD lands a year from now. The purpose of this memo is to recommend which of three hedge strategies, or no hedge at all, best fits our risk posture given the numbers we now have in hand.

## B. Hedge Outcomes

Four strategies were modeled against three settlement scenarios (EUR down 5%, unchanged, and up 5% from today's spot):

| Strategy | EUR −5% (S_T = 1.0983) | Unchanged (S_T = 1.1561) | EUR +5% (S_T = 1.2139) |
|---|---|---|---|
| Unhedged | $4,942,350 | $5,202,450 | $5,462,550 |
| Forward | $5,295,600 | $5,295,600 | $5,295,600 |
| Money-Market | $5,295,793 | $5,295,793 | $5,295,793 |
| Put (floor) | $5,089,950 | $5,089,950 | $5,350,050 |

**Unhedged** is the most volatile by a wide margin — a $520,200 swing across the ±5% range we modeled, and that range is well within a normal year for this currency pair.

**Forward and money-market** land within $193 of each other at every scenario, which is exactly what covered interest rate parity predicts: they are the same trade constructed two different ways, and our validated model confirms they behave that way in practice, not just in theory.

**The put** gives up $205,650 of certainty at the low end (relative to the forward) in exchange for keeping $54,450 of upside at the high end, net of the $112,500 premium already paid regardless of outcome.

## C. Sensitivity Interpretation

The three strategies behave in fundamentally different ways as the euro moves, and that difference — not the specific dollar figures — is what should drive the decision:

- **If the euro depreciates**, the forward and money-market hedges are strictly better than staying unhedged or buying a put: they lock in $5,295,600–$5,295,793 regardless of how far EURUSD falls, while the put's floor sits about $206,000 lower (still far better than unhedged, but the premium cost shows up here as a real drag).
- **If the euro appreciates**, unhedged and the put both benefit from the move, while the forward and money-market hedges cap us at the locked rate no matter how far EURUSD rises. The put's premium is the price of keeping that optionality.
- **Certainty vs. flexibility vs. cost** is the real three-way tradeoff: the forward and money-market hedges buy full certainty at zero explicit cost (the cost is embedded in the rate itself, via the interest-rate differential). The put buys partial certainty (a floor) plus flexibility (upside participation) for an explicit $112,500 premium, paid whether or not we end up needing the protection.

## D. Recommendation

**I recommend the forward contract.**

Between the forward and the money-market hedge, the forward is operationally simpler — one trade, no need to actually borrow EUR or manage an offsetting deposit — and the two produce results within $193 of each other, so there is no material financial reason to prefer the money-market route unless our forward counterparty pricing were materially worse than the rates underlying this model (it isn't, per the parity check).

Between the forward and the put, the put would only be the better call if we had a specific, reasoned view that the euro is more likely to strengthen than weaken over the next year. Nothing in our current market data supports that view over the alternative — the CIP-implied forward simply reflects the interest-rate differential between USD and EUR money markets, not a directional bet either way. Absent a house view that the euro is going to rally, paying a $112,500 premium for upside participation we have no particular reason to expect is not the disciplined choice.

## E. Executive Justification

- **Cash-flow stability:** the forward removes all variance from this receivable's USD value — we know today, to the dollar, what we'll collect in 365 days.
- **Budget certainty:** a $5,295,600 figure can go directly into next year's budget as a hard number rather than a range, which is the entire reason this exercise started (the original $450,000 swing identified in the Phase 1 memo is now fully eliminated).
- **Liquidity:** unlike the money-market hedge, the forward requires no upfront borrowing or collateral posting under normal counterparty terms, keeping our credit lines free for other uses.
- **Optionality:** we are explicitly giving up upside participation. This is the one real cost of this recommendation, and it should be named as such rather than buried in the numbers — if management's house view is that the euro is likely to strengthen materially, the put becomes the better trade and this recommendation should be revisited.
- **Premium cost:** by choosing the forward, we avoid the $112,500 premium entirely. That is not "free money" — it reflects paying nothing for a feature (upside participation) we are choosing not to buy.
