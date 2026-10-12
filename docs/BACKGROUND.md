> **Experimental. We are just trying this.** [Disclaimer](../DISCLAIMER.md) · [Home](../README.md) · TN10 only. Mainnet: **not tested (label B)**.

# Background: what a KCC-20 mint is, and why a rush stresses the network

Written for a smart non-developer first. Each piece of jargon is explained once. Every number links its source. Source numbers like [S12] point to [SOURCES.md](SOURCES.md).

## 1. Words used

- **Transaction, output, input.** A Kaspa transaction spends earlier **outputs** (its **inputs**) and creates new outputs. An unspent output is a **UTXO**, or simply a coin.
- **Mempool.** The waiting room of a node: transactions it has accepted but no block has included yet.
- **Block, 10 blocks per second.** TN10 aims at about 10 blocks a second [S6].
- **Mass.** Kaspa's measure of how much room a transaction takes in a block, in **grams**. There are three budgets per block: **compute** 500,000, **storage** 500,000 and **transient** 1,000,000 grams [S6].
  - **Compute mass** grows with size and script work. A plain one-input, one-output payment is about 1,624 grams, so about 308 fit in a block, about 3,080 per second at 10 blocks per second [S6].
  - **Storage mass** (from KIP-9) charges transactions that create small outputs out of big ones, so the network does not fill up with dust. The charge depends on the outputs relative to the inputs, not on a fixed threshold: in the 2024 KRC-20 beta, splitting a large coin into a big change output and a 0.2 KAS output gave about 25,000 grams per transaction [S30].
  - **Transient mass** charges raw size for a short time.
- **Feerate.** Fee divided by mass, in **sompi per gram** (1 KAS = 100,000,000 sompi on mainnet; 1 tKAS likewise on TN10). Miners pick transactions with a weight of feerate cubed, so a higher feerate wins far more often, but it is a weighted draw, not a strict queue [S20, §4].
- **Fee estimate.** `getFeeEstimate` on a node returns a **priority** bucket, **normal** buckets and **low** buckets: suggested feerates for "next block", "soon" and "eventually" [S17, S8].
- **Covenant.** A script on an output that says how that output may be spent and what the spending transaction must create. Kaspa checks it in consensus, like a signature.
- **KCC-20.** The fungible-token covenant convention, **Last Call, not Final** [S28]. Michael Sutton: a properly written KCC-20 covenant gets layer-1 enforcement of its rules; KCC-20 itself is a spec [S26].
- **Indexer.** A separate program that reads the chain and builds a database (balances, token holdings). Not consensus. It can lag or freeze [S13].

## 2. What a KCC-20 mint is

In plain words: a **minter** is one special output on the chain. It holds a counter of how many tokens are still left to issue (the **remaining allowance**) and the most one mint may take (the **per-mint limit**). To mint, anyone builds a transaction that:

1. spends the minter output,
2. creates a **new minter output** with the counter lowered,
3. creates a **token output** for the recipient, holding the minted amount and a small KAS **carrier** amount, and
4. pays a fee from the minter's own coin.

The covenant checks the rules: the amount is positive, at most the per-mint limit, at most what is left, and the minter's KAS deposit is kept at the same value [S29]. No one has to sign for the minter; that is what "public" or "permissionless" mint means [S20]. The KCC-20 spec itself defines token state and transfers; issuance (the public mint) lives in the reference app `KCC20PublicMint` [S28, S29].

**Our measured mints (TN10, 11 Oct 2026):** tn ops2's bot landed **2,488** odd-leaf mints of `stp trump2028`, 0 rejects, one about every 2 to 3 seconds, on a quiet network [S22]:

| Per mint | Value | Label |
|---|---|---|
| Inputs | leaf minter + payer coin | A |
| Outputs | emptied minter, token output (4,056 units), 1 tKAS carrier, change | A |
| Compute mass | 9,117 grams | A |
| Transient mass | 26,068 grams | A |
| Storage mass | 61,710 to 76,586 grams (storage mass sets the fee) | A |
| Feerate | 204 sompi/gram | A |
| Fee per mint | 0.126 to 0.156 tKAS | A |
| Fees, all 2,488 | 322.74917112 tKAS (plus 2,488 tKAS of carriers) | A |

Grok Build's even half (9 Oct) paid on 80,000 grams of storage mass per mint at 100 sompi/gram, 0.08 tKAS each, and needed **4,999 split transactions** first so that 5,000 minters could work in parallel [S23].

## 3. Why a rush stresses the network

**a) Mints are heavy in the budget that binds.** With storage mass 61,710 to 76,586 grams, the 500,000-gram storage budget holds about **6 to 8 mints per block**. Transient (1,000,000 / 26,068 ≈ 38) and compute (500,000 / 9,117 ≈ 54) allow more, so storage binds first. At 10 blocks a second that is roughly **60 to 80 mints per second** if blocks carried nothing but mints. **Arithmetic from measured mass and the block budgets; not a measured rate. Needs testing.**

**b) The carrier size moves the ceiling a lot.** Our model (rusty-kaspa `calc_storage_mass`) puts a thin 0.2 carrier at 232,955 grams (2 per block), a 1 carrier at 73,069 grams (6 per block), and a 100 carrier at 653 grams (765 per block) (amounts in KAS on mainnet or tKAS on TN10; the mass rules are the same) [S20, packing table]. Wallet defaults decide how fast a rush can clear. That is the same lesson as the 2024 KRC-20 beta, where 0.2 KAS outputs made each transaction about 25,000 grams and only about 20 fit per block [S30]. **Model, not measured on the network.**

**c) Everyone races for the same minter output.** One minter output can only be spent once. When many people build a mint on the same minter at the same moment, the node takes the first and rejects the rest: on 7 Oct a second mint on the same output was rejected with "already spent by transaction … in the mempool" [S20, §3] (**A, one case**). Splitting the minter into many leaves lets mints run in parallel; that is why Build made 5,000 leaves [S23]. Michael Sutton names "contention over shared variables/state" as something multi-contract apps must design for [S26].

**d) It competes with ordinary traffic on price.** If minters pay high feerates, ordinary low-fee transactions wait. On TN10 under a 57-minute overload, the minimum fee waited up to 105 s and 10× the minimum always landed in 3.7 s or less [S8]. **A (TN10).**

**e) Mass budgets are separate.** Plain storm payments mostly use compute mass; mints mostly use storage mass. Whether a block template can pack both at once, so that a rush and a storm do not fully crowd each other out, is **open and needs testing**. This is one of the main things the A/B phases will show ([HYPOTHESIS.md](HYPOTHESIS.md), H3).

**f) Nodes and indexers wobble under load.** A restarted node drops its whole mempool [S9]; a node finishing IBD can keep already-accepted transactions in its mempool (upstream fix proposed, not tested by us) [S31]; the public TN10 indexer once froze for days and served spent coins [S10, S13]. A minter that trusts an indexer for the minter's current state will build on stale state.

## 4. KCC-20 vs KRC-20 in one paragraph

KRC-20 (Kasplex) puts token data in an inscription: a commit transaction locks KAS in a script-hash output, a reveal transaction spends it and shows the data; an off-chain indexer decides whether the mint counts [S32, S33]. KCC-20 puts the rules in a covenant that consensus checks; one transaction per mint, no indexer needed for validity. Full comparison, with every source: **[KCC20-VS-KRC20.md](KCC20-VS-KRC20.md)**.

## 5. What this background does not claim

- Nothing here is a mainnet measurement. Mainnet: **not tested (label B)**.
- The mints/second figures above are arithmetic, not measurements.
- KCC-20 is Last Call, not Final; mint mechanics may change [S28].
