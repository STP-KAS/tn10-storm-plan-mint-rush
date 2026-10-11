> **Experimental. We are just trying this.** [Disclaimer](../DISCLAIMER.md) · [Home](../README.md)

# SOURCES

Every source this plan uses, numbered. Read on 11 Oct 2026 (UTC). Our own repos are under [STP-KAS](https://github.com/STP-KAS); "third-party" means not ours and not checked by us beyond reading it. Pages cite these as [S#].

## tn-master (the map of every TN10 storm)

| # | Source | Used for |
|---|---|---|
| S1 | [tn-master README.md](https://github.com/STP-KAS/tn-master/blob/main/README.md) | Headline per storm; Kaspa Pulse callout; repo layout standard |
| S2 | [tn-master METRICS.md](https://github.com/STP-KAS/tn-master/blob/main/METRICS.md) | Counter vocabulary; Kaspa Pulse's "accepted in blocks" counter; base load about 56 tx/s; complete-minute rule |
| S3 | [tn-master FINDINGS.md](https://github.com/STP-KAS/tn-master/blob/main/FINDINGS.md) | Findings 1, 6, 11, 13, 15 (mass ceiling, node freezes, PR #1147, fees, the hypothetical) |
| S4 | [tn-master ADVICE-TO-DEVELOPERS.md](https://github.com/STP-KAS/tn-master/blob/main/ADVICE-TO-DEVELOPERS.md), incl. [hypothetical section](https://github.com/STP-KAS/tn-master/blob/main/ADVICE-TO-DEVELOPERS.md#hypothetical-scenario-mint-rush-after-smart-contracts-stp-11-oct-2026) | Items 1 to 14; stp's hypothesis |
| S5 | [tn-master history/MASTER.md](https://github.com/STP-KAS/tn-master/blob/main/history/MASTER.md) | Every storm in order; "counters are not the same" |
| S7 | [tn-master credits/KASPA-PULSE.md](https://github.com/STP-KAS/tn-master/blob/main/credits/KASPA-PULSE.md) | Kaspa Pulse inputs 1 to 22: method, saturation rule (3), node-dead rule (12), COUNT (14), mempool per node (17), base load (20) |

## The mass ceiling and the Sep storms

| # | Source | Used for |
|---|---|---|
| S6 | [what-limits-tx-rate README.md](https://github.com/STP-KAS/what-limits-tx-rate/blob/main/README.md) | Block budgets (compute 500,000 / storage 500,000 / transient 1,000,000); 1,624-gram payment, about 308 per block, about 3,040–3,080/s; mempool panic |
| S8 | [tn10-storm-2026-09-25-r1 findings/overload-30m-summary.md](https://github.com/STP-KAS/tn10-storm-2026-09-25-r1/blob/main/findings/overload-30m-summary.md) | Fee tiers under a 57.5-min overload (1× p50 7.0 s, max 105 s; 10× max 3.7 s; 100× no better); probe method; mitigation ideas; floor 100 sompi/gram. Its "included TPS" is the Processed counter (overstates, [S18]) |
| S9 | [tn10-storm-2026-09-25-r1 findings/F2-mempool-not-persisted.md](https://github.com/STP-KAS/tn10-storm-2026-09-25-r1/blob/main/findings/F2-mempool-not-persisted.md) | A restart drops the mempool |
| S10 | [tn10-storm-2026-09-25-r1 findings/F3-public-api-stale-utxos.md](https://github.com/STP-KAS/tn10-storm-2026-09-25-r1/blob/main/findings/F3-public-api-stale-utxos.md) | Public indexer lists spent coins; "orphan" as the signal |
| S11 | [tn10-storm-2026-09-25-r1 findings/F1-mempool-cap-panic.md](https://github.com/STP-KAS/tn10-storm-2026-09-25-r1/blob/main/findings/F1-mempool-cap-panic.md) | `--ram-scale=0.1` cap panic; 30,298 feerate evictions |
| S12 | [tn10-storm-2026-09-25 README.md](https://github.com/STP-KAS/tn10-storm-2026-09-25/blob/main/README.md) (stress findings, items 2 to 7) | Fee setting ignored (2), coin fragmentation (3), coins in flight (4), no fee cap, ~900 tKAS in 11.5 min (5), UTXO index (6), storage mass vs high fees (7) |
| S13 | [tn10-storm-2026-09-25-indexer-stall](https://github.com/STP-KAS/tn10-storm-2026-09-25-indexer-stall) | Public indexer froze ≥ 3 days 15 h |
| S14 | [tn10-storm-2026-09-25 drafts/issue-fee-cap-multiplier.md](https://github.com/STP-KAS/tn10-storm-2026-09-25/blob/main/drafts/issue-fee-cap-multiplier.md) | `min(priority, k × normal)` cap; priority/normal 8.4× (E2) |
| S15 | [tn10-storm-2026-09-26-r4 README.md](https://github.com/STP-KAS/tn10-storm-2026-09-26-r4/blob/main/README.md#flaws-and-operational-lessons) and [upstream-issues/README.md](https://github.com/STP-KAS/tn10-storm-2026-09-26-r4/blob/main/upstream-issues/README.md) | Pacing works (lesson 7); pruning bursts > 15 GB (lesson 4); fee setting ignored |
| S16 | [tn10-storm-2026-09-26-r5 README.md](https://github.com/STP-KAS/tn10-storm-2026-09-26-r5/blob/main/README.md), [findings/round5-notes.md](https://github.com/STP-KAS/tn10-storm-2026-09-26-r5/blob/main/findings/round5-notes.md) | 150× fee backfire on small coins (storage mass) |
| S17 | [tn10-storm-2026-09-26-r6 README.md §4 Fee rule](https://github.com/STP-KAS/tn10-storm-2026-09-26-r6#4-fee-rule-user-1222-double-the-standard-so-we-have-priority) | `max(2 × normal[0], 200)` recomputed every 30 s |
| S18 | [tn10-storm-2026-09-26-r7 README.md, three TPS columns](https://github.com/STP-KAS/tn10-storm-2026-09-26-r7/blob/main/README.md) | Unique accepted 267.2/s vs Processed 357.2/s; virtual-chain-changed accepted ids |
| S19 | [tn10-storm-2026-09-26-r8 README.md, Headline](https://github.com/STP-KAS/tn10-storm-2026-09-26-r8/blob/main/README.md) | Five covenant designs under storm: 0 illegal spends accepted; 2× normal kept next-block latency for ≈20% more fee |

## KCC-20 on TN10 (ours)

| # | Source | Used for |
|---|---|---|
| S20 | [stp-trump2028 README.md](https://github.com/STP-KAS/stp-trump2028/blob/main/README.md) (packing table, split tree, KRC-20 beta context), [RESULTS.md](https://github.com/STP-KAS/stp-trump2028/blob/main/RESULTS.md) (§3 node submission and rejected conflict, §4 fee weight = feerate cubed), [LIVE.md](https://github.com/STP-KAS/stp-trump2028/blob/main/LIVE.md) (7 Oct mint masses and fee rule), [FINDINGS.md](https://github.com/STP-KAS/stp-trump2028/blob/main/FINDINGS.md) (Last Call reading) | Mint shape, model masses by carrier size, contention evidence |
| S22 | [stp-trump2028 map/bot/FINDINGS.md](https://github.com/STP-KAS/stp-trump2028/blob/main/map/bot/FINDINGS.md) | 2,488 odd-leaf mints, 11 Oct, 0 rejects; compute 9,117 / transient 26,068 / storage 61,710–76,586; 204 sompi/gram; 0.126–0.156 tKAS per mint; 322.74917112 tKAS total; one every 2–3 s |
| S23 | [stp-trump2028 map/build/FINDINGS.md](https://github.com/STP-KAS/stp-trump2028/blob/main/map/build/FINDINGS.md) | Genesis, 4,999 splits, 2,500 even mints, 2,500 reclaims, 0 rejects; storage 80,000 per mint at 100 sompi/gram |

## The 9 to 11 Oct storms

| # | Source | Used for |
|---|---|---|
| S24 | [tn10-storm-2026-10-11 go/LOGGING.md](https://github.com/STP-KAS/tn10-storm-2026-10-11/blob/main/go/LOGGING.md) | File format and columns; node-dead rule; reject reasons; after-run steps |
| S25 | tn10-storm-2026-10-11 logs: [SUMMARY.md](https://github.com/STP-KAS/tn10-storm-2026-10-11/blob/main/logs/bot/SUMMARY.md), [COMBINED.md](https://github.com/STP-KAS/tn10-storm-2026-10-11/blob/main/logs/COMBINED.md), [WRITE-UP.md](https://github.com/STP-KAS/tn10-storm-2026-10-11/blob/main/logs/WRITE-UP.md), [CHANGES-DURING-RUN.md](https://github.com/STP-KAS/tn10-storm-2026-10-11/blob/main/logs/bot/CHANGES-DURING-RUN.md), [steps.csv](https://github.com/STP-KAS/tn10-storm-2026-10-11/blob/main/logs/bot/steps.csv), [events.csv](https://github.com/STP-KAS/tn10-storm-2026-10-11/blob/main/logs/bot/events.csv), [confirm.csv](https://github.com/STP-KAS/tn10-storm-2026-10-11/blob/main/logs/bot/confirm.csv) | Headline; fee changes 400/600 → 560/840 (10:31:10Z) → 1,120/1,680 (12:08:40Z); RAM rule; n2 freeze; HALT; confirm.csv empty |
| S36 | [tn10-storm-2026-10-10](https://github.com/STP-KAS/tn10-storm-2026-10-10) ([go/LOGGING.md](https://github.com/STP-KAS/tn10-storm-2026-10-10/blob/main/go/LOGGING.md)) | First run with COUNT and the node-dead rule |
| S37 | [tn10-storm-2026-10-09](https://github.com/STP-KAS/tn10-storm-2026-10-09) | Kaspa Pulse's night count; duplicates in blocks |
| S38 | [tn10-storm-2026-10-01-analysis-public notes/LAUNCH.md](https://github.com/STP-KAS/tn10-storm-2026-10-01-analysis-public/blob/main/notes/LAUNCH.md) | 1 Oct fee rule normal × 3, floor 300 |
| S39 | [tn10-storm-2026-10-11 go/bot/GO-BOT.md](https://github.com/STP-KAS/tn10-storm-2026-10-11/blob/main/go/bot/GO-BOT.md) | Fees 400/600, cap 600, sit-out rule; coin budget rule (1.5× need); do not ping Kaspa Pulse |
| S35 | [tn10-storm-2026-10-04-questions plan/PULSE-README.md](https://github.com/STP-KAS/tn10-storm-2026-10-04-questions/blob/main/plan/PULSE-README.md), [plan/NEXT-STORM-PLAN.md](https://github.com/STP-KAS/tn10-storm-2026-10-04-questions/blob/main/plan/NEXT-STORM-PLAN.md) | Kaspa Pulse's questions and method: plan first, baseline, fixed steps |

## Inscriptions (KNS, ours) and KRC-20 (third-party)

| # | Source | Used for |
|---|---|---|
| S34 | [kns-tn10-testing README.md](https://github.com/STP-KAS/kns-tn10-testing/blob/main/README.md), [docs/runs/STORM-2026-10-01.md](https://github.com/STP-KAS/kns-tn10-testing/blob/main/docs/runs/STORM-2026-10-01.md), [docs/runs/R1.md](https://github.com/STP-KAS/kns-tn10-testing/blob/main/docs/runs/R1.md) | Commit/reveal envelope in P2SH; ~0.035 tKAS network fee per name; "losing reveal still pays"; 154,568 creates; 1 Oct KNS storm 9,509 names ramped to 3,600/min; finding 7 (stranded commits); finding 9 (OOM) |
| S30 | [kaspa-ng/kaspa-ecosystem-updates README.md, "KRC20 Beta in the past"](https://github.com/kaspa-ng/kaspa-ecosystem-updates/blob/main/README.md#krc20-beta-in-the-past) (third-party) | 2024 KRC-20 beta: 0.2 KAS outputs, ~25,000 grams, ~20 per block, ~37,000 backlog, ~4,000 sompi/gram, ordinary users waited hours |
| S32 | [Kasplex KRC-20 docs, mint](https://docs-kasplex.gitbook.io/krc20/technical-overview/krc-20-ops/mint) (third-party; JavaScript page, read through a search-index copy) | Mint JSON, `to` optional, cap behaviour |
| S33 | [KasWare KRC20 API docs](https://docs.kasware.xyz/wallet/developer-documentation/kaspa/kaspa-krc20) (third-party) | Commit to a P2SH address holding the script, then reveal |
| S40 | [codecustard krc20_example README](https://raw.githubusercontent.com/codecustard/kaspa/master/examples/krc20_example/README.md) (third-party, TN10 example) | "Every KRC20 operation requires two transactions"; status by reveal id, `opAccept` |

## KCC-20 spec, reference, statements

| # | Source | Used for |
|---|---|---|
| S26 | [kaspa-master-file README.md, row "KCC-20 wording"](https://github.com/STP-KAS/kaspa-master-file/blob/main/README.md) | Sutton 20 Sep (L1 enforcement; KRC-20 off-chain); Sutton 11 Oct migration reply ([X](https://x.com/michaelsuttonil/status/2109305350008475670)); Sutton 11 Oct KCC-20 vs Solana reply ([X](https://x.com/michaelsuttonil/status/2109315001034875348)) incl. "contention over shared variables/state" |
| S27 | [kaspa-master-file README.md, rows "KRC-20 to KCC", "KRC-20 incident", "Kas-Smiths"](https://github.com/STP-KAS/kaspa-master-file/blob/main/README.md) | Sutton 19 Aug snapshot preference; manyfest three ledgers; ZealousSwap / Kasplex indexer report (third-party); Kas-Smiths topic 160 on covenant UTXO contention (unanswered as read) |
| S28 | [kaspanet/kccs kcc-0020.md @ 3fbec524](https://github.com/kaspanet/kccs/blob/3fbec524/kcc-0020.md) | KCC-20 state; Last Call, not Final |
| S29 | [argent-lang/kcc20-reference contracts/public_mint.ag @ c8a08711](https://github.com/argent-lang/kcc20-reference/blob/c8a08711/contracts/public_mint.ag) | Public mint rules |

## Upstream node

| # | Source | Used for |
|---|---|---|
| S31 | [rusty-kaspa PR #1147](https://github.com/kaspanet/rusty-kaspa/pull/1147) (open as read; link only) | Stale mempool after IBD; not tested by us |
| S41 | [rusty-kaspa PR #1004](https://github.com/kaspanet/rusty-kaspa/pull/1004) (merged 15 May 2026; link only) | Network minimum relay fee raised |
| S42 | [rusty-kaspa PR #1147, comment by oskrcl, 11 Oct 2026 16:20 UTC](https://github.com/kaspanet/rusty-kaspa/pull/1147#issuecomment-6111076516), and [commit b4df2d8](https://github.com/kaspanet/rusty-kaspa/pull/1147/commits/b4df2d8fb334aa2d179a563591781a783b63082d) by atharaldsen, 16:22 UTC (PR open, no maintainer review as of 19:40 UTC; link only) | A stale post-IBD mempool skews the fee estimator: priority 1,644 vs 100, about 8-15× overpay until the 24 h expiry; a non-mining node does not self-correct; the test asserts ready count and mass go to 0 after revalidation. Not tested by us |

**Count: 41 numbered sources** (S1 to S42; S21 is not used). No comments or reactions were made on kaspanet or any third-party repo.
