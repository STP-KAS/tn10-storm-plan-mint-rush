> **Experimental. We are just trying this.** [Disclaimer](../DISCLAIMER.md) · [Home](../README.md) · TN10 only.

# GO-BOT: **draft, no date yet**

**Not a go. Do not run.** This file becomes the bot's locked run sheet once the date is set. Date to be set, after the [rehearsal](../plan/PLAN.md#rehearsal-before-any-date) passes.

Decided 11 Oct ([decisions](../plan/PLAN.md#decisions-11-oct-2026)): the bot runs **all minters** (arms A and B, phases 6 contention and 7 carrier), its storm side, the probes, the chain count on its node and the live Kaspa Pulse-method counter. Steps 2, 5, 10, 20 mints/s. Arm A fee = the phase 1 normal estimate, frozen. The token and its split leaves come from tn ops2. No KRC-20/KNS arm.

What it will contain, copied in shape from [tn10-storm-2026-10-11 GO-BOT](https://github.com/STP-KAS/tn10-storm-2026-10-11/blob/main/go/bot/GO-BOT.md):

- Key times: PREP, gates R1/R2, T0, each phase and step of [PLAN.md](../plan/PLAN.md), END, COUNT (UTC first, CEST second).
- Storm senders: rate, fees, public nodes, node-dead rule (Kaspa Pulse).
- Minters: arm A and arm B split, leaf pool, coin sizes, the [AUTO-FEE](../plan/AUTO-FEE.md) constants frozen at T0.
- Probes: 1×, 2×, 5×, 10× every 30 s.
- Logging: [../logs/README.md](../logs/README.md).
- Guards: RAM, disk, cap, HALT.
- Hard rules: no IP, host:port, key, seed, txid, covenant id, wallet address or local path in git; no tunnel; do not ping Kaspa Pulse (stp does that).
