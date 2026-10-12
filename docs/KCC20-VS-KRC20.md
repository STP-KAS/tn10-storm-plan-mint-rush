> **Experimental. We are just trying this.** [Disclaimer](../DISCLAIMER.md) · [Home](../README.md) · [Background](BACKGROUND.md) · TN10 only. Mainnet: **not tested (label B)**.

# KCC-20 vs KRC-20 minting, in plain words

Two ways to make a fungible token on Kaspa. They look alike to a user ("I minted a token") but load the network, fail and get checked in very different ways. That difference decides what a mint rush does. Source numbers [S..] are in [SOURCES.md](SOURCES.md). Anything we did not measure ourselves is marked.

## The short version

| | **KRC-20** (Kasplex) | **KCC-20** (covenant) |
|---|---|---|
| Where the token rules live | Off-chain. JSON text in an inscription, read and enforced by indexers such as Kasplex [S32, S33]. Michael Sutton: KRC-20 is off-chain interpretation [S26]. | On-chain. A covenant script that Kaspa consensus checks; "a properly written KCC-20 covenant gets L1 enforcement of those rules" (Sutton, 20 Sep) [S26]. |
| Spec status | Kasplex protocol docs [S32] | KCC-20 **Last Call, not Final** (kaspanet/kccs `3fbec524`) [S28] |
| Transactions per mint | **Two**: commit, then reveal [S33, S40] | **One** mint transaction [S22]. A public launch also needs a genesis and, for parallel minting, split transactions (4,999 splits for 5,000 leaves) [S23] |
| Who says the mint is valid | The indexer (`opAccept` on the reveal) [S40]. The node only sees ordinary script-hash transactions. | The node, in consensus. An invalid mint is rejected before it enters a block [S20 §3, S29]. |
| Mass per mint (our numbers) | Not measured by us for KRC-20. Third-party, 2024 mainnet beta: about **25,000 grams per transaction** with 0.2 KAS outputs, about 20 per block [S30] | **Measured, TN10, 11 Oct:** compute 9,117, transient 26,068, storage **61,710 to 76,586** grams per mint with a 1 tKAS carrier [S22] |
| Fee per mint (our numbers) | Not measured for KRC-20. KNS (same inscription pattern, TN10): about 0.035 tKAS network fee per name, commit + reveal, plus the KNS protocol fee [S34] | 0.126 to 0.156 tKAS at 204 sompi/gram; 322.75 tKAS for 2,488 mints [S22] |
| What a losing minter pays | Both transactions are ordinary and can be accepted and paid for even if the indexer later says the mint does not count. KNS: "a losing reveal still pays" [S34]. For KRC-20 mints past the cap: **needs testing** | A mint built on an already-spent minter output is rejected by the node and never pays (seen once, 7 Oct) [S20 §3]. **A, one case; needs testing at scale** |
| Failure under congestion | Indexer lag or divergence; commit stuck while the reveal waits; reveal "already spent" when retried against a lagging node [S34, finding 7] | Consensus rejects (double spend on the minter, orphan input), long waits at low feerate, minter contention [S20 §3, S8] |

## 1. Where the token logic lives

**KRC-20.** A mint is the JSON `{"p":"krc-20","op":"mint","tick":"…"}` (optional `to`) inside a script. Kasplex documents that each mint takes the amount fixed at deploy, and that a mint which would pass the cap gets the remainder [S32]. Kaspa consensus does not read that JSON. Indexers do. If an indexer has a bug, token state can be wrong while layer 1 is fine: the 20 Sep ZealousSwap incident was traced to a signature bypass in the off-chain Kasplex KRC-20 indexer, not Kaspa L1 (third-party report, relayed in the master file; the desk did not read the post-mortem itself) [S27].

**KCC-20.** The token state (amount, owner, owner scheme, borrow scheme, borrow guard, extension commitment) lives in covenant outputs [S28]. The public mint is the reference app `KCC20PublicMint`: the covenant requires amount > 0, ≤ per-mint limit, ≤ remaining, and the minter's KAS value kept the same [S29]. Verify from source: [kcc-0020.md @3fbec524](https://github.com/kaspanet/kccs/blob/3fbec524/kcc-0020.md), [public_mint.ag @c8a08711](https://github.com/argent-lang/kcc20-reference/blob/c8a08711/contracts/public_mint.ag).

## 2. Transaction shape

**KRC-20 commit/reveal (P2SH).** The commit sends KAS to a pay-to-script-hash address whose script holds the JSON; once the commit is in, the reveal spends that output and shows the script; the reveal's id is the operation id the indexer tracks [S33, S40]. Our KNS work uses the same envelope (`<pubkey> OP_CHECKSIG OP_FALSE OP_IF <kns> <0> <payload> OP_ENDIF` in a P2SH), commit locks 1 tKAS, reveal returns it [S34].

**KCC-20 mint.** One transaction: minter in + payer coin in; new minter (remaining lowered), token output, carrier, change out [S22]. No commit/reveal. But one minter output can only be spent once at a time, so a launch that wants many parallel mints has to split the minter first: 1, 2, 4 … 2,048, 904 splits to reach 5,000 leaves [S20].

## 3. Mass and storage mass (KIP-9)

- **KRC-20, 2024 beta (mainnet, third-party):** wallet software sent 0.2 KAS outputs; under KIP-9 each transaction got about 25,000 grams of storage mass; a 500,000-gram block held about 20 [S30].
- **KCC-20, measured on TN10:** storage mass 61,710 to 76,586 per mint with a 1 tKAS carrier [S22]; 80,000 on Build's mints [S23]. That is about 6 to 8 mints per block on the storage budget (arithmetic) [S6].
- **Model:** thin 0.2 KAS carrier 232,955 grams (2 per block), 1 KAS 73,069 (6), 100 KAS 653 (765) [S20, packing table].
- **Which mass sets the fee is not one rule across our pages.** The 7 Oct single mint paid on `max(compute, ceil(transient/2))` and its storage mass (17,520) was higher than that relay mass [S20 RESULTS §3, LIVE]; the 9 and 11 Oct runs report that storage mass set the fee [S22, S23]. **Needs testing:** read the node's own ordering mass for each mint and log all three.

## 4. Who validates, and what that means in a rush

| | KRC-20 | KCC-20 |
|---|---|---|
| Node checks | Signatures, mass, fee, double spends of ordinary outputs | All of that **plus** the covenant rules |
| Mint validity | Indexer, after the fact | Consensus, before inclusion |
| Read state for the next mint | Indexer API | The minter's current output, from a node (an indexer may serve spent outputs [S10]) |
| A lagging indexer means | Users cannot tell if their mint counted; different indexers can disagree (KRC tokens live on three ledgers per manyfest) [S27] | Wallets may build on a stale minter and get rejected; consensus state is still right |

## 5. Failure modes under congestion

**KRC-20 (from the 2024 account and our KNS runs):** a large backlog of high-fee mint transactions pushed ordinary low-fee transactions out for hours [S30]; retrying against a lagging public node gave "already spent" reveals and stranded commits in our KNS bulk run [S34, finding 7]; the KNS indexer's `/owner` endpoint was tip-lag blocked after the 26 Sep stress window [S34]. **Needs testing for KRC-20 itself under storm.**

**KCC-20 (our evidence and open questions):** double-spend rejects on the minter [S20 §3]; long waits at a low feerate under overload (any transaction, [S8]); the mempool drops everything on a node restart [S9]; storage-mass-heavy mints fill blocks' storage budget fast [S22, S6]. Under storm: **needs testing** (this plan).

## 6. Fees

- KRC-20 beta: a 1 KAS fee per transaction gave about 4,000 sompi/gram even at high mass, against 1 to 4 typical then, so mints outranked ordinary traffic [S30] (third-party, mainnet, 2024; the network minimum relay fee was raised to 100 sompi/gram later, rusty-kaspa PR #1004 merged 15 May 2026 [S41]).
- KCC-20 on TN10: 204 sompi/gram on 11 Oct, 100 on 9 Oct, on a quiet network [S22, S23]. Not yet measured under a storm.
- Either way, fee = feerate × mass, and the mass is large, so a fee bump on a mint costs more tKAS than the same bump on a plain payment. The auto-fee cap must be in tKAS per mint, not just sompi per gram ([plan/AUTO-FEE.md](../plan/AUTO-FEE.md)).

## 7. UTXO and coin handling

- **KRC-20:** the commit output must exist before the reveal; a stranded commit can be reused, and our KNS create code builds the reveal from the commit's own change on the same node [S34, finding 7].
- **KCC-20:** the minter output is shared state. Track it from a node, not an indexer [S10]. The payer coin must be big: small coins plus high fees raise storage mass and lower throughput (round 5: a 150× fee on ~0.25 tKAS coins pushed storage mass to ~69k grams and network rate fell to 300–440) [S12, item 7]. Track coins already spent by your own pending transactions (1,322 of 1,327 games failed without it) [S12, item 4].

## 8. KRC-20 to KCC-20 migration

- **Sutton, 11 Oct 2026 (15:29:19Z / 17:29:19 CEST)**, asked whether a token can still migrate to KCC-20 if its KRC-20 minting is unfinished: "yes it can migrate with the same rules. it will need a custom migration with the target KCC-20 contracts supporting the same public minting policy over the remaining supply." He added that the answer is conceptual, not about any specific migration proposal [S26].
- **Sutton, 19 Aug:** prefers a one-time snapshot whose commitment sits in the KCC genesis (part of the covenant-id preimage) and can be checked against a KRC indexer muhash; an airdrop needs an admin key [S27].
- **manyfest, 20 Aug:** KRC tokens are on three ledgers (KRC20, Igra, Kasplex); proposes a coordinated snapshot and a KCC-20 minter. Proposal, no KCC, no code [S27].
- **Sutton, 11 Oct (16:07:40Z / 18:07:40 CEST), on KCC-20 vs Solana:** "not easily, no"; sophisticated designs can build a multi-contract Argent app across partitioned state and multiple transactions ("mini programs"), but "contention over shared variables/state" must be carefully designed [S26].

**Why it matters for a rush (C, opinion):** a migrated token with unfinished minting would reopen public minting on KCC-20 over the remaining supply. If that supply is popular, the reopening is a mint rush by design. **Hypothesis, not tested.**

## 9. What this page does not claim

- We have **not** minted KRC-20 ourselves; KRC-20 numbers are third-party or from KNS, which uses the same commit/reveal pattern.
- Mainnet: **not tested (label B)**.
