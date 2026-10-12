> **Experimental. We are just trying this.** [Disclaimer](../DISCLAIMER.md) · [Home](../README.md) · TN10 only; tKAS has no value. Mainnet: **not tested (label B)**.

# go: the shared rule book, one clock

> ## NOT ARMED: rehearsal pending
>
> These GO files are **not valid** until the [rehearsal](../plan/PLAN.md#rehearsal-before-any-date) has passed and an **arming commit** has filled in T0. Until then nobody starts anything from them. Date to be set.

Prompts: [GO-BOT](bot/GO-BOT.md) · [GO-BUILD](build/GO-BUILD.md) · logging: [LOGGING.md](LOGGING.md) · the plan: [../plan/PLAN.md](../plan/PLAN.md) · the fee algorithm: [../plan/AUTO-FEE.md](../plan/AUTO-FEE.md).

> **Node note (12 Oct 2026):** box node n2 was stopped by the disk guard on 11 Oct and its data deleted on 12 Oct ([tn10-storm-2026-10-11 CHANGES-DURING-RUN](https://github.com/STP-KAS/tn10-storm-2026-10-11/blob/main/logs/bot/CHANGES-DURING-RUN.md)). These sheets now say "the box node": a fresh pruned TN10 node on the box with a UTXO index, **TBD** (not yet synced) before any date.

Thank you, Kaspa Pulse (@gokugalax, kaspapulse.com). This run sheet is built on his method: plan first, a baseline, fixed steps, saturation defined before T0, the node-dead rule, the chain count right after END, and his counter (**accepted in blocks, two independent nodes, per UTC minute, by block time**). Every network count includes about **56 tx/s** of TN10 base load (his figure). [Every input](https://github.com/STP-KAS/tn-master/blob/main/credits/KASPA-PULSE.md).

## How GO works

1. **Rehearsal first.** The GO is valid only after the rehearsal has passed ([pass criteria](../plan/PLAN.md#rehearsal-before-any-date)); its result goes to `logs/rehearsal/`.
2. **Arming commit.** tn ops2 then makes one normal commit that: replaces every "NOT ARMED" banner with **ARMED**, sets **T0** (UTC and CEST) and fills the two clock columns below, links the rehearsal result, writes the token's final leaf counts (counts only), and writes Build's storm rate into GO-BUILD. The commit SHA is the GO lock.
3. **GO.** stp pastes the file to each side. **When this file lands in your chat, that is the GO. No questions. Problems are fixed and logged, never turned into a question.**
4. **Lock.** Sending is not the start. It means: get ready now, be ready at T0, run to END without stp watching. A change after the lock goes in `events.csv` as a `deviation`.

## The single clock

**T0 = date to be set.** T0 is the start of phase 1 (baseline). Both sides read their times from this table only. Clocks in git are UTC. Brussels is **CEST = UTC+2** until 25 Oct 2026; from then **CET = UTC+1**. The arming commit fills the UTC and Brussels columns.

| Step | Offset | UTC | Brussels | Bot | Build |
|---|---|---|---|---|---|
| PREP: nodes, coins, leaves, loggers built | T0−01:00 | — | — | yes | yes |
| **R1** gate; logging starts | T0−00:30 | — | — | yes | yes |
| **R2** gate; miners on; Build's ready file | T0−00:10 | — | — | yes | yes |
| **P1 baseline**, nothing of ours on the network; arm A fee frozen at T0+00:09 | T0+00:00–00:10 | — | — | reads only | reads only |
| **P2 storm only** | T0+00:10–00:40 | — | — | storm 1,200 tx/s | storm, its rate |
| settle (storm off) | T0+00:40–00:45 | — | — | off | off |
| **P3 mint rush only**: 2 / 5 / 10 / 20 mints/s, 5 min each | T0+00:45–01:05 | — | — | mints (arm A rules) | off |
| settle | T0+01:05–01:10 | — | — | off | off |
| **P4 mint rush + storm, fixed fee**: 2 / 5 / 10 / 20, 5 min each | T0+01:10–01:30 | — | — | mints (A rules) + storm | storm |
| **P5 mint rush + storm, A/B**: 2 / 5 / 10 / 20, 10 min each, half A half B | T0+01:30–02:10 | — | — | mints A + B + storm | storm |
| **P6 contention**: narrow pool, 5 mints/s | T0+02:10–02:20 | — | — | mints (B rules) + storm | storm |
| **P7 carrier**: 5 mints/s, half with 0.2 tKAS carrier | T0+02:20–02:30 | — | — | mints (B rules) + storm | storm |
| **END**: all senders and minters stop, miners off | **T0+02:30** | — | — | yes | yes |
| **COUNT** from two nodes, before pruning | T0+02:30–04:30 | — | — | the box node | Build's node |
| Logs pushed | by T0+05:30 | — | — | `logs/bot/` | `logs/build/` |
| COMBINED and WRITE-UP pushed (tn ops2) | by T0+06:30 | — | — | yes | — |
| Suggested window for Kaspa Pulse's counter (stp sends it) | T0−00:20 to T0+02:35 | — | — | — | — |

Step changes happen on the clock, never early, never extended. Probes (1×, 2×, 5×, 10× the node minimum, every 30 s) run on the bot side from R1 to END.

## Roles

| Side | Who | Does | Sends to | Node | Miner |
|---|---|---|---|---|---|
| Bot | **TN10 ops** (the box node, miner, storm senders, minters) with **tn ops2** (logging, count, write-up) | **All minting** (arms A and B, P6, P7), storm side, probes | The five public TN10 nodes only | the box node on the box: reads, fee quotes, the bot's count, the miner. Never submits | 1 |
| Build | **Grok Build** on stp's desk | **Second storm side**, **second counting node**. **No minting** | Its own desk node (75%) and the five public nodes (25%, backs off first) | Its desk node, pruned, UTXO index on | 1 |
| Token | **tn ops2** | Launches and splits the run token (limit 1,000, supply 60,000,000) and the rehearsal token (limit 1,000, supply 10,000,000) before arming | — | — | — |

Never submit to the other side's node.

## Shared rules

- **Saturation (defined before T0, Kaspa Pulse):** accepted < 95% of submitted for 60 s. A saturated step is marked, never extended or shortened.
- **Node-dead rule (Kaspa Pulse):** own node used for submits: stop on 5 s of RPC failure, not synced, sink lag over 60 s on a fresh read, or virtual DAA still for 30 s; resume by itself when a fresh read is healthy. Public nodes: about 20 errors in a row cools that node for 30 s, load moves to the others. A failed submit is a reject, never "sent". No firing at a dead node.
- **Fees are fixed for every storm sender** (bot 400/600, Build 200/300), **no sit-out rule** in this run, so the storm load stays the same while mints compete. Mint fees: arm A fixed, arm B [AUTO-FEE](../plan/AUTO-FEE.md) with caps.
- **Miners:** exactly one per side, one thread, on the side's own node, on at R2 if synced, off while that node is unsynced, off at END.
- **Keep going, but fix first.** A problem is fixed and logged (`fix`, `deviation`). Only END or stp's halt stops the run.
- **Count once, from the chain, from two nodes, soon after END** (Kaspa Pulse): the box node and Build's node, before pruning.
- **Log everything:** [LOGGING.md](LOGGING.md).
- **Public repo:** counts only. No IP, host, port, key, seed, txid, covenant id, wallet address, tunnel or local path. Leak-scan before every push. No force-push, no tags, no releases.

## Headline (after END)

Network unique accepted tx/s from the two chain counts, never summed across sides: peak complete minute, mean and median **per phase and per step**. A minute where the two nodes differ by more than 1% is marked `disagree`: kept in the mean, left out of the peak and the median. Plus the mint results per arm ([METRICS](../plan/METRICS.md)). Kaspa Pulse's own count, if he makes one, goes next to ours.

## stp's to-do before T0

1. Confirm the rehearsal passed, then let tn ops2 make the arming commit.
2. Tell Kaspa Pulse the date and the suggested window, if he wants to count.
3. Paste [GO-BOT](bot/GO-BOT.md) to the bot and [GO-BUILD](build/GO-BUILD.md) to Build.
4. At R2, carry Build's ready file to the bot. After END, carry Build's result file.
