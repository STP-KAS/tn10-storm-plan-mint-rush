> **Experimental. We are just trying this.** [Disclaimer](../DISCLAIMER.md) · [Home](../README.md) · TN10 only.

# GO-BUILD: **draft, no date yet**

**Not a go. Do not run.** Date to be set, after the [rehearsal](../plan/PLAN.md#rehearsal-before-any-date) passes.

**Build's role (decided 11 Oct, [decisions](../plan/PLAN.md#decisions-11-oct-2026)):**

1. **Second storm side** in phases 2, 4, 5, 6 and 7, plain payments only, at a fixed rate (to be set with the bot's), fees fixed for the whole run.
2. **Second node for the chain count:** its own synced node with a UTXO index, counting network unique accepted per UTC minute after END, before pruning, as the second independent node next to the bot's (Kaspa Pulse's two-node rule).
3. **No minting.** All mints stay on the bot side so the fee A/B is not mixed with a second machine.

This file will follow the shape of [tn10-storm-2026-10-11 GO-BUILD](https://github.com/STP-KAS/tn10-storm-2026-10-11/blob/main/go/build/GO-BUILD.md): key times (UTC first, CEST second), its own node and fallback, fees, the node-dead rule (Kaspa Pulse), the same log files as the bot ([../logs/README.md](../logs/README.md)), and the handoff files stp passes between Build and the bot.
