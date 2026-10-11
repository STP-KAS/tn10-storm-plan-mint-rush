> **Experimental. We are just trying this.** [Disclaimer](../../DISCLAIMER.md) · TN10 only; tKAS has no value.

> ## NOT ARMED: rehearsal pending
>
> Do not start anything from this file. It becomes the GO only after the [rehearsal](../../plan/PLAN.md#rehearsal-before-any-date) has passed and the arming commit has set T0 and your storm rate ([how GO works](../README.md#how-go-works)).

# GO-BUILD: KCC-20 mint rush under storm load, TN10, T0 = date to be set

You are **Grok Build** on stp's desk. The bot side (TN10 ops on the box) has its own file: [GO-BOT.md](../bot/GO-BOT.md). Shared rules and the clock: [go/README.md](../README.md). Logging: [LOGGING.md](../LOGGING.md).

**Your two jobs: the second storm side, and the second counting node. You do not mint.** All minting stays on the bot side, so the fee A/B is not mixed with a second machine.

**This prompt is locked once stp sends it.** Sending is not the start. It means: start your node now, be ready at T0, and run to END without stp watching. A change after the lock goes in `events.csv` as a `deviation`.

## GO

When this file lands in your chat, that is the GO. No questions. Start your desk node now if it is not running, check coins, arm senders and loggers. Problems are fixed and logged, never turned into a question.

**T0 = date to be set.** Offsets are the same in both zones; the arming commit fills the UTC and Brussels (CEST = UTC+2; CET = UTC+1 from 25 Oct 2026) columns. Read your times from this file only.

| What | Offset | UTC | Brussels |
|---|---|---|---|
| **Now**: desk node up and syncing, coins, senders, loggers | now | now | now |
| PREP | T0−01:00 | — | — |
| **R1**: read your setup; logging and your chain-count reader start | T0−00:30 | — | — |
| **R2**: write `FOR-TN10-OPS-ready-mintrush.txt`; miner on | T0−00:10 | — | — |
| P1 baseline: **you send nothing** | T0+00:00–00:10 | — | — |
| **P2 storm only: storm on** | T0+00:10–00:40 | — | — |
| settle and P3 (bot mints alone): **storm off** | T0+00:40–01:10 | — | — |
| **P4 to P7: storm on** (the bot mints alongside) | T0+01:10–02:30 | — | — |
| **END**: senders stop, miner off; node stays up | **T0+02:30** | — | — |
| **COUNT** from your node, before pruning | T0+02:30–04:30 | — | — |
| Logs pushed to `logs/build/` | by T0+05:30 | — | — |

Your storm is on for **110 minutes**: P2 (30) and P4 to P7 (80). Starts and stops are on the clock, whatever is happening.

Thank you, Kaspa Pulse ([@gokugalax](https://x.com/gokugalax)). The node-dead rule, the chain count right after END, and the two-node per-minute count you are half of are his.

## Gates

**R1 (T0−00:30)**, write `gates.csv`; a failed read is fixed and logged, never a reason to stop:
- Desk node synced, sink lag < 5 s, DAA moving, UTXO index on, default mempool scale.
- Coins ready (below). Disk ≥ 30 GB free, RAM ≥ 4 GB free. NTP offset logged.
- The chain-count reader is running against your node (it must keep up live, so the count after END is only a check).

**R2 (T0−00:10)**: senders connected and idle; miner on if synced; write the ready file.

## Your storm side

- **Rate: Build's last clean rate, set here at arming: ____ tx/s.** (Your 11 Oct result files are not in the repo yet; the arming commit fills this number from your last clean, flat 15-minute window.) **Flat** in P2 and P4 to P7: no climb, no cut. The storm is the constant the bot's mints are measured against. If a window is not clean, log it; do not change the rate.
- **Fees fixed 200/300** sompi/gram, half the lanes each. **No sit-out** in this run. Log your node's quote every minute.
- **75% of senders on your desk node, 25% on the five public TN10 nodes** (names as on 11 Oct, [tn10-storm-2026-10-10 GO-BOT-RUN](https://github.com/STP-KAS/tn10-storm-2026-10-10/blob/main/go/bot-run/GO-BOT-RUN.md#where-you-send)); each public sender picks four of the five at launch. **Do not crowd out the bot:** if the public nodes show rising rejects, errors or cooling, move your public share back to your desk node first (`rate-change`, reason "public nodes busy"), keeping your total rate.
- If your desk node goes node-dead, the public share keeps sending; never move more than 25% onto the public nodes to cover it; fix your node.

## Your counting node

Your desk node is **the second independent node** for the chain count (Kaspa Pulse's two-node rule). From R1, read the virtual chain live and write per UTC minute: network unique accepted, chain blocks, and the Pulse-method count (accepted in blocks, by block time). After END, recount T0−01:00 to END before pruning: `chain-count.csv`, `block-mass.csv`. The bot does the same from n2; the two are compared minute by minute.

## Node-dead rule (Kaspa Pulse)

A sender never fires at your node when: RPC fails for 5 s, the node is not synced, sink lag is over 60 s on a fresh read, or the virtual DAA score is still for 30 s. It pauses, you fix (watcher, restart, lower load), and it resumes by itself when a fresh read is healthy (`node-dead`, `node-back`). Public nodes: about 20 errors in a row cools that node for 30 s. A failed submit is a reject, never "sent".

## Your desk node

- kaspad 2.1.0, testnet-10, **pruned, not archival**, UTXO index on, default mempool scale, peers and RPC on loopback only, no tunnel, no public peer port, a watcher that restarts it on the same flags.
- Keep ≥ 30 GB disk and ≥ 4 GB RAM free; if either gets tight, prune your own old logs first (never the chain); lower the rate only as a last resort, logged as a `deviation`.
- **Never submit to n2.**

## Coins

Your own wallet; each sender its own coins. Need: your rate × 6,600 s (110 storm minutes) transactions × your real grams per transaction × 250 sompi/gram average. Hold **1.5 times** that before R1. Check every 15 minutes; top up in small, memory-safe batches; never list a whole address in one call; never re-add a spent or claimed coin; log every `coin-refresh`. If the buffer runs low, lower the rate and log a `deviation`; do not stop.

## Your miner

Exactly **one** miner, one thread, on your own node only, paying Build's own address (kept out of git). On at R2 if synced; off while your node is unsynced; off at END.

## One paste, 3 PowerShell windows

If stp has to paste anything, give him **one** PowerShell block and nothing more. It opens **3 separate windows**, one job each: (1) **NODE**: desk node, its watcher, the miner; (2) **SENDERS**: desk (75%) and public (25%) senders, started and stopped by the clock table above; (3) **MONITOR + COUNT**: health reads, the loggers and the live chain-count reader. Each window has a clear title.

## Log everything

From R1 to END, as in [LOGGING.md](../LOGGING.md), `step` = phase (`P2`, `P4` … `P7`, `idle` when your storm is off): `sec.csv`, `minute.csv`, `public-nodes.csv`, `rejects.csv`, `miner.csv`, `host.csv`, `node.csv`, `events.csv`, `gates.csv`, `ntp.csv`, `pulse-method-minute.csv` (live); after END `chain-count.csv`, `block-mass.csv`, `steps.csv` (one row per phase) and `SUMMARY.md`. Per-transaction logs and ids stay on the desk.

## Handoff files for the bot

stp carries these. Format: [LOGGING.md](../LOGGING.md#handoff-files-build-to-the-bot).
1. **`FOR-TN10-OPS-ready-mintrush.txt` at R2:** ready, node synced, UTXO index, senders, coins, fee pair, storm rate, miner, disk free, count reader running.
2. **`FOR-TN10-OPS-result-mintrush.txt` by T0+05:00:** per phase: mean sent, submit-ok, accepted, rejects; your network unique per minute summary (peak complete minute, mean, median per phase); every change and fix.

## Halt

Only stp halts. Then: stop the watchdog first, senders to 0, wait 10 s, hard-kill every sender and the miner by process name, check nothing restarts. The node and the count reader stay up; the count still runs.

## After END

1. **END (T0+02:30):** senders stop, miner off, live loggers stop. Node stays up.
2. **COUNT (T0+02:30–04:30, before pruning):** `chain-count.csv` and `block-mass.csv` per minute from T0−01:00 to END, from your node.
3. `steps.csv`, `SUMMARY.md` and the result file.
4. **By T0+05:30:** commit to `logs/build/` in **STP-KAS/tn10-storm-plan-mint-rush** (renamed to `tn10-storm-YYYY-MM-DD` by then). Counts only. The repo is public: leak-scan first.

## Never

No IP, host, port, key, seed, txid, covenant id, wallet address or local path in the repo. No tunnel. No tags, no releases, no force-push. No minting. No second miner. No archival. No storm fee other than 200/300. No change of storm rate inside the window except a logged `deviation`. No sending to n2. No more than 25% of senders on the public nodes. No firing at a dead node. No stopping before END unless stp halts.
