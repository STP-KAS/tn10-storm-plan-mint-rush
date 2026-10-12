> **Experimental. We are just trying this.** [Disclaimer](../DISCLAIMER.md) · [Home](../README.md) · TN10 only. Mainnet: **not tested (label B)**.

# Hypothesis

## The big one (stp's, 11 Oct 2026)

> A KCC-20 mint rush is likely once KCC-20 is adopted on mainnet. The open question is whether Kaspa absorbs it without ordinary users noticing. Kaspa is permissionless and that is the goal; no protocol change is proposed, and mint rushes are not "bad": they give the fee market a short-term fee and volume boost, with little lasting utility of their own. The concern is timing: if Kaspa breaks out of "its own village" (dapps, payments, many use cases, perhaps a crypto-casino influx and a new meme saga), a rush that disrupts mainnet for hours would hurt the users relying on it and Kaspa's image. Dapps, wallets and infrastructure should have congestion monitoring with an automatic, capped fee raise in place either way.
>
> It may turn out not to be a real concern. That would be a good outcome.

Source: [tn-master ADVICE, hypothetical section](https://github.com/STP-KAS/tn-master/blob/main/ADVICE-TO-DEVELOPERS.md#hypothetical-scenario-mint-rush-after-smart-contracts-stp-11-oct-2026), [FINDINGS 15](https://github.com/STP-KAS/tn-master/blob/main/FINDINGS.md), [OPEN-QUESTIONS 8](https://github.com/STP-KAS/tn-master/blob/main/docs/OPEN-QUESTIONS.md). **Hypothesis, open for debate. Not a prediction.** A TN10 test cannot prove what mainnet users will do. It can show what happens to mints, fees and ordinary traffic when a rush and a storm meet, and whether a capped auto fee helps.

**Who proposed what:** the hypothesis is **stp's**. The idea to test it as a **mint rush under storm load on TN10** came from **TN10 ops (the bot)**.

**Why it is plausible (C, opinion, with one precedent):** the 2024 KRC-20 beta on mainnet was exactly a mint wave that filled blocks; ordinary users waited for hours ([S30](SOURCES.md)). Sutton's 11 Oct reply that unfinished KRC-20 mints can migrate to KCC-20 "with the same rules … over the remaining supply" means public minting may reopen on KCC-20 ([S26](SOURCES.md)).

## What this test can actually check

Each line is a **hypothesis** with what would support it and what would refute it. Thresholds are fixed before T0 (Kaspa Pulse's rule, [S7](SOURCES.md) input 3).

| # | Hypothesis | Supported if | Refuted if | Based on |
|---|---|---|---|---|
| H2 | Under rush + storm, **capped auto-fee minters (B) land faster and more often than fixed-fee minters (A)**, at a bounded extra cost | B's p90 time to accept ≤ 10 s and mint success ≥ 99%, while A misses either; B's mean fee per mint ≤ its cap | A and B land the same, or B hits its cap most of the time and still misses | [S8] fee tiers under overload; [S17] r6 rule; [S19] r8: 2× normal kept covenants in the next block under storm |
| H3 | **Mints and plain payments crowd each other less than their tx counts suggest**, because mints fill the storage budget and payments the compute budget | Network unique accepted stays within 10% of the storm-only phase while mints land at their target | Storm throughput falls in proportion to mint mass | [S6] three mass budgets; [S22] mint masses. **Open** |
| H4 | **Minter contention, not block space, limits a public-mint rush** when many minters share few minter outputs | Double-spend / "already spent" rejects rise with minters per leaf; landed mints per second plateau below the storage-mass bound | Landed mints track the storage-mass bound regardless of collisions | [S20 §3] one rejected conflict on 7 Oct; [S23] 5,000-leaf split; [S26] Sutton on contention |
| H5 | **Carrier size changes rush capacity** (thin carriers fill blocks sooner) | Mints with a smaller carrier show higher storage mass and fewer mints per block | No mass difference | [S20] packing table (model). Optional phase |
| H6 | **A losing KCC-20 minter pays nothing** (rejected before inclusion), unlike a losing inscription reveal | Every collision shows as a node reject with no fee spent | Collisions show up as accepted transactions that paid | [S20 §3] (n=1), [S34] KNS "losing reveal still pays" |
| H7 | **Ordinary low-fee payments suffer during a rush** (time to accept rises) | Probe payments at the minimum fee wait longer in the rush phases than in storm-only | No change | [S8], [S30] |

## What would change the advice

- If H2 holds: tn-master ADVICE item 1 (auto fee raise with a cap) gets a mint-specific, TN10-measured example.
- If H2 fails because the cap is hit: the cap rule needs a budget per mint in tKAS, not a feerate multiple ([plan/AUTO-FEE.md](../plan/AUTO-FEE.md)).
- If H4 holds: the advice to launchers becomes "split the minter widely before launch", and to wallets "treat 'already spent' on a minter as a lost race; re-read the minter from a node and rebuild".

Every result stays **TN10 only**; mainnet stays **not tested (label B)**.
