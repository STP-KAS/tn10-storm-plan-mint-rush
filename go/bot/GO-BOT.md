> **Experimental. We are just trying this.** [Disclaimer](../../DISCLAIMER.md) · TN10 only; tKAS has no value.

> ## NOT ARMED: rehearsal pending
>
> Do not start anything from this file. It becomes the GO only after the [rehearsal](../../plan/PLAN.md#rehearsal-before-any-date) has passed and the arming commit has set T0 ([how GO works](../README.md#how-go-works)).

# GO-BOT: KCC-20 mint rush under storm load, TN10, T0 = date to be set

You are the bot side: **TN10 ops** (operator: n2, the miner, the storm senders, the minters) with **tn ops2** (logging, the chain count, the write-up). Build's file: [GO-BUILD.md](../build/GO-BUILD.md). Shared rules and the clock: [go/README.md](../README.md). Logging: [LOGGING.md](../LOGGING.md).

**This prompt is locked once stp sends it.** Sending is not the start. It means: get ready now, be ready at T0, and run to END without stp watching. A change after the lock goes in `events.csv` as a `deviation`.

## GO

When this file lands in your chat, that is the GO. No questions. Get ready at once; problems are fixed and logged, never turned into a question.

**T0 = date to be set.** Offsets are the same in both zones; the arming commit fills the UTC and Brussels (CEST = UTC+2; CET = UTC+1 from 25 Oct 2026) columns.

| What | Offset | UTC | Brussels |
|---|---|---|---|
| PREP: n2 synced, UTXO index on, coins claimed, minters loaded with their leaves, loggers built | T0−01:00 | — | — |
| **R1**: read n2, coins, disk, RAM, each public node, NTP; logging and probes start | T0−00:30 | — | — |
| **R2**: storm senders and minters connected and idle; miner on | T0−00:10 | — | — |
| **P1 baseline**: nothing sends; at T0+00:09 freeze the arm A fee | T0+00:00–00:10 | — | — |
| **P2 storm only**: storm 1,200 tx/s | T0+00:10–00:40 | — | — |
| settle: storm off | T0+00:40–00:45 | — | — |
| **P3 mint rush only**: 2 / 5 / 10 / 20 mints/s, 5 min each, arm A rules, both wide pools | T0+00:45–01:05 | — | — |
| settle | T0+01:05–01:10 | — | — |
| **P4 rush + storm, fixed fee**: storm 1,200; mints 2 / 5 / 10 / 20, 5 min each, arm A rules | T0+01:10–01:30 | — | — |
| **P5 rush + storm, A/B**: storm 1,200; mints 2 / 5 / 10 / 20, 10 min each; pool A arm A, pool B arm B, half the target each | T0+01:30–02:10 | — | — |
| **P6 contention**: storm 1,200; 64 minters on the 16-leaf narrow pool, 5 mints/s, arm B rules | T0+02:10–02:20 | — | — |
| **P7 carrier**: storm 1,200; 5 mints/s, arm B rules, half the minters with a 0.2 tKAS carrier | T0+02:20–02:30 | — | — |
| **END**: storm senders and minters stop, miner off | **T0+02:30** | — | — |
| **COUNT** from n2, before pruning | T0+02:30–04:30 | — | — |
| Logs pushed to `logs/bot/` | by T0+05:30 | — | — |
| `logs/COMBINED.md`, `logs/WRITE-UP.md` (tn ops2) | by T0+06:30 | — | — |

Steps change on the clock. A saturated step (accepted < 95% of submitted for 60 s, Kaspa Pulse's definition) is marked `saturated` and still runs its full time.

Thank you, Kaspa Pulse ([@gokugalax](https://x.com/gokugalax)). The node-dead rule, the chain count right after END, the two-node per-minute counter and the baseline are his.

## Gates

**R1 (T0−00:30)**, write `gates.csv`; a failed read is fixed and logged, it never moves T0:
- n2 synced, sink lag < 5 s, DAA moving; **UTXO index on** (minters read leaf state from it). If n2 has no UTXO index, minters read through the public nodes and that is a `deviation` (rebuilding the index takes about 18.5 min offline and about 13 GB; never do it inside the window).
- Each of the five public nodes answers, synced; note its fee quote.
- Coins: at least the [holdings below](#coins), claimed in the ledger.
- Leaves: 1,024 in pool A, 1,024 in pool B, 16 in the narrow pool, allowances as in [PLAN, The token](../../plan/PLAN.md#the-token), read from a node (counts only in git).
- Disk ≥ 20 GB free on the box; RAM ≥ 2.5 GB free.
- NTP offset logged.

**R2 (T0−00:10)**: storm senders and all 64 + 64 minter processes connected and idle; probes running; miner on if n2 is synced. Build's ready file comes via stp; do **not** wait for it; log a `deviation` if it is missing at T0.

## Your storm side

- **1,200 tx/s flat** in P2 and P4 to P7. Basis: your 11 Oct 5-hour mean was 1,211.87 tx/s accepted ([steps.csv](https://github.com/STP-KAS/tn10-storm-2026-10-11/blob/main/logs/bot/steps.csv)). No climb, no cut: the storm is the constant the mints are measured against. If a 15-minute window is not clean, log it; do not change the rate.
- Shape as 11 Oct: the no-signature hop. Label: "no-signature hop, anyone-can-spend, not realistic payments".
- At most 18 storm senders; extra processes only while box RAM ≥ 2.5 GB, never under 1.5 GB (11 Oct rule).
- **Fees fixed 400/600** sompi/gram (half the lanes each). **No sit-out** in this run. Log the quote every minute.

## Your minters

- **Arm A (fixed):** feerate = the **median of n2's `normal[0]` reads in P1** (every 5 s, T0+00:00–00:09), frozen at T0+00:09, logged as `fee-freeze`. Never changed. No bumps.
- **Arm B (auto):** [AUTO-FEE](../../plan/AUTO-FEE.md) R1 to R11 with the constants frozen at T0: base `max(2 × normal[0], 2 × floor)` every 30 s; congestion = `normal[0]` up > 50% in 60 s or read mempool > 50,000; bump after 10 s at 1.5×, at most 4 bumps; caps `min(priority, 5 × normal[0])` and **0.5 tKAS per mint**; at the cap: error and pause that minter.
- **Both arms:** coin-in-flight set (R7); leaf state from a node, never an indexer (R8); rebroadcast on reconnect (R9); back-off on `mempool-full` (R11); payer coins ≥ 10 tKAS; carrier 1 tKAS (P7: 0.2 tKAS for half the minters).
- **Processes:** 32 per arm in the wide pools (32 leaves each), 64 in the narrow pool in P6. Each process paces itself to its share of the step target.
- **Where mints go:** the five public nodes, the same set for A and B. Never n2.
- A mint counts as landed only when its id is in the virtual chain's accepted ids. Per-mint ids stay on the box.

## Where you send

- **Storm and mints: the five public TN10 community nodes only** (names as on 11 Oct, [tn10-storm-2026-10-10 GO-BOT-RUN](https://github.com/STP-KAS/tn10-storm-2026-10-10/blob/main/go/bot-run/GO-BOT-RUN.md#where-you-send)). Each process picks four of the five at launch. **Never submit to n2 or to Build's node.**
- **Node-dead rule (Kaspa Pulse), per public node:** about 20 connection or RPC errors in a row cools that node for 30 s; its load moves to the process's other live nodes. No live node: wait, count no attempt. A failed submit is a reject, never "sent". Log `node-dead` / `node-back`.
- **Be a fair guest:** a node that keeps rejecting or dropping for more than 10 minutes is dropped for the hour and logged.

## n2: reads, the count and the miner

- n2: kaspad 2.1.0, testnet-10, pruned (not archival), UTXO index on, RPC and peers on loopback only, no tunnel, default mempool scale.
- For health reads, fee quotes (`feeest.csv`), watching acceptance, the count and the one miner. n2 lagging does not stop senders or minters; it is logged, and arm B reads prices from a live public node meanwhile.
- **Disk:** pruning bursts up to about 16 GB in 30 minutes have been seen. Guard stops n2 below 10 GB free. Prune your own logs, never the chain.
- **Freeze:** DAA still and sink lag over 3 minutes: restart n2 on the same flags (at most once per 20 minutes), log `node-restart`, rebroadcast pending mints (R9).

## Coins

Hold before R1 (arithmetic; 1.5× need):
- **Storm:** 1,200 tx/s × 6,600 s (110 storm minutes) ≈ 7.9 million tx × about 643 grams × 500 sompi/gram average ≈ 25,500 tKAS → **hold ≈ 38,000 tKAS**.
- **Mints:** 50,400 mints × about 1.13 tKAS (1 tKAS carrier + about 0.13 tKAS fee) ≈ 57,000 tKAS, plus arm B's extra up to its cap (≤ 0.5 tKAS per B-rule mint; 11,100 in P5 + 3,000 in P6 + 3,000 in P7 = 17,100, so ≤ about 8,600) → about 65,600 → **hold ≈ 98,000 tKAS**, in payer coins of ≥ 10 tKAS each.
- Through the claim ledger only; never list a whole address in one call; never re-add a spent or claimed coin. Check every 15 minutes. If a buffer runs low, lower that side (storm or mints) and log a `deviation`; do not stop.
- Leaf deposits and split fees are tn ops2's launch costs, not in these numbers.

## Your miner

Exactly one box-CPU kaspa-miner, one thread, on n2's loopback gRPC, paying the bot's own address (kept out of git). On at R2 if n2 is synced; off while n2 is unsynced; off at END.

## Build

Build runs the second storm side and the second counting node on the same clock. It never mints. stp passes Build's ready file at R2; do not wait for it. Never change your rates to cover Build.

## Log everything (tn ops2)

From R1 to END, as in [LOGGING.md](../LOGGING.md), `step` = phase and step (`P5-s3`): `sec.csv`, `minute.csv`, `public-nodes.csv`, `rejects.csv`, `miner.csv`, `host.csv`, `node.csv`, `events.csv`, `gates.csv`, `ntp.csv`, **`mint-minute.csv`, `feeest.csv`, `mint-mass-sample.csv`, `confirm.csv` (probes and mints)**, `pulse-method-minute.csv` (live); after END `chain-count.csv`, `block-mass.csv`, `steps.csv`, `SUMMARY.md`. Live `logs/bot/SUMMARY.md` every 15 minutes: phase and step, storm accepted, mints landed per arm, p90 time to accept per arm, rejects by reason, last network minute.

## Halt

Only stp halts. Then: halt flag, **stop the watchdog first**, then the scheduler, storm senders and minters to 0, wait 10 s, hard-kill every bot sender, every minter and the miner by process name, check nothing restarts. **No rebroadcast after a halt.** n2 and read-only loggers stay up. The count still runs.

## After END

1. **END (T0+02:30):** storm senders and minters stop, miner off, live loggers stop. n2 stays up.
2. **COUNT (T0+02:30–04:30, before pruning):** `chain-count.csv` and `block-mass.csv` per minute from T0−01:00 to END, from n2. Build counts the same minutes from its node.
3. **Headline:** network unique accepted tx/s per phase and step (peak complete minute, mean, median); minutes where n2 and Build's node differ by more than 1% are `disagree` (in the mean, not in peak or median). Mint results per arm: success rate, p50/p90/p99, fee per mint, rejects by reason, collisions. Everything includes about 56 tx/s base load (Kaspa Pulse).
4. **By T0+05:30:** push the logs to `logs/bot/` in **STP-KAS/tn10-storm-plan-mint-rush** (renamed to `tn10-storm-YYYY-MM-DD` by then), as STP-KAS. **By T0+06:30:** tn ops2 adds `logs/COMBINED.md` and `logs/WRITE-UP.md`, fills the README Results, and updates tn-master. Counts only, leak-scanned.

## Never

No IP, host, port, key, seed, txid, covenant id, wallet address or local path in the repo. No tunnel. No tags, no releases, no force-push. No second miner. No archival. No storm fee other than 400/600. No arm B fee above its caps. No submits to n2 or to Build's node. No firing at a dead node. No minting outside the clock. No indexer as the source of leaf state. Do not ping Kaspa Pulse; stp does that.
