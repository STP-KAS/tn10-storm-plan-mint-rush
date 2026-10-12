# KRC-20 launch history (2024) and what it means for the TN10 mint-rush test

Sources are third-party unless marked. Labels: **Verified** = confirmed by at least one linked source (third-party reporting, not our measurement); **Unverified (community report)** = only community posts or not found. No number here is ours.

## 1. Open beta, 30 Jun 2024

| Claim | Status | Source |
|---|---|---|
| KRC-20 beta started on mainnet on 30 Jun 2024 | Verified | [kasmedia, Kasplex Series One](https://kasmedia.com/article/introducing-kasplex-krc-20-krc-721-tokens) |
| Mint/deploy sent a 0.2 KAS output; with KIP-9 storage mass a typical tx was about 25,000 grams (block limit 500,000), so about 20 per block | Verified | [kaspa-ng ecosystem update](https://github.com/kaspa-ng/kaspa-ecosystem-updates/blob/main/README.md#krc20-beta-in-the-past) |
| About 40 per block (vs >200 expected) | Verified, as a second estimate | [kasmedia](https://kasmedia.com/article/introducing-kasplex-krc-20-krc-721-tokens) (so "20-40 per block" spans the two sources) |
| The 0.2 KAS output came from Kasware's mint flow | Unverified (community report) | [summary repost](https://kaspa.cointoken.news/alright-let-me-e07x669t/) |
| Mempool backlog peaked around 37,000 tx; the 1 KAS mint fee gave ~4,000 sompi/gram, so low-fee user txs had almost no chance | Verified | [kaspa-ng ecosystem update](https://github.com/kaspa-ng/kaspa-ecosystem-updates/blob/main/README.md#krc20-beta-in-the-past) |
| Stuck low-fee txs; wallets lacked fee estimation and RBF; RPC gateways overloaded | Verified (stuck txs, missing fee adjust: kasmedia, quoting Michael Sutton on RBF and dynamic fee estimation); RPC overload: Unverified (community report) | [kasmedia](https://kasmedia.com/article/introducing-kasplex-krc-20-krc-721-tokens), [r/kaspa](https://www.reddit.com/r/kaspa/comments/1dsgn4f/summary_of_the_first_day_of_krc20s_on_kaspa/) |
| L1 stayed up ("mainnet worked flawlessly"; Wyborski: mainnet ran "upwards of 100TPS without breaking a sweat"; KIP-9 turned storage damage into temporary congestion, tx selection did not yet weigh mass) | Verified | [kasmedia](https://kasmedia.com/article/introducing-kasplex-krc-20-krc-721-tokens) |
| "First real fee market" | Unverified (characterisation) | none found |
| NACHO was deployed in the beta | Unverified (community report) | not found in the sources above |
| Launch was delayed because of the beta | Partly verified: mainnet came 15 Sep after "two months of stress testing" | [Kasplex announcement repost](https://minerbases.com/krc-20-mainnet-launch-announcement/) |

## 2. Mainnet launch, 15 Sep 2024 12:00 UTC

| Claim | Status | Source |
|---|---|---|
| Launch 15 Sep 2024 12:00 UTC; Kasplex reported 90M+ KRC-20 test txs | Verified (Kasplex's own claim) | [announcement repost](https://minerbases.com/krc-20-mainnet-launch-announcement/), [kastop](https://www.kastop.com/Item/401.aspx) |
| First 24 h: 13M+ txs, average ~200 TPS, peak ~1,000 TPS (Kaspalytics), 80k+ addresses, 156+ tokens (NACHO, GHOAD, SOMPS) | Verified (one analyst's recap, relayed by press) | [CaptainAltcoin, 17 Sep 2024](https://captainaltcoin.com/heres-why-kaspas-launch-of-krc-20-tokens-was-a-big-success-insane-numbers-says-one-kas-holder/) |
| KASPER among the big early tokens | Unverified (community report) | [r/kaspa](https://www.reddit.com/r/kaspa/comments/1fi3sz3/wasnt_this_supposed_to_be_the_next_new_thing/) |
| Kasware fee bug: leftover UTXO value used as fee (sent to miners instead of change); fix 0.6.38.3 with a high-fee warning; a second bug made mints with fees below the 1 KAS minimum | Verified in part: Kasware's update notice is relayed; the bug mechanics are community reports. It was a **fee/change** bug; calling it an "RBF bug" is **unverified** | [Kasware notice repost, 15 Sep 2024](https://kaspa.cointoken.news/please-help-spread-xwcpofcr/), [r/kaspa](https://www.reddit.com/r/kaspa/comments/1fi3sz3/wasnt_this_supposed_to_be_the_next_new_thing/), [r/kaspa](https://www.reddit.com/r/kaspa/comments/1fjxr4a/anyone_else_have_this_issue_regarding_kaswarekrc20/) |
| High priority fees; exchange/wallet friction (suspended withdrawals) | Unverified (community report) | [r/kaspa](https://www.reddit.com/r/kaspa/comments/1fhju9s/what_is_going_on_with_krc20/) |
| Shai Wyborski called it a "legendary PoW stress test" | Unverified, dropped as a quote | not found |

## 3. Later totals (as of about 7 Oct 2024)

| Claim | Status | Source |
|---|---|---|
| ~40M KAS cumulative fees, 30M+ KRC-20 txs, 900+ tokens | Verified (exchange research report) | [CoinEx Insight](https://www.coinex.com/en/insight/report/krc20-innovative-token-protocol-driving-kaspa-ecosystem-development-6703aa228b08cff65dcd2890) |
| Daily active addresses from <20k to a peak of 90k+; daily txs peaked above 8M | Verified (same report, citing kas.fyi) | same |

## 4. What is different with KCC-20

- **Rules on chain.** A KRC-20 token's state is interpreted by off-chain indexers, not L1 consensus ([kaspa-master-file, "KRC-20 incident"](https://github.com/STP-KAS/kaspa-master-file#readme)). A properly written KCC-20 covenant gets **L1 enforcement** of its rules; KCC-20 itself is a spec (Last Call, not Final) (Michael Sutton, [20 Sep 2026](https://x.com/michaelsuttonil/status/2101641365419417797), via [kaspa-master-file, "KCC-20 wording"](https://github.com/STP-KAS/kaspa-master-file#readme)). A Sutton post on "mini-contracts" was not found in the master file; not quoted.
- **One transaction per mint, not two** (commit/reveal), with storage mass 61,710-76,586 grams per mint on TN10 (11 Oct 2026, our measurement) ([tn-master FINDINGS 16](https://github.com/STP-KAS/tn-master/blob/main/FINDINGS.md), [KCC20-VS-KRC20.md](KCC20-VS-KRC20.md)).
- **A mint on an already-spent minter is rejected by consensus before it pays** (seen once on TN10).
- **Mechanics: to verify against spec** ([kaspanet/kccs](https://github.com/kaspanet/kccs)) before the run.
- **Node side since 2024:** rusty-kaspa added replace-by-fee in the mempool ([commit 56c19c9, #499](https://github.com/biryukovmaxim/rusty-kaspa/commit/56c19c995c096ca0b13651003b71c0ac6bb16ff4), mirror) and fee estimation (`getFeeEstimate`, used in our storms).

## 5. Lessons for the TN10 mint-rush test

| 2024 failure | Test metric |
|---|---|
| Small outputs → high storage mass → ~20-40 tx per block | Storage mass per mint and mints per block (`mass per block` in [plan/METRICS.md](../plan/METRICS.md)); compare carriers |
| Mempool backlog ~37k, users crowded out | Mempool size per node per minute; time to accept p50/p90/p99 for ordinary payments next to mints |
| No dynamic fees, stuck low-fee txs | Fee-estimate buckets (priority/normal/low) per minute; fee per mint; does capped auto-raise (ADVICE 1, 3) clear the queue |
| Wallet used leftover value as fee / bad change handling | Fee paid vs fee intended per mint; change output present and correct on every mint; rejects by reason |
| Fees under the protocol minimum, mints failed but paid | Rejects by reason; minted vs paid |
| RPC gateways overloaded | Node-dead rule events and RPC error rate per node |
