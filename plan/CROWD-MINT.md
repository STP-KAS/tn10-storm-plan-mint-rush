# Phase 9: crowd mint, KASPER-style (DRAFT, NOT ARMED)

> **NOT ARMED.** Approved by stp as a design only. **Date: TBD (stp decides).** Testnet-10 only, tKAS has no value. Separate run, about 1 hour, announced a day ahead in the TN10 channels with a UTC window.

## One question

**Do ordinary payments still confirm during a 2,000-wallet mint rush?**

## Token: KASPER's real KRC-20 tokenomics, copied

Verified from the Kasplex API, [api.kasplex.org/v1/krc20/token/KASPER](https://api.kasplex.org/v1/krc20/token/KASPER), read 12 Oct 2026 16:52 UTC (`dec` = 8):

| Field | API value | Tokens |
|---|---|---|
| `max` | 2870000000000000000 | **28,700,000,000** max supply |
| `lim` | 2870000000000 | **28,700** per mint |
| `pre` | 0 | no premine |
| `mintTotal` | 1000000 | **1,000,000 mints** (= max / lim) |
| `state` | finished | fully minted |
| `mtsAdd` / `mtsMod` | 1726412411492 / 1726422569778 | 15 Sep 2024 15:00:11 UTC / 17:49:29 UTC (deploy/added and last state change; mainnet KRC-20 launch day) |
| `holderTotal` | 15088 | holders as of the read |

Our test token copies **max 28,700,000,000, 28,700 per mint, no premine, so 1,000,000 mints to mint out**. The steps below land about 33,300 mints (arithmetic), about 3.3% of supply, so the token does **not** mint out in the hour; that is intended (we measure the rush, not the finish). The deployer address from the API is not copied here.

**Token standard.** KCC-20 if a working KCC-20 minter is available on TN10 on the day; otherwise a **KRC-20-style stand-in (commit/reveal, indexer-checked), labelled "stand-in, not KCC-20"** in every file and chart. Which one: **TBD** at the gate.

## Wallets and coins

- **About 2,000 wallets** derived from **one test seed** (testnet only). The seed **never** goes in git or in logs; only wallet indexes 0-1999 are logged.
- **Fan-out funding** from the bot's main wallet (address not in git): about **10 tKAS each**, so about 20,000 tKAS plus fees (arithmetic). Fan-out done and confirmed before the gate; coins checked against a node, not an indexer (ADVICE 7).
- Fees per mint on the stand-in: **TBD** (KCC-20 on TN10 paid 0.126-0.156 tKAS per mint on 11 Oct, [tn-master FINDINGS 16](https://github.com/STP-KAS/tn-master/blob/main/FINDINGS.md)). Check that 10 tKAS covers each wallet's share before arming.

## Senders and behaviour

| Group | Share of wallets | Behaviour |
|---|---|---|
| Normal | most (about 80%, design) | Pay the node's **normal** fee estimate |
| Overpayers | some (about 15%, design) | Pay the **priority** estimate or a fixed multiple, capped |
| Retry-spammers | a few (about 5%, design) | Resubmit on any delay or reject, short backoff; logged as such |

- Senders split between **the box and Build**; each side submits to its own node plus public fallbacks (node-dead rule).
- **Shared launch second:** every minter starts at the same UTC second (T0, announced).
- **Steps: the existing 2 / 5 / 10 / 20 mints/s**, 15 min each (about 1 hour). Mints per step: 1,800 / 4,500 / 9,000 / 18,000 = about 33,300 (arithmetic).
- **Ordinary-payment probe:** a separate sender makes ordinary small payments at the normal fee throughout (rate **TBD**, low, e.g. 1/s), with a 10-min baseline before T0. This is the answer to the question.

## Metrics (per minute unless noted; columns as [METRICS.md](METRICS.md))

| Metric | Why (2024 lesson, [KRC20-HISTORY §5](../docs/KRC20-HISTORY.md#5-lessons-for-the-tn10-mint-rush-test)) |
|---|---|
| Mints per block, storage mass per mint | Small outputs → high mass → ~20-40 tx/block in the 2024 beta |
| Mempool size per node | ~37k backlog in the 2024 beta |
| Fee-estimate jumps, **multi-node** (box, Build, public nodes; resynced-node drift as rusty-kaspa #1147) | No dynamic fees in 2024; stale mempools can skew estimates |
| Failures by reason (fee too low, already spent, orphan, mempool full, RPC) | Mints under the 1 KAS minimum failed but paid in Sep 2024 |
| Wallet change errors (change missing or wrong; fee paid vs intended) | Sep 2024 wallet bug sent change to miners as fee |
| **Ordinary-payment wait time** p50/p90/p99 vs baseline | Low-fee user txs waited hours in the 2024 beta |
| Retry-spammer share of mempool and of rejects | Retry storms add load without adding mints |

## Gates and stop rules

- Gate: nodes synced, fan-out confirmed, token standard chosen (KCC-20 or labelled stand-in), launch second announced, ≥30 GB free on Build, 10 GB box guard as 11 Oct.
- Only stp halts. Node-dead rule, freeze restart, count right after END before pruning, leak-scan before push (no seed, keys, addresses, txids, IPs).
