> **Experimental. We are just trying this.** [Disclaimer](../DISCLAIMER.md) · [Home](../README.md) · TN10 only. Mainnet: **not tested (label B)**. **Draft, no date yet.**

# METRICS: what to measure, and how

One vocabulary with tn-master: [METRICS.md](https://github.com/STP-KAS/tn-master/blob/main/METRICS.md). TPS and tx/s are the same unit; what differs is **what is counted** and **over what window**. File formats: [../go/LOGGING.md](../go/LOGGING.md).

## Mint metrics (new for this test)

| Metric | Definition | Per | Why | Based on |
|---|---|---|---|---|
| **Mint attempts** | Mint transactions a minter built and submitted | arm, phase, step, minute | Denominator | — |
| **Mint submit-ok** | Submits a node took into its mempool | same | Sender health only, never "landed" | [S2] Submitted |
| **Mints landed** | Mint ids seen in the virtual chain's accepted ids | same | The real outcome | [S18] column A |
| **Mint success rate** | landed / attempts, within 120 s of submit (*draft* window) | arm, step | H2 | — |
| **Time to accept p50 / p90 / p99, max** | Submit until the id appears in the virtual chain's accepted ids, read from a `virtual-chain-changed` subscription | arm, step | H2; the 11 Oct `confirm.csv` has only its header row, so this was **not measured on 11 Oct** | [S8] method, [S25] confirm.csv |
| **Fee paid per mint** | fee in tKAS and feerate in sompi/gram, mean / p90 / max | arm, step | Cost side of H2 | [S22] |
| **Bumps per mint** | How many times an auto-fee mint was rebuilt at a higher fee | arm B | Is the algorithm busy or calm | [AUTO-FEE.md](AUTO-FEE.md) |
| **Cap hits** | Mints that reached the cap and paused | arm B | Is the cap too low | [S12] item 5 |
| **Rejects by reason** | `fee-too-low`, `mempool-full`, `orphan`, `double-spend` (incl. "already spent … in the mempool"), `mass`, `script` (covenant rule failed), `rpc-timeout`, `rpc-closed`, `other` | arm, second | H4, H6 | [S24] reasons + `script` |
| **Minter collisions** | Rejects whose conflicting input is a minter output | pool, step | H4 contention | [S20 §3] |
| **Evictions** | Our mints or probes that left a mempool without being accepted (gone from mempool, not in chain after 120 s) | arm | Feerate eviction under a full pool | [S11] 30,298 evictions |
| **Dropped on restart** | Pending mints lost when a node restarted; rebroadcast count | node | The mempool is not saved on restart | [S9] |
| **Coins-in-flight errors** | Builds that tried to spend a coin already spent by our own pending tx | minter | Without tracking, 1,322 of 1,327 games failed | [S12] item 4 |
| **Mass per mint** | compute, transient, storage, and the node's ordering mass | sample of 5 per minute | Which mass sets the fee (open, [KCC20-VS-KRC20 §3](../docs/KCC20-VS-KRC20.md#3-mass-and-storage-mass-kip-9)) | [S22], [S20] |

## Network metrics (same as 11 Oct)

| Metric | Definition | Based on |
|---|---|---|
| **Network unique accepted tx/s** (headline) | Each id once, per complete UTC minute, all senders, from the chain count after END; peak complete minute, mean, median | [S2], [S25] SUMMARY |
| **Kaspa Pulse's counter** | Accepted in blocks, two independent nodes, per UTC minute, by block time; live and after END | [S2], [S7] |
| **Mints per block, payments per block** | Accepted mint ids and other ids per chain block, per minute | H3 |
| **Mass per block** | Mean compute, storage and transient mass of chain blocks per minute, against the 500,000 / 500,000 / 1,000,000 budgets | [S6] |
| **Fee-estimate buckets over time** | `getFeeEstimate` priority, normal[0..], low[0..] every 5 s, per node we read | [S17], [S19] |
| **Mempool size over time** | Per node, labelled as that node's view (nodes disagree) | [S7] input 17 |
| **Probe time to accept** | Plain self-payments at 1×, 2×, 5×, 10× the minimum feerate every 30 s | [S8] |
| **Node health** | synced, sink lag, DAA score moving, restarts, pruning point moves | [S24] node.csv |
| **Host health** | CPU, RAM free, disk free, sender/minter processes up | [S24] host.csv; [S25] RAM rule |

## Windows

Per complete UTC minute for the headline; per step for comparisons; per arm inside a step for A/B. A 10- or 20-second peak is never a sustained rate ([S2] Windows). The **complete-minute rule** from 11 Oct applies ([S2]).

## Comparisons that will be reported

1. Storm-only vs mint-rush+storm: network unique accepted, probes, mempool, fee estimate (H3, H7).
2. Mint-only vs mint-rush+storm, fixed fee: mint success and time to accept (what the storm does to mints).
3. **A vs B inside phase 5**, step by step: success rate, p50/p90/p99, fee per mint, rejects (H2).
4. Narrow vs wide pool: collisions per landed mint (H4, H6).

Each with n, window, and label **A (TN10)**. Mainnet: **not tested (label B)**.
