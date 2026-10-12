> **Experimental. We are just trying this.** [Disclaimer](../../DISCLAIMER.md) · TN10 only; tKAS has no value.

> ## NOT ARMED: rehearsal pending
>
> Do not start anything from this file. It becomes the GO only after the [rehearsal](../../plan/PLAN.md#rehearsal-before-any-date) has passed and the arming commit has set T0 ([how GO works](../README.md#how-go-works)). **Once armed, this file pasted into your chat is the GO: no questions, problems are fixed and logged.**

# GO-BUILD: KCC-20 mint rush under storm load, TN10 (Build side, on stp's desk)

You are **Grok Build** on stp's desk. This file is complete on its own. The bot runs the other half from [GO-BOT](../bot/GO-BOT.md); the plan is [PLAN.md](../../plan/PLAN.md).

**Your jobs:** second storm side; **half of all minting** (both arms, P3 to P7) against **your desk node**; the second counting node. No probes.

## Clock (shared with the bot)

| Step | Offset | UTC | CEST | Storm (this side) | Mints (this side, total of both arms) |
|---|---|---|---|---|---|
| PREP: node synced, coins, leaves, loggers built | T0−01:00 | arming | arming | off | off |
| **R1** gate; logging (and probes, bot only) start | T0−00:30 | arming | arming | off | off |
| **R2** gate; miner on; ready file | T0−00:10 | arming | arming | off | off |
| **P1 baseline**; arm A fee frozen at T0+00:09 | T0+00:00–00:10 | arming | arming | off | off |
| **P2 storm only** | T0+00:10–00:40 | arming | arming | **ON** | off |
| settle | T0+00:40–00:45 | arming | arming | off | off |
| **P3 mint rush only**, 5 min per step, arm A rules on both wide halves | T0+00:45–01:05 | arming | arming | off | **1 / 2.5 / 5 / 10 mints/s** |
| settle | T0+01:05–01:10 | arming | arming | off | off |
| **P4 rush + storm, fixed fee**, 5 min per step, arm A rules | T0+01:10–01:30 | arming | arming | **ON** | **1 / 2.5 / 5 / 10** |
| **P5 rush + storm, A/B**, 10 min per step; half arm A, half arm B | T0+01:30–02:10 | arming | arming | **ON** | **1 / 2.5 / 5 / 10** (0.5 / 1.25 / 2.5 / 5 per arm) |
| **P6 contention**: 32 minters on this side's 8 narrow leaves, arm B rules | T0+02:10–02:20 | arming | arming | **ON** | **2.5** |
| **P7 carrier**: arm B rules, half the processes with a 0.2 tKAS carrier | T0+02:20–02:30 | arming | arming | **ON** | **2.5** |
| **END**: storm and minters stop, miner off, node stays up | **T0+02:30** | arming | arming | off | off |
| **COUNT** from this side's node, before pruning | T0+02:30–04:30 | arming | arming | — | — |
| Logs pushed | by T0+05:30 | arming | arming | — | — |

UTC in git; Brussels is CEST = UTC+2 until 25 Oct 2026, CET = UTC+1 after. The arming commit replaces every "arming" cell with the real time. Steps change on the clock, never early, never extended. Both sides run the **same clock**; the network sees the sum (2 / 5 / 10 / 20 mints/s in P3 to P5, 5 in P6 and P7).

Your storm is on for 110 minutes (P2: 30, P4 to P7: 80).

## Your node: the desk node

- kaspad 2.1.0, testnet-10, **pruned, not archival**, **UTXO index on**, default mempool scale, RPC and peers on loopback only, no tunnel, a watcher that restarts it on the same flags. Never `--enable-unsynced-mining`.
- Used for: your storm (75%), **your minters' submits (primary)**, leaf state, fee reads, the count, the miner. Minter fallback: the five public TN10 nodes, at most 25% of minter submits, only while your node is node-dead.
- Keep ≥ 30 GB disk and ≥ 4 GB RAM free; prune your own logs first, never the chain.

## Storm side

- **Rate: your last clean rate, set here at arming: ____ tx/s**, flat in P2 and P4 to P7.
- **75% of storm senders on your desk node, 25% on the five public TN10 nodes**; if the public nodes show rising rejects or cooling, move your public share back to the desk node first (`rate-change`), keeping your total. Never more than 25% public.
- **Fees fixed 200/300** sompi/gram, half the lanes each, no sit-out. Log your node's quote and at least one public node's every minute; `fee-outlier` if your node's priority is more than 3× the other(s). Record only.

## Your minters (half of the run)

| Item | Value |
|---|---|
| Wide pool A leaves | **leaves 512-1023** (512) |
| Wide pool B leaves | **leaves 512-1023** (512) |
| Narrow pool leaves | **leaves 8-15** (8) |
| Wide minter processes | **32**: 16 on pool A, 16 on pool B, 32 leaves each (P3 and P4: all 32 use arm A rules; P5: pool A = arm A, pool B = arm B; P7: all arm B) |
| Narrow minter processes (P6 only) | **32**, 4 per leaf on purpose |
| Minter processes live at once | at most 32 (wide) + 32 idle narrow until P6 |
| Pace at the top step | 10 mints/s on this side: each wide process about every 3.2 s; a leaf is reused about every 100 s |
| Mints on this side | P3 to P5: 22,200; P6: 1,500; P7: 1,500; **total 25,200** (half the run) |

## Coins (arithmetic, hold before R1)

- **Storm:** your rate × 6,600 s × your real grams per tx × 250 sompi/gram average, held 1.5×.
- **Mints:** 25,200 × about 1.13 tKAS (1 tKAS carrier + about 0.13 tKAS fee at 204 sompi/gram, 11 Oct measured) ≈ 28,500 tKAS, plus arm B's extra up to its cap (B-rule mints on this side: 5,550 in P5 + 1,500 in P6 + 1,500 in P7 = 8,550, ≤ 0.5 tKAS each ≈ 4,300) ≈ 32,800 → **hold ≈ 49,000 tKAS** (1.5×) in payer coins of ≥ 10 tKAS, on addresses only this side uses.
- Mint coins come from a funding transfer from the bot main wallet to a Build-held address before arming (**stp approves the transfer**; address not in git), or from your own wallet if stp says so. Never shared with the bot's coins. Check every 15 min; memory-safe top-ups; log every `coin-refresh`.

## RAM and processes on the desk

Storm senders + 32 live minter processes + loggers; launch only while free RAM ≥ 4 GB. **To be confirmed at the rehearsal.**

## Gates

- **R1 (T0−00:30)**, `gates.csv`: desk node synced, sink lag < 5 s, UTXO index on; coins ≥ holdings; your 512 + 512 + 8 leaves read from your node with their allowances; disk ≥ 30 GB, RAM ≥ 4 GB free; NTP offset logged; chain-count reader running live.
- **R2 (T0−00:10)**: storm senders and 32 wide minter processes connected and idle; miner on if synced; write `FOR-TN10-OPS-ready-mintrush.txt` (format: [LOGGING](../LOGGING.md#handoff-files-build-to-the-bot); add `minters: 32`, `leaves: 512/512/8`, `mint_coins_tkas`).

## Miner

Exactly one, one thread, on your desk node only, paying Build's own address (not in git). On at R2 if synced; off while unsynced; off at END.

## Rules (complete; same on both sides)

- **Saturation** (Kaspa Pulse's definition): accepted < 95% of submitted for 60 s. The step is marked `saturated` and still runs its full time.
- **Node-dead rule** (Kaspa Pulse): stop using a node on 5 s of RPC failure, not synced, sink lag over 60 s on a fresh read, or virtual DAA still for 30 s; resume by itself on a healthy fresh read. Public nodes: about 20 errors in a row cools that node for 30 s. A failed submit is a reject, never "sent". Log `node-dead` / `node-back`.
- **Minters, both arms:** coin-in-flight set (AUTO-FEE R7); leaf state read from a node with a UTXO index, never from an indexer (R8); rebroadcast pending mints on reconnect or restart (R9); back off on `mempool-full` 1, 2, 4 s (R11); payer coins ≥ 10 tKAS; carrier 1 tKAS (P7: 0.2 tKAS for half the processes). A mint counts as landed only when its id is in the virtual chain's accepted ids.
- **Arm A (fixed):** feerate = median of `normal[0]` read every 5 s in P1 (T0+00:00–00:09) from your own node plus 2 public nodes, frozen at T0+00:09, logged as `fee-freeze`. Never changed, no bumps. The two sides freeze independently; both values are logged and compared.
- **Arm B (auto):** [AUTO-FEE](../../plan/AUTO-FEE.md) R0 to R11, constants frozen at T0: R0 median of 2-3 live nodes, `fee-outlier` on a node > 3× the median; base `max(2 × normal[0], 2 × floor)`; congestion: `normal[0]` +50% in 60 s or mempool > 50,000; bump after 10 s at 1.5×, at most 4; caps `min(priority, 5 × normal[0])` and **0.5 tKAS per mint**; at a cap: error and pause that process.
- **Disjoint sets:** only your own leaves and your own coins. Never build on the other side's leaves or coins; never submit to the other side's node.
- **Keep going, but fix first:** a problem is fixed and logged (`fix`, `deviation`). Only END or stp's halt stops the run.
- **Halt (stp only):** halt flag, stop the watchdog first, then the scheduler, storm senders and minters to 0, wait 10 s, hard-kill every sender, minter and the miner by process name, check nothing restarts. No rebroadcast after a halt. Node and read-only loggers stay up; the count still runs.
- **Public repo:** counts only. No IP, host, port, key, seed, txid, covenant id, wallet address, tunnel or local path. Leak-scan before every push. No force-push, no tags, no releases. Do not ping Kaspa Pulse; stp does that.

Thank you, Kaspa Pulse (@gokugalax, kaspapulse.com): the node-dead rule, the baseline, saturation before T0, the two-node per-minute count and the chain count right after END are his.

## Logging

From R1 to END ([LOGGING.md](../LOGGING.md)), `side = build`, `step` = phase and step: `sec`, `minute`, `public-nodes`, `rejects`, `miner`, `host`, `node`, `events`, `gates`, `ntp`, `mint-minute`, `feeest`, `mint-mass-sample`, `confirm` (mints), `pulse-method-minute` (live); after END `chain-count`, `block-mass`, `steps`, `SUMMARY.md`. Per-transaction logs and ids stay on the desk.

## One paste, 3 PowerShell windows

If stp has to paste anything: one PowerShell block opening 3 titled windows: (1) NODE: desk node, watcher, miner; (2) SENDERS + MINTERS: storm senders and the 32 + 32 minter processes, on the clock; (3) MONITOR + COUNT: health reads, loggers, live chain-count reader.

## After END

1. COUNT T0+02:30–04:30 from your node: `chain-count.csv`, `block-mass.csv`, T0−01:00 to END.
2. `FOR-TN10-OPS-result-mintrush.txt` by T0+05:00 (stp carries it): per phase storm sent / submit-ok / accepted / rejects; mints per arm landed / attempts / p50 / p90 / p99 / fee per mint / rejects by reason / collisions; network unique per phase; every change.
3. By T0+05:30 commit `logs/build/`. Counts only, leak-scanned.
