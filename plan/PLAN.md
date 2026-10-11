> **Experimental. We are just trying this.** [Disclaimer](../DISCLAIMER.md) · [Home](../README.md) · TN10 only. Mainnet: **not tested (label B)**. **Decided 11 Oct; date to be set, after the rehearsal passes.** Numbers marked *draft* are starting constants frozen at arming.

# PLAN: KCC-20 mint rush under storm load

Times will be written **UTC first, CEST second** once T0 is set. Until then they are offsets from T0. Source numbers [S..] are in [../docs/SOURCES.md](../docs/SOURCES.md).

## Decisions (11 Oct 2026)

stp left these to TN10 ops ("you choose", 17:20Z / 19:20 CEST). They replace the matching *draft* items below.

| # | Decision | Was |
|---|---|---|
| 1 | **Token launch and split: tn ops2** | open decision 5 |
| 2 | **Volume: 2, 5, 10, 20 mints/s per step (about 44,400 mints in phases 3 to 5), gated on the [REHEARSAL](#rehearsal-before-any-date) passing first** | open decision 3 |
| 3 | **Build: second storm side plus the second node for the chain count. All minting (arms A and B, phases 6 and 7) stays on the bot side**, so the fee A/B is not mixed with a second machine | open decision 2 |
| 4 | **Phases 6 (contention) and 7 (carrier): in. KRC-20 / KNS comparison arm: out for now** | open decisions 6, 7 |
| 5 | **Arm A fixed fee = the phase 1 (baseline) normal fee estimate**, read once and frozen | open decision 8 |
| 6 | **Run date: to be set**, after the rehearsal passes | open decision 1 |
| 7 | **Storm rate: bot side 1,200 tx/s flat** in every storm phase (2, 4, 5, 6, 7), fees fixed 400/600. Basis: the bot's 11 Oct 5-hour window mean was 1,211.87 tx/s accepted ([steps.csv](https://github.com/STP-KAS/tn10-storm-2026-10-11/blob/main/logs/bot/steps.csv), `window` row). **Build side: Build's last clean rate, set in GO-BUILD at arming** (Build's 11 Oct result files are not in the repo yet), fees fixed 200/300 | open decision 4 (stp: "solve it", 17:26Z / 19:26 CEST) |
| 8 | **Token: room for 60,000 mints. Per-mint limit 1,000 tokens, total supply 60,000,000.** Use: 44,400 (phases 3 to 5) + 6,000 (phases 6 and 7 at 5 mints/s × 10 min each) = 50,400, plus 9,600 spare (about 19%). The **rehearsal runs on its own small launch** (same limit, supply 10,000,000 = 10,000 mints; it needs 8,100), so a failed rehearsal never eats run supply. tn ops2 launches and splits both; split tree: [Token](#the-token) | open decision 5 (stp: "solve it", 17:26Z / 19:26 CEST) |

**Nothing is open except the date** (to be set, after the rehearsal passes). Run sheets: [go/](../go/README.md).

## Rehearsal (before any date)

A separate short run on a **quiet network** (no storm of ours), before the run date is set. Reason: the only KCC-20 mints we have landed came one every 2 to 3 s from one submitter ([S22]); the plan needs 20 mints/s from parallel minters.

- **Who:** tn ops2 launches and splits the separate rehearsal token (decision 8); the bot runs parallel minters with the arm B code (AUTO-FEE R1 to R11) and full logging ([../go/LOGGING.md](../go/LOGGING.md)).
- **Shape:** 5 min at 2 mints/s, 5 min at 5, then **10 min at 10 mints/s**, from at least 8 parallel minter processes on the wide pool.
- **Pass criteria (all must hold):**
  1. **Sustained 10 mints/s for 10 min** landed (mean of complete minutes ≥ 10, no complete minute under 8).
  2. **Mint success ≥ 99%** (landed within 120 s of first submit / attempts, bumps not counted as attempts).
  3. **0 coin-in-flight errors.**
  4. Time to accept, p90 ≤ 10 s.
  5. 0 `script` rejects (the covenant never refused a mint we built).
  6. Every logger wrote every minute; `confirm.csv` and `mint-minute.csv` filled; host RAM stayed above the 1.5 GB floor.
  7. Leak-scan clean on the pushed logs.
- **Fail:** fix, log the fix, rehearse again. No run date until a rehearsal passes. The rehearsal result is pushed to this repo under `logs/rehearsal/`.

## Rules carried over (do not change without a logged deviation)

1. **Plan first, published with its commit SHA; deviations listed** (Kaspa Pulse, [S35] method 1).
2. **10-minute baseline first** (Kaspa Pulse, [S35] method 2).
3. **Fixed steps, not one blast** (Kaspa Pulse, [S35] method 3).
4. **Saturation defined before T0:** accepted < 95% of submitted for 60 s ([S7] input 3).
5. **Node-dead rule** for every sender and every minter: RPC failing 5 s, not synced, sink lag over 60 s, or DAA score still for 30 s means stop and fail over; a failed submit is a reject, never "sent" (Kaspa Pulse, [S7] input 12; [S24]).
6. **COUNT within two hours of END,** from our own node, before pruning (Kaspa Pulse, [S7] input 14; [S24] "After the run").
7. **Headline = network unique accepted** per complete UTC minute; cross-check with **Kaspa Pulse's counter: accepted in blocks, two independent nodes, per UTC minute, by block time**; every count includes about **56 tx/s** of base load ([S2]).
8. **Counts only in git.** No IP, host:port, key, seed, txid, covenant id, wallet address, tunnel or local path. Per-transaction logs stay on the machine that made them ([S24] Format).
9. **No mempool-scale tricks:** every node we submit to stays on the default mempool scale (the 0.1 scale panicked at 100,001 in kaspad 2.1.0) ([S11], [S6] "How a full mempool panics").
10. **Pace, do not stop and start** ([S15] lesson 7; tn-master ADVICE 10).

## The token

A **new KCC-20 Last Call public-mint launch** on TN10, just for this test (trump2028's 5,000 leaves are all minted: even half by Build [S23], odd half by the bot [S22]). Same reference app and builder as trump2028: `KCC20PublicMint`, argent-lang/kcc20-reference `c8a08711` [S29, S22].

- **Supply and limit (decision 8):** per-mint limit **1,000**, supply **60,000,000** (60,000 mints). Rehearsal: a separate launch, limit 1,000, supply 10,000,000.
- **Leaves (minters), sized to the parallel minters:** split tree as in [S20] (1, 2, 4, … rounds). Each minter process owns its leaves, so mints on one leaf are serial and do not collide, except in the narrow pool.
  - **Wide pool A: 1,024 leaves, wide pool B: 1,024 leaves**, each leaf with an allowance of **26 mints (26,000 tokens)**: 53,248 mints of room for phases 3, 4, 5 and 7 (they need 47,400). Phases 3 and 4 use both pools with arm A rules; phase 5 gives pool A to arm A and pool B to arm B. **32 minter processes per arm**, 32 leaves each; at the top step (10 mints/s per arm) a process mints about every 3.2 s and cycles through its 32 leaves, so a leaf is reused about every 100 s, long after its last mint landed.
  - **Narrow pool: 16 leaves**, each **300 mints (300,000 tokens)**: 4,800 mints of room (phase 6 needs 3,000), shared by **64 minter processes** (phase 6, H4): four minters race for each leaf on purpose.
  - The remaining 1,952,000 tokens stay on the root minter as reserve.
  - 2,064 leaves = 2,063 splits. Rehearsal launch: **256 leaves** (255 splits), 39 mints each, 8+ minter processes.
  - tn ops2 may change leaf counts at launch for a technical reason; it logs that as a `deviation` and writes the final counts (no ids) in [go/README.md](../go/README.md).
- **Carrier:** 1 tKAS on each token output (the measured shape, [S22]). Payer coins large (≥ 10 tKAS) so storage mass stays near the measured range and fee bumps do not shrink change into dust ([S12] item 7).
- **Minters:** independent processes, each with its own payer coins and its own leaves (wide pools), or racing for shared leaves (narrow pool), like independent users in a rush.
- **Volume:** decision 2; timing per phase: [go/README.md](../go/README.md#the-single-clock).

## Phases

| # | Phase | Duration | Storm (plain payments) | Mints | Fees | What it answers |
|---|---|---|---|---|---|---|
| R | **REHEARSAL** (separate day, quiet network) | about 20 min + setup | off | 2, 5, then 10 mints/s | arm B rules | Can our minters sustain the rate? [Pass criteria](#rehearsal-before-any-date) |
| 0 | PREP and gates | T0−60 to T0 | off | off | — | Nodes synced, UTXO index on, coins counted, NTP offset logged, loggers up 30 min before T0 ([S24]) |
| 1 | **Baseline** | 10 min | off | off | — | Base load (about 56 tx/s), fee estimate at rest, mempool at rest ([S35]) |
| 2 | **Storm only** | 30 min | on: **bot 1,200 tx/s flat** (decision 7; 11 Oct window mean 1,211.87 accepted, [S25] steps.csv), **Build at its last clean rate** | off | storm 400/600 sompi/gram (the 11 Oct GO fees, [S39]) | Reference for network unique accepted, mempool, fee estimate, probe times |
| 3 | **Mint rush only** | 4 steps × 5 min | off | fixed steps **2, 5, 10, 20 mints/s** (decision 2) | fixed: the normal estimate read in phase 1 | Mint mass, mints per block, time to accept, rejects, without a storm |
| 4 | **Mint rush + storm, fixed fee** | 4 steps × 5 min | on, as phase 2 | same steps | minters fixed as phase 3 | What a storm does to fixed-fee mints (H7 also via probes) |
| 5 | **Mint rush + storm, A/B** | 4 steps × 10 min | on, as phase 2 | same steps, minters split 50/50 by index parity | **A:** fixed as phase 3. **B:** capped auto fee ([AUTO-FEE.md](AUTO-FEE.md)) | H2: does the capped auto fee land mints faster, and at what cost? H3: does storm throughput hold? |
| 6 | **Contention** (in, decision 4) | 10 min | on | narrow pool, 64 minters on 16 leaves, **5 mints/s** target | B rules | H4, H6: how many lose the race, and do losers pay? |
| 7 | **Carrier** (in, decision 4) | 10 min | on | **5 mints/s**, half the minters with a 0.2 tKAS carrier | B rules | H5: does the thin carrier raise storage mass and slow the rush? |
| 8 | **END, then COUNT** | END to END+2 h | off | off | — | Chain count per minute before pruning (Kaspa Pulse) |

**Probes in every phase:** one plain self-payment per fee tier every 30 s at 1×, 2×, 5× and 10× the node's minimum feerate, timed from submit to acceptance on the virtual chain (the round 1 method, [S8]). They show what happens to ordinary users (H7).

**Between steps:** if saturation (rule 4) holds for 60 s, the step is marked *saturated* and the next step is still run for its fixed time, never extended. If a node trips the node-dead rule, its senders and minters fail over and the event is logged.

### Why this order

- Storm only before mint only, so the mint phases have a storm reference from the same day.
- Phase 4 before phase 5, so the A/B phase starts from a known fixed-fee picture.
- A and B run **at the same time, on the same leaves pool, same nodes and same coin sizes**; only the fee rule differs. Running them one after the other would mix in changes in network load.

## Roles (decided 11 Oct)

| Who | Does | Writes |
|---|---|---|
| **Bot (TN10 ops)** | Storm senders, **all minters** (arms A and B, phases 6 and 7), probes, the chain count on its node, the live Kaspa Pulse-method counter | `logs/bot/` |
| **Build** | **Second storm side** and **the second node for the chain count** (decision 3). No minting | `logs/build/` |
| **tn ops2** | **Launches and splits the token** (decision 1; it built the trump2028 odd mints, [S22]) | `logs/` |
| **Kaspa Pulse** | Independent outside count, if he wants to; stp asks him. We never ping him ourselves ([S39]) | his own files |
| **stp** | Date, go/halt, deviations | — |

## Coins (arithmetic)

Per mint about **1.13 tKAS**: 1 tKAS carrier plus about 0.13 tKAS fee at 204 sompi/gram ([S22]). Auto-fee minters pay more, up to their cap. Splits: about 0.08 tKAS each at 100 sompi/gram plus a 1 tKAS deposit per leaf, returned on reclaim minus about 0.02 tKAS ([S23]). The storm needs its own budget as on 11 Oct ([S39] Coins). Hold 1.5× the total before T0, as in GO-BOT.

With the decided steps: phases 3 to 5 = (2+5+10+20) mints/s × (300 + 300 + 600) s = **44,400 mints ≈ 50,000 tKAS** plus auto-fee extra, plus splits. Phases 6 and 7 add 6,000 mints (≈ 6,800 tKAS). Rehearsal: 8,100 mints ≈ 9,200 tKAS. Exact holdings per side: [GO-BOT](../go/bot/GO-BOT.md#coins), [GO-BUILD](../go/build/GO-BUILD.md#coins).

## Stop rules

- **HALT** by stp at any time: senders and minters to 0, then a hard kill, nothing restarted ([S25] CHANGES-DURING-RUN, 14:33:01 CEST).
- **Disk guard:** stop at 5 GB free; pruning can take about 16 GB in 30 minutes ([S15] lesson 4, tn-master ADVICE 14).
- **RAM guard:** extra processes only while box RAM ≥ 2.5 GB, never under 1.5 GB (the 11 Oct rule, [S25]).
- **Cap guard:** an auto-fee minter that hits its cap returns an error and pauses; it never drains ([S12] item 5).
- **Coins:** a side that runs out stops; that is logged, not hidden.

## Open decisions for stp

**All items were decided on 11 Oct (the date: to be set, after the rehearsal); see [Decisions](#decisions-11-oct-2026).** The list is kept as it was asked.

1. **Run date and T0** (then the repo is renamed to `tn10-storm-YYYY-MM-DD`).
2. **Build's role:** second storm side, contention minters, or second counting node.
3. **Mint volume target:** the draft steps (2, 5, 10, 20 mints/s; about 44,400 mints) or smaller/larger. The storage-mass arithmetic puts the block ceiling near 60 to 80 mints/s ([BACKGROUND §3](../docs/BACKGROUND.md#3-why-a-rush-stresses-the-network)).
4. **Storm rate** in phases 2, 4, 5: the draft is about 1,200 tx/s bot side.
5. **Who launches the token** (tn ops2 or Build), and the per-mint limit and supply.
6. **Optional phases 6 and 7:** run or skip.
7. **KRC-20 arm:** none planned. A comparison with inscription mints (KNS creates or a test KRC-20 ticker on Kasplex's TN10 indexer) would need its own tooling check first. **Needs testing.**
8. **Fixed-fee level for arm A:** the phase 1 normal estimate (draft) or the node minimum.
