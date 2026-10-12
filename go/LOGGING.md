> **Experimental. We are just trying this.** [Disclaimer](../DISCLAIMER.md) · [go/](README.md) · TN10 only. **NOT ARMED: rehearsal pending.**

# LOGGING: both sides, same files, same columns

Both sides log the same files with the same columns, so the bot's minutes, Build's minutes and Kaspa Pulse's minutes line up row by row. Base: [tn10-storm-2026-10-11 go/LOGGING.md](https://github.com/STP-KAS/tn10-storm-2026-10-11/blob/main/go/LOGGING.md), plus the mint files below. Thank you, Kaspa Pulse (@gokugalax, kaspapulse.com): the per-UTC-minute format, the chain count before pruning and the two-node count are his.

Logging and probes run from **R1 (T0−00:30)** to END. The count files are written after END. Land in `logs/bot/`, `logs/build/` and, for the rehearsal, `logs/rehearsal/`.

## Format (unchanged from 11 Oct)

- CSV, UTF-8, comma, one header row, LF line ends.
- Time in UTC, ISO 8601 with `Z`. Seconds `2026-10-11T07:30:05Z`, minutes `2026-10-11T07:30Z`.
- `side` is `bot` or `build`. `step` is `PREP`, `R` (rehearsal), `P1` … `P7`, `P9` (crowd mint, separate run) (the [PLAN.md](../plan/PLAN.md) phase) plus a step suffix, e.g. `P5-s3`, `idle` (a side not sending), or `COUNT`.
- An empty cell means **not measured**. Never 0 for not measured.
- **Counts only in git.** No txid, key, seed, host IP, port, covenant id or wallet address. Public node names are allowed. Per-transaction logs stay on the machine that made them.

## Files kept from 11 Oct (same columns)

`sec.csv`, `minute.csv`, `public-nodes.csv`, `rejects.csv`, `miner.csv`, `host.csv`, `node.csv`, `events.csv`, `steps.csv`, `gates.csv`, `ntp.csv`, `chain-count.csv`, `confirm.csv`, `pulse-method-minute.csv`, `SUMMARY.md`. Columns: [11 Oct LOGGING](https://github.com/STP-KAS/tn10-storm-2026-10-11/blob/main/go/LOGGING.md#files) (their `step` takes the values above). Changes:

- `rejects.csv` `reason` gains `script` (covenant rule failed) next to fee-too-low, mempool-full, orphan, double-spend, mass, rpc-timeout, rpc-closed, other.
- `confirm.csv` must be filled this time (it has only a header row on 11 Oct), with `fee_tier` = probe tier or `mint-A` / `mint-B`, and a `p99_s` column added.
- `events.csv` gains `phase-start`, `step-start`, `fee-bump`, `fee-cap-hit`, `fee-outlier`, `minter-collision`, `rebroadcast`.

## New files

### mint-minute.csv (every minute, per arm)

| Column | Meaning | Unit |
|---|---|---|
| `minute_utc` | the minute | ISO UTC minute |
| `side` | bot or build | text |
| `step` | phase and step | text |
| `arm` | `A` fixed fee or `B` auto fee | text |
| `pool` | `wide` or `narrow` | text |
| `target_mints_s` | step target | mints/s |
| `minters_live` | minter processes that wrote a row | count |
| `attempts` | mint submits, bumps included | tx |
| `submit_ok` | taken into a mempool | tx |
| `landed` | mint ids in the virtual chain's accepted ids | tx |
| `rejects` | failed submits (reasons in rejects.csv) | tx |
| `collisions` | rejects whose conflicting input is a minter output | tx |
| `evicted` | left a mempool, not on chain after 120 s | tx |
| `bumps` | rebuilds at a higher fee | count |
| `cap_hits` | mints that reached a cap and paused | count |
| `inflight_errors` | builds that hit a coin already in flight | count |
| `tta_p50_s` / `tta_p90_s` / `tta_p99_s` / `tta_max_s` | time to accept of mints landed this minute | s |
| `fee_tkas_sum` | fees of landed mints | tKAS |
| `feerate_mean` / `feerate_max` | feerate of landed mints | sompi/gram |

### feeest.csv (every 5 s, per node read)

| Column | Meaning | Unit |
|---|---|---|
| `utc` | time | ISO UTC |
| `side` | bot or build | text |
| `node` | `box`, Build's desk node, or a public node name | text |
| `priority` | priority bucket feerate | sompi/gram |
| `normal0` / `normal1` | first two normal buckets | sompi/gram |
| `low0` | first low bucket | sompi/gram |
| `mempool_size` | that node's mempool | tx |
| `congested` | AUTO-FEE R2 state | true/false |

### block-mass.csv (every minute, from the chain count)

| Column | Meaning | Unit |
|---|---|---|
| `minute_utc` | the minute | ISO UTC minute |
| `blocks` | chain blocks | blocks |
| `mint_tx` | accepted mints (by our known covenant, matched locally; only the count is written) | tx |
| `other_tx` | all other accepted transactions | tx |
| `compute_mass_mean` / `storage_mass_mean` / `transient_mass_mean` | mean per chain block | grams |
| `complete_minute` | 11 Oct complete-minute rule | true/false |

### mint-mass-sample.csv (5 mints per minute)

`minute_utc, arm, compute_mass, transient_mass, storage_mass, ordering_mass, fee_sompi, carrier_tkas, payer_coin_tkas` (counts and masses only, no ids).


## Who writes what

| File | Bot | Build |
|---|---|---|
| sec, minute, public-nodes, rejects, miner, host, node, events, gates, ntp, steps, SUMMARY | yes | yes |
| chain-count, block-mass, pulse-method-minute | yes (the box node) | yes (desk node) |
| mint-minute, feeest, mint-mass-sample | yes | yes |
| confirm | yes (probes and mints) | yes (mints) |

## Events added for this run

`phase-start`, `step-start`, `saturated` (accepted < 95% of submitted for 60 s), `fee-freeze` (arm A fee set at T0+00:09), `fee-bump`, `fee-cap-hit`, `fee-outlier` (a node's priority bucket more than 3× the median of the live nodes read, or 2 nodes more than 3× apart, or only 1 live node: node, its priority and normal[0], the median, reason; [AUTO-FEE R0](../plan/AUTO-FEE.md)), `minter-collision`, `rebroadcast`, plus every 11 Oct event (`gate`, `rate-change`, `fix`, `node-dead`, `node-back`, `node-restart`, `miner-on`, `miner-off`, `coin-refresh`, `disk-guard`, `halt`, `deviation`, …).

## Handoff files, Build to the bot

stp carries these. `key: value`, one per line, UTC times.

`FOR-TN10-OPS-ready-mintrush.txt` (at R2):

```
build_ready: yes
written_utc: <time>
own_node_synced: yes | no
utxoindex: yes | no
senders: <n>
storm_rate_tx_s: <n>
coins_ready_tkas: <n>
fee_pair: 200/300
miner: on | off
disk_free_gb: <n>
count_reader: running | not running
notes: <anything the bot should know>
```

`FOR-TN10-OPS-result-mintrush.txt` (by T0+05:00):

```
phases: one line per phase (P2, P4, P5, P6, P7): mean_sent, mean_submit_ok, mean_accepted, rejects
network_unique: per phase: peak complete minute, mean, median (from your chain-count.csv)
events: every rate-change, fix, restart, node-dead, deviation (UTC)
```

## After the run

COUNT within two hours of END from both nodes, before pruning (Kaspa Pulse). Then `steps.csv`, `SUMMARY.md`, the README Results section, and tn-master's storm page. Counts only; leak-scan before every push.
