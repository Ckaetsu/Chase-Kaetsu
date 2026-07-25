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

We're set to receive EUR 4,500,000 in one year. The euro amount is fixed. What it converts to in dollars is not, and that's the whole problem. At EURUSD 1.10, we collect $4,950,000. At 1.00, it's $4,500,000 — a $450,000 gap on a move that's well within a normal year for this currency pair.

## Why It's Risky

We have no natural offset for this exposure anywhere else in the business. If the euro weakens before settlement, the loss lands directly on this receivable, in full.

## Hedge Options

A forward contract locks in today's rate for delivery in a year. It removes the uncertainty completely — the only cost is giving up any benefit if the euro moves in our favor instead.

A money-market hedge gets us to the same outcome through a different mechanism: borrow EUR now, convert to dollars at today's spot rate, and invest the proceeds until settlement. It's worth considering if forward pricing isn't attractive, though it draws on credit lines we may need for other purposes.

The third option, a put option, is the only one that keeps upside intact. We pay a premium for the right, not the obligation, to sell EUR at a set rate. If the euro strengthens, we participate in that gain; if it weakens, we're protected. The premium is paid either way.

## Next Steps

Before recommending one of these, I want to see them modeled side by side. That means specifying the model, building it with AI assistance and auditing every formula myself, replacing placeholder rates with live-sourced data, and running an independent validation pass. I'd rather bring you a recommendation that's been tested than one that hasn't.
