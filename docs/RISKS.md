> **Experimental. We are just trying this.** [Disclaimer](../DISCLAIMER.md) · [Home](../README.md) · TN10 only. Mainnet: **not tested (label B)**.

# RISKS: what can go wrong, and the guard for each

Each risk names a time it already happened (or why it might) and the guard in the plan. Source numbers [S..] are in [SOURCES.md](SOURCES.md).

## Machine and nodes

| Risk | Seen before | Guard |
|---|---|---|
| **Box runs short of RAM; sender/minter processes stall** (open: this plan runs up to 18 storm senders plus 128 minter processes on the box; 11 Oct capped the bot at about 18 senders on RAM, tn-master LESSONS; feasibility **unverified**, the rehearsal uses 8+ processes) | 11 Oct: `events.csv` has 182 `sender-start` rows for at most 18 senders (relaunches after `sender-stall`); a box pause of about 4 minutes; extra senders only with ≥ 2.5 GB free ([S25] CHANGES-DURING-RUN, SUMMARY). KNS runs hit the OOM killer ([S34] finding 9) | RAM guard from 11 Oct; minters count against the same budget; stall detector restarts one at a time; stalls logged as `fix` |
| **Own node freezes** (DAA still, sink lag) | n2, 11 Oct 11:08–11:25Z (13:08–13:25 CEST) ([S25] WRITE-UP, [S3] finding 6) | Node-dead rule; minters fail over to public nodes; the chain count notes lumped minutes |
| **Mempool cap panic** on a node with a small mempool | kaspad 2.1.0 panicked at `100001 > 100000` with `--ram-scale=0.1` ([S11], [S6]) | Default mempool scale only; stop extra submitters while public mempools climb toward 100,000 ([S6]) |
| **Restart drops pending mints** | The mempool is not saved ([S9]) | Rebroadcast on reconnect (AUTO-FEE R9) |
| **Stale mempool after IBD** inflates a public node's mempool | Upstream issue, fix proposed in rusty-kaspa PR #1147 (open, not tested by us) ([S31]) | Mempool sizes labelled per node; never a conclusion from one node's mempool ([S7] input 17) |
| **Disk runs out during pruning bursts** | Up to about 16 GB in 30 min ([S15] lesson 4) | Disk guard: box node stops below 10 GB free (GO-BOT); Build ≥ 30 GB; COUNT before pruning |
| **UTXO index lost or rebuilt mid-run** | Rebuild took 18.5 min offline and about 13 GB ([S12] item 6) | Do not touch the index during the run; minters need a node with a UTXO index to read minter state |
| **Indexer serves stale coins or stale minter state** | Public TN10 indexer froze ≥ 3 days and served spent outputs ([S10], [S13]) | Read state from a node (AUTO-FEE R8) |

## Coins and fees

| Risk | Seen before | Guard |
|---|---|---|
| **Fee burn** when paying "priority" with no cap | ~900 tKAS in 11.5 min ([S12] item 5) | Two caps (sompi/gram and tKAS per mint) and pause at cap (AUTO-FEE R4) |
| **High fee on small coins backfires** (storage mass up, throughput down) | 150× fee on ~0.25 tKAS coins ([S12] item 7) | Payer coins ≥ 10 tKAS (AUTO-FEE R5) |
| **Coins reused while in flight** | 1,322 of 1,327 games failed ([S12] item 4) | In-flight set (AUTO-FEE R7) |
| **Coin fragmentation** kills builders that take only the first coin | vprogs carriers panicked on small first coins ([S12] item 3) | Pick a coin that covers carrier + capped fee, else consolidate first |
| **Running out of tKAS** mid-phase | 11 Oct budget rule: hold 1.5× need ([S39]) | Budget in [PLAN.md](../plan/PLAN.md#coins-arithmetic); a side that runs out stops and logs it |

## Method

| Risk | Seen before | Guard |
|---|---|---|
| **Counting the wrong number** (submit-ok or Processed as "landed") | Processed overstates unique by about a third ([S18]); senders "sent" to a dead node ([S7] input 12) | Mints count only when in the virtual chain's accepted ids; headline = network unique; Kaspa Pulse cross-check |
| **Time to accept not logged** | 11 Oct `confirm.csv` has only its header row ([S25]) | Subscription-based timing for every mint and probe ([METRICS.md](../plan/METRICS.md)) |
| **A and B not comparable** | — | Same time, same pool, same nodes, same coin sizes; only the fee rule differs |
| **Base load or other TN10 users change mid-run** | Base load about 56 tx/s ([S2]); covenant activity seen growing on Kaspa Stream on 11 Oct (stp's observation, not our measurement) ([S25]) | Baseline phase; Kaspa Pulse's independent count; label confounders |
| **Mint tooling not ready at scale** | The bot's 11 Oct mints were one about every 2–3 s from one submitter ([S22]) | Rehearsal on a quiet network first (2, 5, then 10 mints/s, [PLAN](../plan/PLAN.md#rehearsal-before-any-date)); no date until it passes |

## Spec and scope

| Risk | Why | Guard |
|---|---|---|
| **KCC-20 changes** | Last Call, not Final ([S28]; kaspa-master-file "KCC still open"); the missing Last-Call-Deadline is noted there for KCC-1 and KCC-2, and for KCC-20 only in [S20] FINDINGS §3 (unverified against kcc-0020.md) | Pin the reference commit used; re-read before T0 |
| **Over-reading TN10 as mainnet** | tKAS has no value; mainnet nodes, users and fees differ | Every page: TN10 only; mainnet **not tested (label B)** |
| **Disturbing other TN10 users** | TN10 is shared (KNS, covenant devs) | Fixed, announced window; stop rules; no attack framing |

## Leaks

No IP, host:port, key, seed, txid, covenant id, wallet address, tunnel or local path in git. Leak-scan before every push. Per-transaction logs stay local ([S24] Format).
