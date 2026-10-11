> **Experimental. We are just trying this.** [Disclaimer](DISCLAIMER.md) · Kaspa Testnet-10 only; tKAS has no value. **This is a plan, not a run. No date is set.** Mainnet: **not tested (label B)**.

# tn10-storm-plan-mint-rush: a KCC-20 mint rush under storm load (plan)

Will be renamed to `tn10-storm-YYYY-MM-DD` once a run date is set (the [11 Oct naming rule](https://github.com/STP-KAS/tn-master/blob/main/repos/INDEX.md)). Part of the [tn-master](https://github.com/STP-KAS/tn-master) map.

## Map

| Path | What is in it |
|---|---|
| [README.md](README.md) | This page: map, intro, briefing, results placeholder |
| [DISCLAIMER.md](DISCLAIMER.md) | Experimental, TN10 only, certainty labels |
| **docs/** | |
| [docs/BACKGROUND.md](docs/BACKGROUND.md) | What a KCC-20 mint is, in plain words, and why a rush stresses the network |
| [docs/KCC20-VS-KRC20.md](docs/KCC20-VS-KRC20.md) | How KCC-20 minting differs from KRC-20 minting: where the rules live, transaction shape, mass, who validates, failure modes, fees, coins, migration |
| [docs/HYPOTHESIS.md](docs/HYPOTHESIS.md) | stp's hypothesis, and the narrower things this test can actually check |
| [docs/RISKS.md](docs/RISKS.md) | What can go wrong (machine, nodes, coins, spec), and the guard for each |
| [docs/SOURCES.md](docs/SOURCES.md) | Every source, numbered, with the exact link and file path |
| **plan/** | |
| [plan/PLAN.md](plan/PLAN.md) | Phases: baseline, storm only, mint rush only, mint rush + storm, and auto-fee vs fixed-fee minters (A/B) |
| [plan/METRICS.md](plan/METRICS.md) | What to measure and how: mint success, time to accept p50/p90/p99, fee per mint, rejects by reason, evictions, network unique accepted, mass per block, fee-estimate buckets |
| [plan/AUTO-FEE.md](plan/AUTO-FEE.md) | The capped automatic fee-raise algorithm under test, every rule tied to past evidence |
| **go/** | |
| [go/README.md](go/README.md) | Shared rules, the single clock (T0), roles, how GO works, arming. **NOT ARMED: rehearsal pending** |
| [go/bot/GO-BOT.md](go/bot/GO-BOT.md) | Bot GO: minting arms A/B, its storm side, probes, count on n2 |
| [go/build/GO-BUILD.md](go/build/GO-BUILD.md) | Build GO: second storm side, second counting node, no minting |
| [go/LOGGING.md](go/LOGGING.md) | Every log file and column, both sides |
| **logs/** | |
| [logs/README.md](logs/README.md) | Where the logs will land (`logs/bot/`, `logs/build/`, `logs/rehearsal/`); columns in [go/LOGGING.md](go/LOGGING.md) |

> ## Thank you, Kaspa Pulse ([@gokugalax](https://x.com/gokugalax))
>
> This plan is built on Kaspa Pulse's method. He counts our storms from the outside on his own two nodes, and these rules come from him ([every input](https://github.com/STP-KAS/tn-master/blob/main/credits/KASPA-PULSE.md)):
>
> - **The node-dead rule.** A sender (and here, every minter) stops when its node is dead: RPC failing, not synced, lag over 60 s, or DAA score still for 30 s. A failed submit is a reject, never "sent".
> - **The chain count right after END,** on our own node, before pruning removes the blocks.
> - **The "accepted in blocks" counter:** transactions accepted in blocks, **two independent nodes, per UTC minute, by block time**. It is the cross-check for every number this plan will report.
> - **The base load:** TN10 carries about **56 tx/s** without us. Every network count here includes it.
> - **The method:** write the plan first and publish it; a 10-minute baseline; fixed steps, not one blast; saturation defined before T0 (accepted under 95% of submitted for 60 s); NTP offsets at start and end; raw data next to the summary. That is why this repo exists before any run.

## Intro

### What

A test plan for **Kaspa Testnet-10 (TN10)**: many people minting the same new KCC-20 token at once (a **mint rush**), while the network is also busy with ordinary payments (a **storm**). Half of the minters keep a fixed fee. The other half watch the network and raise their fee automatically, with a hard cap. We compare the two.

- A **KCC-20 token** is a token whose rules are enforced by Kaspa itself through a covenant (a script that says how an output may be spent). Plain-words background: [docs/BACKGROUND.md](docs/BACKGROUND.md).
- A **storm** is what we have run on TN10 since 18 Sep: senders pushing simple transactions, then counting what the chain accepted ([tn-master](https://github.com/STP-KAS/tn-master)).

### Why

**The hypothesis is stp's** (11 Oct 2026): the first big real-world load on mainnet after smart contracts land may well be a mint rush, like the KCC-20 mints on TN10. If so, dapps, wallets and infrastructure should already have congestion monitoring with an automatic, capped fee raise in place ([tn-master ADVICE, hypothetical section](https://github.com/STP-KAS/tn-master/blob/main/ADVICE-TO-DEVELOPERS.md#hypothetical-scenario-mint-rush-after-smart-contracts-stp-11-oct-2026)). **The idea to test it as a mint rush under storm load on TN10 came from TN10 ops (the bot)**, and stp liked it.

There is a precedent. During the 2024 KRC-20 beta on mainnet, a mint wave built a mempool backlog of about 37,000 transactions, and ordinary users' low-fee transactions waited for hours ([kaspa-ng ecosystem update](https://github.com/kaspa-ng/kaspa-ecosystem-updates/blob/main/README.md#krc20-beta-in-the-past), third-party account). KCC-20 mints are built differently ([docs/KCC20-VS-KRC20.md](docs/KCC20-VS-KRC20.md)), so we do not assume the same thing happens. We test it.

What we already know, all TN10: paying the minimum fee under overload meant waits up to 105 s, 10× the minimum always landed in 3.7 s or less ([round 1](https://github.com/STP-KAS/tn10-storm-2026-09-25-r1/blob/main/findings/overload-30m-summary.md)); paying the "priority" estimate with no cap burned about 900 tKAS in 11.5 minutes ([stress findings, item 5](https://github.com/STP-KAS/tn10-storm-2026-09-25#not-addressed-by-pr-165-status-per-the-author-26-sep)); and 2,488 KCC-20 mints landed with 0 rejects on a quiet network on 11 Oct ([stp-trump2028](https://github.com/STP-KAS/stp-trump2028/blob/main/map/bot/FINDINGS.md)). Nobody here has tested a mint rush **under** storm load.

### How

1. **Baseline** 10 minutes, nothing of ours on the network (Kaspa Pulse's rule).
2. **Storm only**, at a fixed rate.
3. **Mint rush only**, in fixed steps of mints per second.
4. **Mint rush + storm**, fixed-fee minters.
5. **Mint rush + storm, A/B**: fixed-fee minters (A) next to capped auto-fee minters (B), same coins, same nodes, interleaved.
6. **COUNT** right after END, then the write-up.

Full plan: [plan/PLAN.md](plan/PLAN.md). Measurements: [plan/METRICS.md](plan/METRICS.md). The fee algorithm: [plan/AUTO-FEE.md](plan/AUTO-FEE.md).

## Briefing (one minute)

- **A KCC-20 mint is one heavy transaction.** Our 2,488 mints on 11 Oct each had a storage mass of 61,710 to 76,586 grams with a 1 tKAS carrier output ([bot FINDINGS](https://github.com/STP-KAS/stp-trump2028/blob/main/map/bot/FINDINGS.md)). A block has a 500,000-gram storage budget ([what-limits-tx-rate](https://github.com/STP-KAS/what-limits-tx-rate#why)), so only about **6 to 8 such mints fit in one block** (arithmetic, not a measured rate). A plain payment is about 1,624 grams of compute mass, about 308 per block.
- **A KRC-20 mint is two lighter transactions** (commit, then reveal), and its token rules are checked by an off-chain indexer, not by the node ([docs/KCC20-VS-KRC20.md](docs/KCC20-VS-KRC20.md)).
- **Mints compete for the same minter output.** A public KCC-20 minter is one output that each mint spends and recreates. Two mints on the same output: one wins, the node rejects the other as "already spent" (seen once on 7 Oct, [RESULTS.md §3](https://github.com/STP-KAS/stp-trump2028/blob/main/RESULTS.md)). A rush is a contention test as much as a throughput test.
- **The question for builders:** does a capped automatic fee raise keep mints landing in seconds during a rush plus storm, and what does it cost compared with a fixed fee? **Needs testing.**
- **Decided 11 Oct:** see below. **Only the date is open:** to be set, after the rehearsal passes. Run sheets: [go/](go/README.md) (**NOT ARMED: rehearsal pending**).

### Decisions (11 Oct 2026)

stp left these to TN10 ops ("you choose", 17:20Z / 19:20 CEST). Recorded here and in [plan/PLAN.md](plan/PLAN.md#decisions-11-oct-2026).

1. **Token launch and split:** tn ops2.
2. **Volume:** the draft steps, 2, 5, 10, 20 mints/s (about 44,400 mints), **only after a scale rehearsal passes** ([REHEARSAL](plan/PLAN.md#rehearsal-before-any-date)).
3. **Build:** second storm side, and the second node for the chain count. **All minting stays on the bot side**, so the fee A/B stays clean.
4. **Optional phases:** contention (6) and carrier (7) are **in**. A KRC-20 / KNS comparison arm is **out** for now.
5. **Arm A fixed fee:** the baseline (phase 1) normal fee estimate.
6. **Run date:** to be set, after the rehearsal passes.
7. **Storm rate** (stp: "solve it", 17:26Z / 19:26 CEST): bot side **1,200 tx/s flat** in every storm phase (11 Oct bot 5-hour mean 1,211.87 tx/s accepted, [steps.csv](https://github.com/STP-KAS/tn10-storm-2026-10-11/blob/main/logs/bot/steps.csv)); Build side at **Build's last clean rate, set in GO-BUILD at arming**.
8. **Token:** per-mint limit **1,000**, supply **60,000,000** (room for 60,000 mints: 44,400 in phases 3 to 5 + 6,000 in phases 6 and 7 + about 19% spare). The rehearsal runs on its own small launch (supply 10,000,000). tn ops2 launches and splits both, the split tree sized to the parallel minters.

## Results

**No run yet.** This section will hold, per phase: mint success rate, time to accept p50/p90/p99 per arm, fee per mint per arm, rejects by reason, network unique accepted tx/s (peak complete minute, mean, median), and the Kaspa Pulse cross-check. Until then nothing here is a result.

Counts only. No IP, host, port, key, seed, txid, covenant id or wallet address in this repo. Public TN10 node names are allowed.

---

Intentions are good; thought process is questionable. STP remains delusional.
