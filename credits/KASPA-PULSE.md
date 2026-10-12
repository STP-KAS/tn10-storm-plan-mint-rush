> **Experimental. We are just trying this.** [Disclaimer](../DISCLAIMER.md) · [Home](../README.md)

# Kaspa Pulse (@gokugalax, kaspapulse.com): inputs to this plan

He counts our storms from the outside on his own two nodes. Contact goes through stp only; the bots never ping him.

| # | Date | Input | What we did | Where |
|---|---|---|---|---|
| 1 | before 11 Oct | **Method**: plan first, 10-minute baseline, fixed steps, saturation defined before T0, node-dead rule, count right after END before pruning | Rules 1-6 of the plan | [plan/PLAN.md](../plan/PLAN.md#rules-carried-over-do-not-change-without-a-logged-deviation), [go/README.md](../go/README.md), [tn-master inputs](https://github.com/STP-KAS/tn-master/blob/main/credits/KASPA-PULSE.md) |
| 2 | 10-12 Oct | **His counts as baseline evidence**: accepted in blocks, two independent nodes, per UTC minute, by block time; ~56 tx/s base load | Headline counter and cross-check; base-load figure used throughout | [plan/METRICS.md](../plan/METRICS.md), [tn10-storm-2026-10-11 credits](https://github.com/STP-KAS/tn10-storm-2026-10-11/tree/main/credits/kaspa-pulse), [tn10-storm-2026-10-10 credits](https://github.com/STP-KAS/tn10-storm-2026-10-10/tree/main/credits/kaspa-pulse) |
| 3 | 12 Oct | **Short runs, one clear question each**, about an hour, announced a day ahead in the TN10 channels with a UTC window | Crowd-mint phase designed as a separate ~1 h run with one question | [plan/CROWD-MINT.md](../plan/CROWD-MINT.md) |
| 4 | 12 Oct | **Multi-node fee logging offer**: fee estimate of both his nodes every minute | Fee estimates read from several nodes (AUTO-FEE R0, #1147 effect) | [plan/AUTO-FEE.md](../plan/AUTO-FEE.md) |
| 5 | 12 Oct | **Review of this plan**: the question is fair and worth its own run after Saturday 17 Oct | Kept as its own run; date TBD (stp decides) | [docs/REVIEW-NOTES.md](../docs/REVIEW-NOTES.md) |
| 6 | 12 Oct | **The narrower countable question**: while a mint wave runs, do normal payments at normal fees still get in, and how long do they wait? Whether minting "dilutes" mainnet is a different question TN10 cannot answer | Stated in the README (without naming him in the Why) and measured by ordinary-payment wait times | [README](../README.md), [plan/METRICS.md](../plan/METRICS.md) |
| 7 | 12 Oct | **Second review of this plan** (run size, pre-date checks). Approved text below, approved by him 12 Oct 2026 ([row 7 text](#row-7-second-review-12-oct-2026)) | Run 1 = crowd mint first and alone; pre-date checklist; not-a-mint-out line | [plan/PLAN.md](../plan/PLAN.md#run-order-12-oct-2026-after-outside-review) |

## Row 7: second review, 12 Oct 2026

Text approved by Kaspa Pulse (@gokugalax, kaspapulse.com) on 12 Oct 2026, published verbatim:

1. **Size.** Kaspa Pulse (@gokugalax, kaspapulse.com) pointed out that phases 1 to 7 asked six questions in about two and a half hours under a 1,200 tx/s storm, so a problem in an early phase would make every later phase hard to read. He suggested running the crowd mint (ordinary payments during a crowd mint) first and alone, with the probe payments as the headline, and the auto-fee A/B, contention and carrier tests each as a short run later. Adopted: Run 1 = crowd mint.
2. **Time to accept.** He noted that time to accept was not measured on 11 Oct (confirm.csv stayed empty), and that this number is the answer, so the rehearsal must prove the logger works. Adopted: rehearsal pass criterion 6.
3. **Mint/payment split.** He noted that his counter sees accepted transactions and blocks per minute and cannot tell mints from payments, so the split has to come from our logs. Adopted: tx_type on every send row.
4. **Not a mint-out.** He noted that the run copies KASPER's 1,000,000-mint cap but lands about 33,300 mints, and that the announcement should say so. Adopted: announcement text in CROWD-MINT.md.
