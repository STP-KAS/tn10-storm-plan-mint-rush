> **Experimental. We are just trying this.** [Disclaimer](../DISCLAIMER.md) · [Home](../README.md) · TN10 only. Mainnet: **not tested (label B)**. **Draft algorithm under test, not a recommendation.**

# AUTO-FEE: the capped automatic fee raise under test (arm B)

What tn-master ADVICE item 1 says in one line, written out so it can be tested: *watch the network live, raise your fee when congestion rises, rebuild a transaction that waits too long, and cap it so a flood cannot drain you* ([S4] item 1). Every rule below names the evidence it comes from. Constants marked *draft* are starting values, logged at T0, never changed mid-phase without a `deviation` row.

## Signals (read every 5 s, per node in use)

| Signal | Source | Why | Evidence |
|---|---|---|---|
| `getFeeEstimate`: priority, normal[0], low[0] | node RPC | The node's own view of the price for the next block | r6 rule used normal[0] ([S17]); r8 followed it under storm ([S19]) |
| Mempool size | node RPC | Backlog building | round 1: "Detect congestion (mempool size / fee-estimate jump) and auto-bump" ([S8] Mitigation ideas) |
| Own time to accept | own `virtual-chain-changed` subscription | The only signal that is about **our** mints | round 1: rebuild if a tx waits > 10 s ([S8]) |
| Own rejects by reason | submit replies | `fee-too-low`, `mempool-full`, `double-spend` mean different fixes | [S24] reasons |
| Same estimate from other nodes | `getFeeEstimate` on 2 or more live nodes (ideally 3) | One node's estimate can be badly wrong: a stale mempool made one TN10 node quote priority 1,644 vs 100 on another (upstream report, not tested by us) | [S42], [S31] |
| Node state | node-dead rule | Do not read prices from a dead or lagging node | Kaspa Pulse ([S7] input 12) |

## Rules

**R0. Multi-node fee read (applies before R1 to R4).** Every 5 s read `getFeeEstimate` from **at least 2, ideally 3, live nodes** (node-dead rule R10 first). Take the **median** of `priority` and of `normal[0]` across them; R1 to R4 use these medians, never one node's raw numbers. A node whose `priority` is **more than 3× the median priority** is ignored for that read and logged as a `fee-outlier` event (node, its priority and normal[0], the median). With only 2 live nodes a median cannot expose an outlier: if their priority buckets differ by more than 3×, use the lower node and log `fee-outlier`. With 1 live node, R0 cannot run: log `fee-outlier` reason `single-node` and keep the cap of R4 (no raise above the last multi-node median). Evidence: a non-mining TN10 node with a stale mempool (about 21.8k entries, about 99.7% already accepted) quoted priority 1,644 against 100 on a public endpoint, so fee logic that followed its priority bucket would overpay about 8-15× until the 24 h expiry ([S42]; upstream, **not tested by us**). Constants 3× and 2-3 nodes are *draft*.

**R1. Base feerate.** `base = max(2 × normal[0], 2 × floor)`, recomputed every 30 s. `floor` = the node's minimum relay feerate (100 sompi/gram on our TN10 nodes, [S8]; mainnet's floor was raised by rusty-kaspa PR #1004, merged 15 May 2026, [S41]). Evidence: the round 6 rule `max(2 × normal[0], 200)` ([S17]); round 8: 2× normal kept covenant actions in the next block under storm with ≈20% more fee ([S19]). The 1 Oct storm used normal × 3, floor 300 ([S38]).

**R2. Congestion jump.** If `normal[0]` rises by more than 50% within 60 s, or the mempool of the read node passes 50,000 (*draft*), new mints use `min(priority, cap_rate)` until both calm down for 2 minutes. Evidence: [S8] Mitigation ideas.

**R3. Bump a waiting mint.** If a mint is not accepted 10 s after submit (*draft*, from [S8]), rebuild it at `min(1.5 × previous feerate, cap_rate)` on the **same inputs** (same minter output, same payer coin), so the old and new versions conflict and at most one lands. At most 4 bumps. **Needs testing:** whether the node accepts the higher-fee version while the old one is still in its mempool (replacement) or rejects it as a double spend; both outcomes are logged. If replacement is refused, the minter waits for the old version to land or leave, then rebuilds.

**R4. Cap, in two units.**
- `cap_rate = min(priority, 5 × normal[0])` sompi/gram (*draft* k = 5), with `priority` and `normal[0]` the R0 medians, so one node with a stale mempool cannot lift the cap. Evidence: priority vs normal reached 8.4× and paying priority as-is burned about 900 tKAS in 11.5 min ([S12] item 5, [S14]); the draft issue proposes `min(priority, k × normal)` or an absolute maximum ([S14]).
- `cap_fee = 0.5 tKAS per mint` (*draft*): an absolute tKAS cap, because mint mass is large (storage 61,710 to 76,586 grams, [S22]) and a feerate multiple alone can still cost a lot per mint ([KCC20-VS-KRC20 §6](../docs/KCC20-VS-KRC20.md#6-fees)).
- At the cap: **return an error and pause that minter**, never drain ([S12] item 5).

**R5. Never buy priority by shrinking change.** The fee comes from a payer coin of ≥ 10 tKAS (*draft*) so the change stays large and storage mass does not jump. Evidence: a 150× fee on ~0.25 tKAS coins raised storage mass to ~69k grams, lowered the effective feerate to ~140 sompi/gram and dropped network rate to 300–440 tx/s ([S12] item 7, [S16]). Log storage mass before and after each bump.

**R6. Fee = feerate × the node's ordering mass.** Compute the mint's compute, transient and storage mass and pay on the one the node orders by. Our pages disagree on which one that is for a covenant mint ([KCC20-VS-KRC20 §3](../docs/KCC20-VS-KRC20.md#3-mass-and-storage-mass-kip-9)); log all three. **Needs testing.**

**R7. Track coins in flight.** Keep a set of outpoints spent by our own pending mints; never build on one of them; release it only when the spending mint lands or is known gone. Evidence: 1,322 of 1,327 games failed with "already spent … in the mempool" without this ([S12] item 4); tn-master ADVICE 8 ([S4]).

**R8. Read minter state from a node, never from an indexer alone.** After a `double-spend` on a minter output (a lost race), re-read that leaf's current output from a node with a UTXO index, then rebuild on the new minter state or move to another leaf. Evidence: the public indexer served spent outputs; "orphan" rejects are the signal ([S10], [S13]); 7 Oct conflict reject ([S20] §3).

**R9. Rebroadcast after reconnect or restart.** On a node restart or socket reconnect, resubmit every pending mint that is not on the chain. Evidence: the mempool is not saved on restart ([S9]); upstream, a node finishing IBD may keep stale transactions in its mempool, fix proposed in rusty-kaspa PR #1147, not tested by us ([S31]).

**R10. Node-dead rule.** Stop using a node for prices or submits when it is dead or lagging; fail over (Kaspa Pulse, [S7]).

**R11. Back off on `mempool-full`.** Retry after 1, 2, 4 s (*draft*), on another live node if one exists ([S4] item 6).

## Arm A (fixed fee), for comparison

Same minters, same pool, same coins, same R7 to R11. R0 still runs for logging (`fee-outlier` events are written in both arms), but arm A never changes its fee. Only R1 to R4 are replaced by one fixed feerate: the phase 1 normal estimate (*draft*, [PLAN.md](PLAN.md) open decision 8). No bumps.

## What is logged for every arm B mint

R0 medians and the nodes used for each read, `fee-outlier` events, feerate at each attempt, bump count, reason for each bump (R2 or R3), cap hit yes/no, masses, fee in tKAS, outcome. Columns: [../go/LOGGING.md](../go/LOGGING.md).

## Pseudocode

```text
every 5 s:  read fee estimate, mempool size, node state from 2-3 live nodes (skip dead nodes)
            prio = median(priority), normal0 = median(normal[0]); drop and log fee-outlier any node with priority > 3*prio
            (2 nodes: if priorities differ > 3x, use the lower one, log fee-outlier)
every 30 s: base = max(2*normal0, 2*floor)
            cap_rate = min(priority, 5*normal0)
            congested = normal0 jumped >50% in 60 s or mempool > 50,000
mint:
  leaf  = random leaf from pool, current state read from a node
  coin  = payer coin not in in_flight, >= 10 tKAS
  rate  = congested ? min(priority, cap_rate) : base
  fee   = rate * ordering_mass(tx)
  if fee > cap_fee: error, pause minter
  submit; add inputs to in_flight; start timer
  if not accepted after 10 s and bumps < 4:
      rate = min(1.5*rate, cap_rate); rebuild on same inputs; resubmit; log bump
  on double-spend of the minter: re-read leaf from node; pick new leaf; rebuild
  on reconnect/restart: rebroadcast all pending not on chain
  on accepted or gone: remove inputs from in_flight
```
