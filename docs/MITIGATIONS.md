> **Early thesis by stp.** Our own reasoning from the numbers and data, open to challenge. **Not a claim of absolute truth. Not advice, and not a proposal to Kaspa core developers.** [Disclaimer](../DISCLAIMER.md) · [Home](../README.md) · TN10 only. Mainnet: **not tested (label B)**.

# MITIGATIONS: what could keep ordinary payments flowing during a mint rush

Labels on every line: **[fact]** = sourced (link given); **[code]** = our reading of rusty-kaspa source at `master` commit [`01b532e`](https://github.com/kaspanet/rusty-kaspa/tree/01b532e) (22 Sep 2026), not a maintainer statement; **[reasoning]** = our thinking, not measured; **[measure]** = what our runs can count.

**Attribution:** Independent counts by Kaspa Pulse stand behind the numbers. The analysis and thesis here are stp's, based on those counts plus our minute data. Kaspa Pulse signs the counts; stp signs the conclusions.

## The starting rule

**[reasoning]** The network has to stay healthy **assuming worst-case token issuers**: careless, greedy or malicious. Anything that only works if every issuer behaves is good practice, not a defence. So the real defences are the ones nobody can opt out of: **the fee market, wallets, and general (type-neutral) node policy**, with consensus capacity underneath.

## What Kaspa already has (sourced)

- **[fact]** Covenants are live on L1 since Toccata (DAA 474165565); KCC-20 is at Last Call, not Final; DAGKnight is Proposed ([kaspa-master-file](https://github.com/STP-KAS/kaspa-master-file#readme)).
- **[fact]** KIP-9 storage mass (quadratic, against UTXO growth) is Active ([kaspa-master-file, research.kas.pa topic 159](https://research.kas.pa/t/quadratic-storage-mass-and-kip9/159)). A KCC-20 mint on TN10 had 61,710-76,586 grams of storage mass, so about 6-8 fit a block ([tn-master FINDINGS 16](https://github.com/STP-KAS/tn-master/blob/main/FINDINGS.md)).
- **[code]** Minimum relay fee 100,000 sompi per 1,000 grams (100 sompi/gram) ([`mining/src/mempool/config.rs`](https://github.com/kaspanet/rusty-kaspa/blob/01b532e/mining/src/mempool/config.rs) L20).
- **[code]** Mempool limits: 1,000,000 transactions, about 1 GB estimated size; transactions expire after 24 h; orphans capped at 500 (same file, L3-L15).
- **[code]** When the mempool is full, a new transaction evicts the **lowest-feerate low-priority** ready transactions, and is rejected (`RejectMempoolIsFull`) if everything left pays more ([`transactions_pool.rs`, `limit_transaction_count`](https://github.com/kaspanet/rusty-kaspa/blob/01b532e/mining/src/mempool/model/transactions_pool.rs)). This is type-neutral: feerate only.
- **[code]** Replace-by-fee exists ([`replace_by_fee.rs`](https://github.com/kaspanet/rusty-kaspa/blob/01b532e/mining/src/mempool/replace_by_fee.rs)); RBF since [rusty-kaspa #499](https://github.com/kaspanet/rusty-kaspa/pull/499) (22 Jul 2024).
- **[code]** Block templates are filled by a feerate-weighted sampling selector over the ready frontier ([`frontier.rs`](https://github.com/kaspanet/rusty-kaspa/blob/01b532e/mining/src/mempool/model/frontier.rs)). Since Toccata there is also a **per-block lane cap, keyed by subnetwork id**, and it is a **consensus rule**, not only node policy:
  - **Limits:** at most **50 lanes (distinct subnetwork ids) per block** and **1,000,000,000 gas per lane** ([`consensus/core/src/config/constants.rs`](https://github.com/kaspanet/rusty-kaspa/blob/01b532e/consensus/core/src/config/constants.rs#L94-L100) L94-L100, "Toccata lane limits"; the 50 is commented "at 10 BPS, allows a worst-case rate of 500 SMT lane updates per second").
  - **Enforced in block validation:** every non-coinbase transaction counts toward its subnetwork's lane; a block over 50 lanes fails with `ExceedsLanesPerBlockLimit`, a lane over its gas with `ExceedsGasPerLaneLimit` ([`body_validation_in_isolation.rs`](https://github.com/kaspanet/rusty-kaspa/blob/01b532e/consensus/src/pipeline/body_processor/body_validation_in_isolation.rs#L98-L120) L98-L120). The template selector applies the same two limits ([`frontier/selectors.rs`](https://github.com/kaspanet/rusty-kaspa/blob/01b532e/mining/src/mempool/model/frontier/selectors.rs#L35-L64) L35-L64; lane = `tx.subnetwork_id`, [`feerate_key.rs`](https://github.com/kaspanet/rusty-kaspa/blob/01b532e/mining/src/mempool/model/frontier/feerate_key.rs#L42-L44) L42-L44).
  - **Which subnetworks:** native (id all zeros, `FIRST_BYTE = 0`), coinbase and registry are built in; any other id must be a user lane `[4-byte namespace, 16 zero bytes]` ([`consensus/core/src/subnets.rs`](https://github.com/kaspanet/rusty-kaspa/blob/01b532e/consensus/core/src/subnets.rs#L172-L200) L172-L200). The native lane is one lane like the others, with zero gas.
  - **Which subnetwork a KCC-20 mint uses: the native one.** [code] The KCC-20 reference ([argent-lang/kcc20-reference `c8a08711`](https://github.com/argent-lang/kcc20-reference/tree/c8a08711)) builds its transactions through argent-runtime ([`Cargo.toml`](https://github.com/argent-lang/kcc20-reference/blob/c8a08711/Cargo.toml#L18-L20) L18-L20, rev `9a9f4b10`) and never sets a lane; argent-runtime's `TxContext` "defaults to ... the native lane with zero gas" unless `.lane(id, gas)` is called ([`crates/argent-runtime/src/context.rs`](https://github.com/argent-lang/argent/blob/9a9f4b107116d0b259fae629f200fa3b663e7e9d/crates/argent-runtime/src/context.rs#L298-L328) L298-L328). [fact] One of our own 11 Oct TN10 KCC-20 mints, read from the public TN10 API on 12 Oct 2026, has `subnetwork_id` all zeros (native) (txid not published, as per our counts-only rule).
  - **[reasoning] So:** the lane cap separates *subnetwork lanes* (e.g. future vprogs/L2 lanes with gas), not token types. KCC-20 mints and plain payments **share the same native lane**, so the lane cap gives ordinary payments **no separate room** during a KCC-20 mint rush. A covenant designer *could* put a token on its own user lane (argent supports `.lane()`); we have not seen a KCC-20 token do so, and the KCC-20 spec text was not checked for a lane rule (**unverified**).
- **[fact]** A node's fee estimate can be skewed by a stale mempool after IBD: rusty-kaspa #1146, fix PR #1147 open, not merged ([kaspa-master-file](https://github.com/STP-KAS/kaspa-master-file#readme)).
- **[fact]** Covenant UTXO contention and MEV at high BPS is an open discussion, no replies at our read ([kas-smiths topic 160](https://kas-smiths.org/t/high-bps-covenant-state-pressure-utxo-contention-storage-mass-and-settlement-frequency-seeking-corrections/160), via kaspa-master-file).

## The options

### 1. The fee market as it is (node + consensus, already live)

- **How:** blocks take the best feerate per gram; a full mempool evicts the cheapest. A payer who needs to land pays more, or bumps with RBF. **[code]**
- **Who:** nothing new; node policy and consensus mass limits.
- **Pros [reasoning]:** neutral, no one decides what is "ordinary"; mint rushes pay miners; already deployed.
- **Cons / risks [reasoning]:** during a rush, a payer who keeps the old fee waits; minters with large budgets can price small payers out for the length of the rush; the price signal only works if wallets read it correctly (#1147).
- **[measure]:** Run 1 probe payments at the normal estimate and at 1×/2×/5×/10× the minimum: time to accept p50/p90/p99 vs baseline; how far the normal estimate moves; whether eviction happens (`rejects.csv` mempool-full).

### 2. Wallets (wallet developers)

- **How:** read the estimate from more than one node, re-read under congestion, bump with RBF under a hard cap, back off on mempool-full, never "fire and forget" ([AUTO-FEE.md](../plan/AUTO-FEE.md)).
- **Who:** wallet and app developers. No protocol change.
- **Pros [reasoning]:** neutral, user-side, can ship any time; fixes the 2024 lessons (wrong fee, change lost, hours of waiting) at the point where they happened ([KRC20-HISTORY](KRC20-HISTORY.md)).
- **Cons / risks [reasoning]:** uneven adoption; auto-bumping by everyone can turn into a bidding race (fee spiral) with no gain in throughput; a stale-mempool node gives wrong numbers (#1147); an uncapped "priority" fee burned about 900 tKAS in 11.5 min on TN10 ([stress findings item 5](https://github.com/STP-KAS/tn10-storm-2026-09-25#not-addressed-by-pr-165-status-per-the-author-26-sep)).
- **[measure]:** later run L1 (auto-fee A/B): landed time and cost per arm; whether B arms raise the estimate for everyone.

### 3. Token-level throttles (token designers): voluntary good practice, NOT a defence

- **How:** a covenant can limit minting (per-block or per-time caps, staged supply, many minter leaves to spread contention, fair-launch windows). **[reasoning]**
- **Who:** each token issuer, by choice.
- **Pros [reasoning]:** less contention on one minter output, fewer losing mints, smoother load for that token.
- **Cons / risks [reasoning]:** **it hands responsibility to issuers, and a careless or malicious issuer simply won't throttle**; one unthrottled token is enough to fill blocks. Throttles can also create a race at each window opening (MEV-like ordering games around the minter output). So it is good manners, never something the network can rely on.
- **[measure]:** later run L2 (contention): losers per leaf, whether losers pay; compare wide pool (many leaves) vs narrow pool.

### 4. General, type-neutral node policy (node developers / operators)

- **How (ideas, [reasoning]):**
  - **Fee-rate eviction** (exists, [code] above); possible tuning: a dynamic minimum relay fee when the mempool is near full.
  - **Per-sender or per-outpoint limits:** cap how many unconfirmed descendants one coin chain may have in the mempool, or how many transactions one peer may relay per second.
  - **RPC submit rate limits** per connection (operator setting).
- **Who:** node policy; each operator; no consensus change.
- **Pros [reasoning]:** applies to every transaction type alike, so it stays neutral; limits spam from one source without judging content.
- **Cons / risks [reasoning]:** "sender" is cheap to fake on a UTXO chain: a rush from 2,000 wallets (our Run 1 design) is not one sender, so per-sender limits barely bite on a real crowd; peer limits can hurt honest high-volume services (exchanges, payment processors); policy that differs per node makes propagation uneven.
- **[measure]:** Run 1: mempool size per node, rejects by reason, whether any node evicts or rejects ordinary probes; Pulse's two-node per-minute count vs ours.

### 5. Type-based mempool share, e.g. "at most 70% of the mempool (or block) for KCC transactions"

- **How [reasoning]:** nodes classify transactions (covenant/KCC vs plain payment) and cap one class at a share, reserving the rest.
- **Who:** node policy (mempool/template) for a soft version; consensus for a hard block rule.
- **Pros [reasoning]:** a guaranteed lane for plain payments during a rush.
- **Cons / risks [reasoning]:**
  - **Neutrality:** the network starts judging what a transaction is for; KCC is far more than mint rushes (dapps, real tokens), and they would be throttled with the memes.
  - **Gaming:** classification follows script shape; a minter can wrap a mint to look "plain", or plain payments can be made to look like covenants; every rule invites a workaround.
  - **MEV-like effects:** a capped class becomes its own auction; ordering inside the 70% can be sold.
  - **Centralisation / consistency:** a soft policy differs per node and per miner (miners can ignore it); a hard rule needs a consensus change and a fixed definition of "type".
  - **Efficiency:** reserved space can sit empty while the other class waits.
- **Already in Kaspa:** **[code]** lane caps exist in consensus, keyed by subnetwork id, not by token type; KCC-20 mints use the native lane together with plain payments (see above). We found no type-based share for covenant transactions. **[reasoning]** A lane-based version of this idea would need tokens to mint on their own user lane, which again depends on issuers (see option 3).
- **[measure]:** Run 1 tells whether a reserved share is needed at all: if probes at the normal fee land in seconds during the rush, the case for it is weak.

### 6. Consensus capacity (core / KIPs)

- **How:** more space per second (higher BPS, mass limits), or mass rules that price heavy outputs (KIP-9, live). **[fact]** above.
- **Who:** consensus, via KIPs and hardforks.
- **Pros [reasoning]:** raises the ceiling for everyone, neutral.
- **Cons / risks [reasoning]:** slow, needs broad agreement; a rush can grow to fill new space; out of scope for a TN10 test desk.
- **[measure]:** mints per block and mass per block in Run 1 (6-8 mints per block is arithmetic, not measured yet).

### 7. Public congestion gauge (idea, 15 Oct 2026)

Kaspa Pulse noted that our mitigation ideas around early warning match something they already do: they read fee estimates per minute on two nodes. A public congestion gauge — mempool and block fill per minute — would give builders the early warning described. Worth thinking about together. **[reasoning / offer]** Not a claim; an open idea.

## Comparison

| # | Option | Who | Neutral? | Works with worst-case issuers? | Exists today | Main risk | What decides it |
|---|---|---|---|---|---|---|---|
| 1 | Fee market | node + consensus | yes | yes | yes [code] | small payers priced out during a rush | Run 1 probes |
| 2 | Wallets: estimate, capped bump, RBF | wallet devs | yes | yes | partly (RBF yes; wallet behaviour varies) | fee spiral, stale estimates (#1147) | L1 A/B |
| 3 | Token throttles | issuers | yes | **no**: good practice only | possible with covenants [reasoning] | issuers opt out; window races | L2 contention |
| 4 | General node policy | node devs / operators | yes | yes | fee eviction, orphan cap, expiry [code] | per-sender limits easy to dodge; uneven nodes | Run 1 rejects, mempool per node |
| 5 | Type-based share (e.g. 70% KCC) | node policy or consensus | **no** | partly | no: lane caps exist (consensus, 50 lanes/block) but KCC-20 mints and payments share the native lane [code] | neutrality, gaming, MEV-like auction, empty reserved space | only if Run 1 shows probes stuck |
| 6 | Consensus capacity | core / KIPs | yes | yes | KIP-9 live; DAGKnight Proposed [fact] | slow; demand fills space | mass per block |
| 7 | Public congestion gauge | independent counters / community | yes | yes | partly (Pulse already reads fee estimates per minute on two nodes) | needs reliable public data | whether builders use the early warning |

**A note on standards (stp's view).** Covenants are permissionless: anyone can build a KCC-style token standard on mainnet. In our view KCC-20 is the benchmark, because it comes out of the work of Kaspa's core developers and contributors and its rules are enforced on Kaspa itself, not by one off-chain indexer. Two KRC-20 episodes show why that matters: [the 2024 launch rush](KRC20-HISTORY.md#2-mainnet-launch-15-sep-2024-1200-utc), and [the September 2026 Kasplex indexer exploit](KRC20-HISTORY.md#6-the-september-2026-kasplex-indexer-exploit) ([Kasplex, 22 Sep 2026](https://x.com/kasplex/status/2102259081277059509); [go-krc20d release krc20d-3.01.260930](https://github.com/kasplex/go-krc20d/releases)), where forged transfers were credited off-chain while Kaspa itself was unaffected. Any other KCC-style standard should be checked against KCC-20 before people trust it with a mint rush. KCC-20 itself is at Last Call, not final, and testing it on TN10 under load is part of what earns that role.

**Why core moves carefully (stp's view).** Working on these tests, stp came to understand why Kaspa's core developers and contributors move carefully and ship only when ready. [The September 2026 Kasplex exploit](KRC20-HISTORY.md#6-the-september-2026-kasplex-indexer-exploit) hit an off-chain indexer, while Kaspa L1 itself was unaffected; our own storm runs keep surfacing edge cases that only appear under load (for example [rusty-kaspa #1147](https://github.com/kaspanet/rusty-kaspa/pull/1147)). Shipping only when ready is slower, but it is why the base layer has held, and it is why public load tests on TN10 are useful to that careful work.

## Conclusion (early thesis by stp, based on numbers and data, [reasoning])

Not a claim of absolute truth. The network must stay healthy assuming worst-case issuers, so we rank token throttles as good manners, not a defence. The defences we would lean on are type-neutral: the fee market (already live), wallets that read fees correctly and bump within a cap, and general node policy. A type-based share such as 70% for KCC is the option we like least: it breaks neutrality, invites gaming, and would hit useful covenant apps along with memes. We do not know yet whether any of this is needed. **Run 1 is the test:** if ordinary payments at the normal fee still land in seconds during a crowd mint, the fee market is doing its job and the honest answer is "no new mechanism needed". If they wait for minutes, we report the numbers and leave the design to the people who build Kaspa. Kaspa Pulse has offered that a public congestion gauge (mempool and block fill per minute) is something they could help with, given they already read fee estimates per minute on two nodes.
